<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" placeholder="Search for a city or state"
      v-model="searchQuery"
      @input="getSearchResults"
      class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-socondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      >
      <ul 
      v-if="mapboxSearchResults"
      class="absolute bg-weather-secondary text-white w-full shadow-sm py-2 px-1 top-[66px]">
        <p v-if="searchError">Sorry something went wrong</p>
        <p v-if="!searchError && mapboxsearchResults.lenght == 0">No match results</p>
        <template v-else>
            <li v-for="searchResult in mapboxSearchResults" 
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)">
          
            {{ searchResult.place_name }}
          </li>
        </template>
        
        
      </ul>
    </div>
 
  </main>
</template>

<script setup>
    import { ref } from "vue";
    import axios from "axios";
    import {userRouter} from 'vue-router';
import router from "../router";

    const previewCity = (searchResult) => {
      const [city,state] = searchResult.place_name.split(",");
      router.push({
        name: 'cityView',
        params: {state: state.replaceAll(" ",""), city: city},
        query : {
          lat: searchResult.geometry.coordinates[1],
          lng: searchResult.geometry.coordinates[0],
          preview: true
        }
      })
    };

    const searchQuery = ref("");
    const queryTimeout = ref(null);
    const mapboxSearchResults = ref(null);
    const searchError = ref(null);

    const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";

    const getSearchResults = () => {
      clearTimeout(queryTimeout.value);
      queryTimeout.value = setTimeout(async () => {
        if (searchQuery.value != ''){
          try {
            const result = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
            );
            mapboxSearchResults.value = result.data.features;
          }
          catch {
            searchError.value= true;
          }

          return;
        }
        mapboxSearchResults.value=null;
      }, 300);
    };
</script>