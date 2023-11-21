<template>
  <div
    id="app"
    :class="{
      'cold': typeof weather.main !== 'undefined' && weather.main.temp < 4,
      'avg': typeof weather.main !== 'undefined' && weather.main.temp >= 4 && weather.main.temp <= 16,
      'warm': typeof weather.main !== 'undefined' && weather.main.temp > 16
    }"
  >
    <main>
      <div class="language-selector">
        <button
          @click="setLanguage('es')"
          :class="currentLanguage.lang == 'es' ? 'active' : ''"
        >
          <img
            src="./assets/icons/spain.png"
            alt="Botón"
          >
        </button>
        <button
          @click="setLanguage('fr')"
          :class="currentLanguage.lang == 'fr' ? 'active' : ''"
        >
          <img
            src="./assets/icons/france.png"
            alt="Botón"
          >
        </button>
        <button
          @click="setLanguage('en')"
          :class="currentLanguage.lang == 'en' ? 'active' : ''"
        >
          <img
            src="./assets/icons/britain.png"
            alt="Botón"
          >
        </button>
        <button
          @click="setLanguage('it')"
          :class="currentLanguage.lang == 'it' ? 'active' : ''"
        >
          <img
            src="./assets/icons/italy.png"
            alt="Botón"
          >
        </button>
      </div>

      <div class="search-box">
        <input
          type="text"
          class="search-bar"
          :placeholder="currentLanguage.placeholder"
          v-model="query"
          @keypress="handleKeyPress"
        >
        <button
          type="button"
          @click="fetchWeather"
        >
          <img
            src="./assets/icons/search.png"
            alt="search"
          >
        </button>
      </div>

      <div v-if="weather.main">
        <div class="weather-wrapp">
          <div class="location-box">
            <div class="location">
              {{ `${weather.name}, ${weather.sys.country}` }}
            </div>
            <div class="date">
              {{ dateBuilder() }}
            </div>
          </div>

          <div class="weather-box">
            <div class="temp">
              {{ `${Math.round(weather.main.temp)}º` }}
            </div>
            <div class="weather">
              {{ capitalizeFirstLetter(weather.weather[0].description) }}
            </div>
          </div>
        </div>
      </div>

      <div
        v-if="queryError"
        class="error"
      >
        <img
          :src="urlError"
          alt=""
          class="error-image"
        >
      </div>
    </main>
  </div>
</template>

<script setup>
import {ref} from 'vue';

import es from '@/locales/es.json';
import en from '@/locales/en.json';
import fr from '@/locales/fr.json';
import it from '@/locales/it.json';

const apiKey = '67cb310b44ecf636039acc3a0c10caea';
const urlBase = 'https://api.openweathermap.org/data/2.5/';
const query = ref('');
const weather = ref({});
const currentLanguage = ref(es);
const queryError = ref(false);
const urlError = ref('');

const fetchWeather = async (e) => {
  try {
    const response = await fetch(`${urlBase}weather?q=${query.value}&units=metric&lang=${currentLanguage.value.lang}&APPID=${apiKey}`);

    if (!response.ok) {
      weather.value = '';
      queryError.value = true;
      urlError.value = `https://http.cat/${response.status}`;
      throw new Error(`Error ${response.status}: ${response.statusText}`);
    }

    const results = await response.json();
    weather.value = results;
    queryError.value = false;
  } catch (error) {
    console.error('Error fetching weather data:', error.message);
  }
};

const dateBuilder = () => {
  const d = new Date();

  const day = currentLanguage.value.days[d.getDay()];
  const date = d.getDate();
  const month = currentLanguage.value.months[d.getMonth()];
  const year = d.getFullYear();

  return `${day} ${date} ${month} ${year}`;
};

const setLanguage = (e) => {
  switch (e) {
    case 'es':
      currentLanguage.value = es;
      break;
    case 'en':
      currentLanguage.value = en;
      break;
    case 'fr':
      currentLanguage.value = fr;
      break;
    case 'it':
      currentLanguage.value = it;
      break;

    default:
      break;
  }
};

const capitalizeFirstLetter = (string) =>{
  return string.charAt(0).toUpperCase() + string.slice(1);
};

const handleKeyPress = (e) => {
  if (e.key === 'Enter') {
    fetchWeather();
  }
};

</script>


<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'montserrat', sans-serif;
}

#app {
  background-image: url('./assets/avg-bg.jpg');
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}

#app.warm {
  background-image: url('./assets/warm-bg.jpg');
}

#app.cold {
  background-image: url('./assets/cold-bg.jpg');
}

main {
  min-height: 100vh;
  padding: 25px;

  background-image: linear-gradient(to bottom,
      rgba(0, 0, 0, 0.25),
      rgba(0, 0, 0, 0.75));
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
  position: relative;
}

.search-box button {
  position: absolute;
  right: 10px;
  top: 10px;
  height: 40px;
  background-color: transparent;
  border: none;
}

.search-box img {
  height: 30px;
  opacity: 0.8;
}

.search-box img:hover {
  height: 35px;
  transition: 0.4s;
  cursor: pointer;
  opacity: 1;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;

  color: #313131;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;

  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}

.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.location-box .location {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
  color: #FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}

.weather-box {
  text-align: center;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;

  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #FFF;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.language-selector {
  height: 50px;
  align-items: center;
}

.language-selector button {
  border-radius: 50%;
  border: none;
  background-color: transparent;
  cursor: pointer;
}

.language-selector img {
  width: 35px;
  opacity: 0.6;
}

.language-selector .active img {
  opacity: 1
}

.language-selector img:hover {
  opacity: 1;
  transition: 0.4s;
}
.error{
  text-align: center;
}
.error-image{
  width: 100%;
  max-width: 500px;
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  border-radius: 16px 0px 16px 0px;
}
</style>
