<template>
  <v-container fluid style="background-color: var(--v-theme-surface)">
    <!-- First Row -->
    <v-row style="max-width: 1200px; width: 100%;" class="pa-1">
      <v-col cols="6">
        <v-sheet class="pa-2" height="250">
          <p>Enter date range for Analysis</p>
          <v-divider></v-divider>
          <!-- Start Date -->
          <v-text-field
            label="Start date"
            type="date"
            density="compact"
            variant="solo-inverted"
            class="mr-5"
            flat
            style="max-width: 300px;"
            v-model= "start"
          ></v-text-field>
          <!-- End Date -->
          <v-text-field
            label="End date"
            type="date"
            density="compact"
            variant="solo-inverted"
            flat
            style="max-width: 300px;"
            v-model= "end"
          ></v-text-field>
          <v-spacer></v-spacer>
          <!-- Analyze Button -->
          <v-btn @click="updateLineCharts(); updateCards(); updateHistogramCharts()" text="Analyze" color="primary" variant="tonal"></v-btn>
        </v-sheet>
      </v-col>
      <v-col cols="3" class="d-flex justify-center align-center">
        <v-card title="Temperature" width="250" variant="outlined" color="primary" density="compact" rounded="lg" class = "text-center">
          <!-- Min, Range, Max -->
          <v-card-item class="mb-n5">
            <v-chip-group class="d-flex flex-row justify-center" color="primaryContainer" variant="flat">
              <v-tooltip text="Min" location="start">
                <template v-slot:activator="{ props }">
                  <v-chip v-bind="props">{{ temperature.min }}</v-chip> 
                </template>
              </v-tooltip>

              <v-tooltip text="Range" location="top">
                <template v-slot:activator="{ props }">
                  <v-chip v-bind="props">{{ temperature.range }}</v-chip>
                </template>
              </v-tooltip>

              <v-tooltip text="Max" location="end">
                <template v-slot:activator="{ props }">
                  <v-chip v-bind="props">{{ temperature.max }}</v-chip> 
                </template>
              </v-tooltip>
            </v-chip-group>
          </v-card-item>

          <!-- Average Temperature -->
          <v-card-item class="align-center justify-center">
            <span class="text-h1 text-primary font-weight-bold">
              {{ temperature.avg }} 
            </span>
          </v-card-item>
        </v-card>
      </v-col>


      <v-col cols="3" class="d-flex justify-center align-center">
        <v-card title="Humidity" width="250" variant="outlined" color="primary" density="compact" rounded="lg" class = "text-center">
          <!-- Min, Range, Max -->
          <v-card-item class="mb-n5">
            <v-chip-group class="d-flex flex-row justify-center" color="primaryContainer" variant="flat">
              <v-tooltip text="Min" location="start">
                <template v-slot:activator="{ props }">
                  <v-chip v-bind="props">{{ humidity.min }}</v-chip> 
                </template>
              </v-tooltip>

              <v-tooltip text="Range" location="top">
                <template v-slot:activator="{ props }">
                   <v-chip v-bind="props">{{ humidity.range }}</v-chip> 
                </template>
              </v-tooltip>

              <v-tooltip text="Max" location="end">
                <template v-slot:activator="{ props }">
                   <v-chip v-bind="props">{{ humidity.max }}</v-chip> 
                </template>
              </v-tooltip>
            </v-chip-group>
          </v-card-item>

          <!-- Average Temperature -->
          <v-card-item class="align-center justify-center">
            <span class="text-h1 text-primary font-weight-bold">
              {{ humidity.avg }} 
            </span>
          </v-card-item>
        </v-card>
      </v-col>
    </v-row>

    <!-- Second Row -->
    <v-row style="max-width: 1200px; width: 100%;">
      <v-col cols="12">
        <figure class="highcharts-figure">
          <div id="container"></div>
        </figure>
      </v-col>
      <v-col cols="12">
        <figure class="highcharts-figure">
          <div id="container0"></div>
        </figure>
      </v-col>
    </v-row>
    <!-- Third Row -->
    <v-row style="max-width: 1200px; width: 100%;">
      <v-col cols="12" class="border">
        <figure class="highcharts-figure">
          <div id="container1"></div>
        </figure>
      </v-col>
      <v-col cols="12">
        <figure class="highcharts-figure">
          <div id="container2"></div>
        </figure>
      </v-col>
      <v-col cols="12">
        <figure class="highcharts-figure">
          <div id="container3"></div>
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
const start = ref("");
const end = ref("");
const temperature = reactive({"min":0,"max":0,"avg":0,"range":0})
const humidity = reactive({"min":0,"max":0,"avg":0,"range":0})
const tempHiChart = ref(null);
const tempHiChart1 = ref(null); 
const tempHiChart2 = ref(null);
const tempHiChart3 = ref(null);
const tempHiChart4 = ref(null);// Chart object
const CreateCharts = async () => {
// TEMPERATURE CHART
tempHiChart.value = Highcharts.chart('container', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF" },
title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
subtitle: {text: 'The heat index, also known as the "apparent temperature," is a measure that combines air temperature and relative humidity to assess how hot it feels to the human body. The relationship between heat index and air temperature is influenced by humidity levels. As humidity increases, the heat index also rises, making the perceived temperature higher than the actual air temperature.', align: "left"},
yAxis: {
title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}},
labels: { format: '{value} °C' }
},
xAxis: {
type: 'datetime',
title: { text: 'Time', style:{color:'#000000'} },
},
tooltip: { shared:true, pointFormat: 'Humidity: {point.x} % <br/> Temperature: {point.y} °C' },
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
color: Highcharts.getOptions().colors[1]
} ],
});

tempHiChart1.value = Highcharts.chart('container0', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF" },
title: { text: 'Humidity Analysis', align: 'left' },
yAxis: {
title: { text: 'Humidity' , style:{color:'#000000'}},
labels: { format: '{value} %' }
},
xAxis: {
type: 'datetime',
title: { text: 'Time', style:{color:'#000000'} },
},
tooltip: { shared:true, pointFormat: 'Humidity: {point.x} % <br/> Temperature: {point.y} °C'},
series: [
{
name: 'Humidity',
type: 'spline',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[3]
}],
});

tempHiChart2.value = Highcharts.chart('container1', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF" },
title: { text: 'Frequency Distribution Analysis', align: 'left' },
series: [{name: 'Humidity', type: 'column', data: [], turboThreshold: 0,color: Highcharts.getOptions().colors[0], pointInterval: 3600000},
{name: 'Temperature', type: 'column', data: [], turboThreshold: 0,color: Highcharts.getOptions().colors[1], pointInterval: 3600000},
{name: 'Heat Index', type: 'column', data: [], turboThreshold: 0,color: Highcharts.getOptions().colors[2], pointInterval: 3600000}
],
});


tempHiChart3.value = Highcharts.chart('container2', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF" },
title: { text: 'Temperature & Heat Index Correlation Analysis', align: 'left' },
yAxis: {
title: { text: 'Heat Index' , style:{color:'#000000'}},
labels: { format: '{value} °C' }
},
xAxis: {
title: { text: 'Temperature', style:{color:'#000000'} },
labels: { format: '{value} °C' }
},
tooltip: { shared:true, pointFormat: 'Temperature: {point.x} °C <br/> Heat Index: {point.y} °C'},
series: [{name: 'Analysis',
type: 'scatter',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[6]
}],
});

tempHiChart4.value = Highcharts.chart('container3', {
chart: { zoomType: 'x', backgroundColor: "#D4E2FF" },
title: { text: 'Humidity & Heat Index Correlation Analysis', align: 'left' },
yAxis: {
title: { text: 'Heat Index' , style:{color:'#000000'}},
labels: { format: '{value} °C' }
},
xAxis: {
title: { text: 'Humidity', style:{color:'#000000'} },
labels: { format: '{value} %' }
},
tooltip: { shared:true, pointFormat: 'Humidity: {point.x} % <br/> Heat Index: {point.y} °C'},
series: [{name: 'Analysis',
type: 'scatter',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[6]
}],
});


};



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



const updateLineCharts = async ()=>{
  console.log(start.value, end.value);
if(!!start.value && !!end.value){
// Convert output from Textfield components to 10 digit timestamps
let startDate = new Date(start.value).getTime() / 1000;
let endDate = new Date(end.value).getTime() / 1000;
// Fetch data from backend
const data = await AppStore.getAllInRange(startDate,endDate);
// Create arrays for each plot
let temperature = [];
let heatindex = [];
let humidity = []
let temperatureHeatIndex = [];
let humidityHeatIndex = [];
// Iterate through data variable and transform object to format recognized by highcharts
data.forEach(row => {
temperature.push({"x": row.timestamp * 1000, "y": parseFloat(row.temperature.toFixed(2)) });
heatindex.push({ "x": row.timestamp * 1000,"y": parseFloat(row.heatindex.toFixed(2)) });
humidity.push({ "x": row.timestamp * 1000,"y": parseFloat(row.humidity.toFixed(2)) });
temperatureHeatIndex.push({"x": parseFloat(row.temperature.toFixed(2)), "y": parseFloat(row.heatindex.toFixed(2)) });
humidityHeatIndex.push({"x": parseFloat(row.humidity.toFixed(2)), "y": parseFloat(row.heatindex.toFixed(2)) });
});
// Add data to Temperature and Heat Index chart
tempHiChart.value.series[0].setData(temperature);
tempHiChart.value.series[1].setData(heatindex);
tempHiChart1.value.series[0].setData(humidity);
tempHiChart3.value.series[0].setData(temperatureHeatIndex);
tempHiChart4.value.series[0].setData(humidityHeatIndex);
}
}

const updateCards = async () => {
// Retrieve Min, Max, Avg, Spread/Range
if(!!start.value && !!end.value){
// 1. Convert start and end dates collected fron TextFields to 10 digit timestamps
let startDate = new Date(start.value).getTime() / 1000;
let endDate = new Date(end.value).getTime() / 1000;
// 2. Fetch data from backend by calling the API functions
const temp = await AppStore.getTemperatureMMAR(startDate,endDate);
const humid = await AppStore.getHumidityMMAR(startDate,endDate);
console.log(humid)
temperature.max = temp[0].max.toFixed(1);
temperature.min = temp[0].min.toFixed(1);
temperature.avg = temp[0].avg.toFixed(1);
temperature.range = temp[0].range.toFixed(1);
//3. complete for min, avg and range
//4. complete max, min, avg and range for the humidity variable
humidity.max = humid[0].max.toFixed(1);
humidity.min = humid[0].min.toFixed(1);
humidity.avg = humid[0].avg.toFixed(1);
humidity.range = humid[0].range.toFixed(1);
}
}

const updateHistogramCharts = async () => {
  // Ensure start and end dates exist before proceeding
  if (!!start.value && !!end.value) {
    // 1. Convert start and end dates from TextFields to 10-digit timestamps
    const startDate = Math.floor(new Date(start.value).getTime() / 1000); 
    const endDate = Math.floor(new Date(end.value).getTime() / 1000);

    // 2. Fetch data (temperature, humidity, and heat index) from the backend
    const temp = await AppStore.getFreqDistro("temperature", startDate, endDate);
    const humid = await AppStore.getFreqDistro("humidity", startDate, endDate);
    const hi = await AppStore.getFreqDistro("heatindex", startDate, endDate);

    // 3. Create empty arrays for each variable
    let temperature = [];
    let humidity = [];
    let heatindex = [];

    // 4. Transform temperature data into { "x": x_value, "y": y_value } format
    temp.forEach(row => {
      temperature.push({ "x": row["_id"], "y": row["count"] });
    });

    // 5. Transform humidity data
    humid.forEach(row => {
      humidity.push({ "x": row["_id"], "y": row["count"] });
    });

    // 6. Transform heat index data
    hi.forEach(row => {
      heatindex.push({ "x": row["_id"], "y": row["count"] });
    });

    tempHiChart2.value.series[0].setData(humidity, true);

    tempHiChart2.value.series[1].setData(temperature, true);

    tempHiChart2.value.series[2].setData(heatindex, true);
  }
};

</script>


<style scoped>
/** CSS STYLE HERE */

Figure {
border: 2px solid black;
}

</style>