<template>
    <v-container fluid class="d-flex flex-column align-center" style="background-color: var(--v-theme-surface)">
      <!-- First Row -->
      <v-row style="max-width: 1200px; width: 100%;">
        <!-- Temperature Graph Column -->
        <v-col class="flex-grow-1 flex-shrink-0" cols="9" >
          <figure class="highcharts-figure">
            <div id="container"></div>
          </figure>
        </v-col>
        
        <!-- Cards Column -->
        <v-col cols="3" class="flex-grow-0 flex-shrink-1">
          <v-card class="mb-5" color="primaryContainer">
            <v-card-item>
               <span class="text-h3 text-onPrimaryContainer">{{ temperature }} °C</span>
            </v-card-item>
            <v-card-subtitle>Temperature</v-card-subtitle>
          </v-card>
  
          <v-card class="mb-5" color="tertiaryContainer">
            <v-card-item>
              <span class="text-h3 text-onTertiaryContainer">{{ heatindex }} °C</span> 
            </v-card-item>
            <v-card-subtitle>Heat Index (Feels like)</v-card-subtitle>
          </v-card>
  
          <v-card class="mb-5" color="secondaryContainer">
            <v-card-item>
              <span class="text-h3 text-onSecondaryContainer">{{ humidity }} %</span> 
            </v-card-item>
            <v-card-subtitle>Humidity</v-card-subtitle>
          </v-card>
        </v-col>
      </v-row>
  
      <!-- Second Row -->
      <v-row style="max-width: 1200px; width: 100%;" justify="start">
        <!-- Humidity Graph Column -->
        <v-col class="flex-grow-1 flex-shrink-0" cols="9" >
          <figure class="highcharts-figure">
            <div id="container1"></div>
          </figure>
        </v-col>
      </v-row>
    </v-container>
  </template>
<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { useAppStore } from "@/store/appStore";
import { storeToRefs } from "pinia"; 
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
Exporting(Highcharts);
more(Highcharts);
 
// VARIABLES
const Mqtt = useMqttStore();
const AppStore = useAppStore();
const router      = useRouter();
const route       = useRoute(); 
const { payload, payloadTopic } = storeToRefs(Mqtt); 
const tempHiChart = ref(null);
const tempHiChart1 = ref(null);
const CreateCharts = async () => {
// TEMPERATURE CHART
tempHiChart.value = Highcharts.chart('container', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF"},
title: { text: 'Temperature Analysis (Live)', align: 'left' },
yAxis: {
title: { text: 'Air Temperature' , style:{color:'#000000'}},
labels: { format: '{value} °C' }
},
xAxis: {
type: 'datetime',
title: { text: 'Time', style:{color:'#000000'} },
},
tooltip: { shared:true, },
series: [
{
name: 'Temperature',
type: 'spline',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[0]
},
{
name: 'Heat Index',
type: 'spline',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[5]
}
],
});

tempHiChart1.value = Highcharts.chart('container1', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF" },
title: { text: 'Humidity Analysis (Live)', align: 'left' },
yAxis: {
title: { text: 'Humidity' , style:{color:'#000000'}},
labels: { format: '{value} %' }
},
xAxis: {
type: 'datetime',
title: { text: 'Time', style:{color:'#000000'} },
},
tooltip: { shared:true, },
series: [
{
name: 'Humidity',
type: 'spline',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[3]
}],
});
};

const points = ref(20); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false);

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();
    Mqtt.connect();

    setTimeout(()=>{
    // Subscribe to each topic
    Mqtt.subscribe("620161521");
},3000);
});

onBeforeUnmount(()=>{
  Mqtt.unsubcribeAll();
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
});


// WATCHERS


watch(payload,(data)=> {
if(points.value > 0){ points.value -- ; }
else{ shift.value = true; }
tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 },
true, shift.value);
tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 },
true, shift.value);
tempHiChart1.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 },
true, shift.value);
})

// COMPUTED PROPERTIES

const temperature = computed(()=>{
if(!!payload.value){
return `${payload.value.temperature.toFixed(2)}`;
}
});

const humidity = computed(()=>{
if(!!payload.value){
return `${payload.value.humidity.toFixed(2)}`;
}
});

const heatindex = computed(()=>{
if(!!payload.value){
return `${payload.value.heatindex.toFixed(2)}`;
}
});


</script>




<style scoped>
/** CSS STYLE HERE */
Figure {
border: 2px solid black;
}



</style>