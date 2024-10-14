<script setup>
import { ref } from "vue";

const searchQuery = ref("");
const queryTimeOut = ref(null);
const searchQueryResults = ref([]);
const selectedWeather = ref(null);

const weatherAPIKey = "e1bba6c4e110f91d7ae5d03fab3c88dd";

const getSearchResults = () => {
  clearTimeout(queryTimeOut.value);

  queryTimeOut.value = setTimeout(async () => {
    if (searchQuery.value.trim()) {
      try {
        const result = await fetch(
          `https://api.openweathermap.org/geo/1.0/direct?q=${encodeURIComponent(
            searchQuery.value
          )}&limit=5&appid=${weatherAPIKey}`
        );
        const data = await result.json();
        searchQueryResults.value = data;
      } catch (error) {
        console.error("Error fetching search results:", error);
      }
    } else {
      searchQueryResults.value = [];
    }
  }, 300);
};

const getWeather = async (lat, lon) => {
  try {
    if (searchQuery !== "") {
      const result = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${weatherAPIKey}&units=metric`
      );
      const data = await result.json();
      selectedWeather.value = {
        temp: data.main.temp,
        description: data.weather[0].description,
        city: data.name,
        country: data.sys.country,
        icon: data.weather[0].icon,
      };
    } else {
      selectedWeather.value = null;
    }
  } catch (error) {
    console.error("Error fetching weather data:", error);
  }
};

</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a City or State..."
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-primary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]" />

      <div v-if="searchQueryResults.length > 0">
        <div
          v-for="place in searchQueryResults"
          :key="place.lat + place.lon"
          class="py-1 px-2 text-xs text-weather-primary hover:text-weather-secondary cursor-pointer">
          <button @click="getWeather(place.lat, place.lon)" class="text-white">
            {{ place.name }}, {{ place.country }}
          </button>
        </div>
      </div>

      <div v-if="selectedWeather" class="mt-4 text-center">
        <div class="mt-4">
          <img
            :src="`http://openweathermap.org/img/wn/${selectedWeather.icon}@2x.png`"
            alt="Weather Icon"
            class="w-24 h-24 mx-auto" />
        </div>

        <h3 class="text-xl mt-2">
          {{ selectedWeather.city }}, {{ selectedWeather.country }}
        </h3>
        <p class="text-lg">Temperature: {{ selectedWeather.temp }}°C</p>
        <p class="text-lg">Condition: {{ selectedWeather.description }}</p>
      </div>
    </div>
  </main>
</template>

<style scoped></style>
