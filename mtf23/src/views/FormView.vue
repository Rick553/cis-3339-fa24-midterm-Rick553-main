<template>
    <div>
        <form @submit.prevent="fetchRemoteData">
            
            <div class="form-group">
                <h2> Weather Data </h2>
                <p> Please fill in the form and use the submit button to load data.</p>
                <label for="">Location</label>
                <br>
                <!-- The input field for the location, bound to the data_inputted.location property -->
                <input class="form-control rounded" v-model="formInput.location" type="text" required />
            </div>
            
            <div class="form-group">
                <label for="">Start Date</label>
                <br>
                <!-- The input field for the start date, bound to the data_inputted.start_dt property -->
                <input class="form-control rounded" type="date" v-model="formInput.start_dt" required />
            </div>
            
            <div class="form-group">
                <label for="">End Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="formInput.end_dt" required />
            </div>
            <br>
            <div>
                <button class="btn btn-danger" type="submit">Submit</button>
            </div>
        </form>
        <br /><br />

        <div v-if="dataLoaded">
            <Bar :data="chartData" :options="chartOptions" />
        </div>

        <div v-if="dataLoaded">
            <hr>
            <h5> Weather Data in a Table</h5>
            <table class="table">
                <thead>
                    <tr>
                        <th v-for="adate in keys" :key="adate">{{ adate }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td v-for="atemp in values" :key="atemp">{{ atemp }}</td>
                    </tr>
                </tbody>
            </table>            
        </div>

    </div>
</template>

<script>

//One of the first things we need to do is get ref from vue and axios from axios

import { ref } from 'vue';
import axios from "axios";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
} from 'chart.js';
import { Bar } from 'vue-chartjs';

//making sure we are getting the necessary chartjs components
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

export default {
  components: {
    Bar,
  },
  //Using the setup function to encapsulate the logic and state management
  setup() {
    // Replaced data properties of options with reactive state using ref for the form input fields
    const formInput = ref({
      start_dt: "",
      end_dt: "",
      location: "",
    });
    //replaced chart options with a reactive reference using ref
    const chartOptions = ref({
      responsive: true,
      maintainAspectRatio: true,
      scales: {
        x: {
          title: {
            display: true,
            text: "Time",
          },
        },
        y: {
          title: {
            display: true,
            text: "Temperature (F)",
          },
        },
      },
    });

    //Changed the chartdata configurtion using ref the same way as the previous ones
    const chartData = ref({
      labels: [],
      datasets: [
        {
          label: "Average Temperature",
          backgroundColor: "#f87979",
          data: [],
        },
      ],
    });
    //Defining reactive state to track whether or not the data has been loaded
    const dataLoaded = ref(false);
    const keys = ref([]);
    const values = ref([]);

    // The fetchRemoteData function is set up to handle the API requests and allow better integration
    const fetchRemoteData = async () => {
      const options = {
        method: 'GET',
        url: 'https://weatherapi-com.p.rapidapi.com/history.json',
        params: {
          q: formInput.value.location,
          dt: formInput.value.start_dt,
          end_dt: formInput.value.end_dt,
          lang: 'en'
        },
        headers: {
          'X-RapidAPI-Key': '6c6f790650msh5ce4da2fced77a7p1b1776jsn247d9f531b74',
          'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
        }
      };
      
      //Making the api request, processing the repsonse and updating the chart data
      try {
        const resp = await axios.request(options);
        const w_data = resp.data.forecast.forecastday;
        keys.value = w_data.map((item) => item.date);
        values.value = w_data.map((item) => item.day.avgtemp_f);

        chartData.value = {
          labels: keys.value,
          datasets: [
            {
              label: "Average Temperature",
              backgroundColor: "#f87979",
              data: values.value,
            },
          ],
        };
        //This is to indicate that the data was loaded
        dataLoaded.value = true;
      } catch (error) {
        console.error(error);
      }
    };

    // Returns all reactive state variables and methods from the setup function
    return {
      formInput,
      chartOptions,
      chartData,
      dataLoaded,
      keys,
      values,
      fetchRemoteData,
    };
  },
};
</script>


//References used is ChatGPT
//I asked "How to change from Options API to Composition API
//I asked "Can you look through my code and find the syntax errors"
