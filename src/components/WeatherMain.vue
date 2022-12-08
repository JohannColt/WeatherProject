<template>
<div class="container">
  <div class="header">
    <h1>
      Welcome to my weather app!
    </h1>
  </div>
  <div class="body">
      <ul class="list">
        <li class="list__component">
          <div class="list__element">City</div>
          <div class="list__element blue_back">Precipitation</div>
          <div class="list__element">Max Temperature</div>
          <div class="list__element blue_back">Apparent Max Temperature</div>
          <div class="list__element">Min Temperature</div>
          <div class="list__element blue_back">Apparent Min Temperature</div>
          <div class="list__element">Precipitation Sum</div>
          <div class="list__element blue_back">Wind Speed</div>
          <div class="list__element">Wind Direction</div>
          <div class="list__element blue_back"></div>
        </li>
       <li class="list__component" v-for="component in components" :key="component.city">
        <div class="list__element">{{component.city}}</div>
        <div class="list__element blue_back">{{component.precipitation}}</div>
         <div class="list__element">{{component.maxTemperature}}</div>
         <div class="list__element blue_back">{{component.maxAppTemperature}}</div>
         <div class="list__element">{{component.minTemperature}}</div>
         <div class="list__element blue_back">{{component.minAppTemperature}}</div>
         <div class="list__element">{{component.precipitationSum}}</div>
         <div class="list__element blue_back">{{component.windSpeed}}</div>
         <div class="list__element">{{component.windDirection}}</div>
         <div class="list__element blue_back">
           <button v-on:click="removeComponent(component.city)" class="button button__third"><img class="remove-icon" src="./icons/remove-icon.png"></button>
         </div>
       </li>
        <li class="list__component button__section">
          <div class=" list__element add-button">
            <button class="button button__first" v-if="!inputVision" v-on:click="inputVision=!inputVision">Add city</button>
            <button class="button button__second" v-if="inputVision" v-on:click="addNewComponent">Submit</button>
          </div>
          <div v-if="inputVision" class="list__element add-input">
            <input class="input"  v-model="cityName" placeholder="Write your city">
          </div>
        </li>
      </ul>

  </div>
</div>
</template>

<script>
import axios from "axios";

export default {
  name: "WeatherMain",
  data() {
    return {
      apiInfo: null,
      cityName: null,
      inputVision: false,
      components: []
    }
  },
  methods: {
     async addNewComponent() {
      this.inputVision = !this.inputVision
       let weatherData = await this.takeWeather();
      this.components.push({city: this.cityName,precipitation:this.weatherInterpretation(weatherData.weathercode[0]),
        maxTemperature:weatherData.temperature_2m_max[0]+"°C",maxAppTemperature:weatherData.apparent_temperature_max[0]+"°C",
        minTemperature:weatherData.temperature_2m_min[0]+"°C",minAppTemperature:weatherData.apparent_temperature_min[0]+"°C",
        precipitationSum:weatherData.precipitation_sum[0]+"mm",windSpeed: weatherData.windspeed_10m_max[0]+"km/h",
        windDirection:this.windDirection(weatherData.winddirection_10m_dominant[0])
      })
    },
    async takeGeoData() {
      let info;
      await axios.get('https://geocoding-api.open-meteo.com/v1/search?name=' + this.cityName + '&count=1').then(response => (info = response));
      return {latitude: info.data.results[0].latitude, longitude: info.data.results[0].longitude}
    },
    async takeWeather() {
      let today = this.takeDate();
      let geolocation = this.takeGeoData();
      let info;
      await axios.get('https://api.open-meteo.com/v1/forecast?latitude=' + (await geolocation).latitude + '&longitude=' + (await geolocation).longitude + '&daily=weathercode,temperature_2m_max,temperature_2m_min,apparent_temperature_max,apparent_temperature_min,sunrise,sunset,precipitation_sum,rain_sum,showers_sum,snowfall_sum,precipitation_hours,windspeed_10m_max,winddirection_10m_dominant&timezone=Europe%2FBerlin&start_date=' + today + '&end_date=' + today).then(response => (info = response));
      return info.data.daily
    },
    takeDate() {
      let today = new Date();
      let dd = String(today.getDate()).padStart(2, '0');
      let mm = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!
      let yy = today.getFullYear();
      today = yy + '-' + mm + '-' + dd;
      return today;
    },
    weatherInterpretation(a) {
      if (a === 0) {
        return 'Clear sky'
      }
      if (a === 1) {
        return 'Mainly clear'
      }
      if (a === 2) {
        return 'Partly cloudy'
      }
      if (a === 3) {
        return 'Overcast'
      }
      if (a === 45) {
        return 'Fog'
      }
      if (a === 48) {
        return 'Depositing rime fog'
      }
      if (a === 51) {
        return 'Light drizzle'
      }
      if (a === 53) {
        return 'Moderate drizzle'
      }
      if (a === 55) {
        return 'Dense drizzle'
      }
      if (a === 56) {
        return 'Light freezing drizzle'
      }
      if (a === 57) {
        return 'Dense freezing drizzle'
      }
      if (a === 61) {
        return 'Slight rain'
      }
      if (a === 63) {
        return 'Moderate rain'
      }
      if (a === 65) {
        return 'Dense rain'
      }
      if (a === 66) {
        return 'Light freezing rain'
      }
      if (a === 67) {
        return 'Heavy freezing rain'
      }
      if (a === 71) {
        return 'Slight snowfall'
      }
      if (a === 73) {
        return 'Moderate snowfall'
      }
      if (a === 75) {
        return 'Heavy snowfall'
      }
      if (a === 77) {
        return 'Snow grains'
      }
      if (a === 80) {
        return 'Slight rein shower'
      }
      if (a === 81) {
        return 'Moderate rein shower'
      }
      if (a === 82) {
        return 'Violent rein shower'
      }
      if (a === 85) {
        return 'Slight snow shower'
      }
      if (a === 86) {
        return 'Heavy snow shower'
      }
      if (a === 95) {
        return 'Slight or moderate thunderstorm'
      }
      if (a === 96) {
        return 'Thunderstorm with slight hail'
      }
      if (a === 99) {
        return 'Thunderstorm with heavy hail'
      }
    },
    windDirection(a) {
       if (a >=337 || a <= 23){
         return "N";
       }
       if(a>=24 && a<=68){
          return "NE";
       }
      if(a>=69 && a<=113){
        return "E";
      }
      if(a>=114 && a<=158){
        return "SE";
      }
      if(a>=159 && a<=203){
        return "S";
      }
      if(a>=204 && a<=248){
        return "SW";
      }
      if(a>=249 && a<=293){
        return "W";
      }
      if(a>=294 && a<=336 ){
        return "NW";
      }
    },
    removeComponent (a){
      this.components = this.components.filter(function (obj){
        return obj.city !== a;
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.container{
  max-width: 1600px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: 18px;
}
.header{
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100vw;
  height: 150px;
  background-color: #00FA9A;
}
.body{
  padding-top: 15px;
}
.list{
  display: grid;
  grid-template-columns: 1fr;
  padding: 0;
  &__component{
    padding-left: 10px;
    padding-right: 10px;
    display: grid;
    grid-template-columns: repeat(10, 1fr);

  }
  &__element{
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding-left: 10px;
    padding-right: 10px;
    border: solid 1px rgba(0, 0, 0, .1);

  }
}
.remove-icon{
  width:15px;
  height: 15px;
}
.add-button{
  padding: 0;
  border: 0;
}
.add-input{
  border: solid 1px #00CED1;
  border-radius: 6px;

}
.button{
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  border: 0;
  font-size: 18px;
  font-weight: bold;
  padding-top: 5px;
  padding-bottom: 5px;
  color: #ffffff;
  border-radius: 6px;

  &__section{
    padding-top: 10px;
  }
  &__first{
    background-color: #00FF7F	;
  }
  &__second{
    background-color: #00CED1;
  }
  &__third{
    background-color: #CCF5F6;
  }
}
.input{
  width: 100%;
  height: 100%;
  border: 0;
  background-color: #ffffff;
  font-size: 12px;
}
.blue_back{
  background-color: #CCF5F6;
}
</style>