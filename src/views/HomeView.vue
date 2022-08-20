<template>
  <main class="container text-white ">
    <div class="pt-4 mb-8 relative">
      <input type="text" 
      @input="getSearchResults"
      v-model="searchQuery"
      placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b 
        focus:border-weather-secondary focus:outline-none 
        focus:shadow-[0px_1px_0_0_#004E71]">
        <ul 
        v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
        <p v-if="searchError">Sorry an error occure please try again</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
        No result found try another search please !
        </p>
          <template v-else>
            <li  @click="previewCity(searchResult)" v-for="searchResult in mapboxSearchResults" :key="searchResult.id" class="py-2 cursor-pointer">
            {{searchResult.place_name}}
            </li>
          </template>
        </ul>
    </div>
  </main>
</template>
<script setup>
import { ref } from 'vue';
import axios from 'axios'
import { useRouter } from 'vue-router';

const router = useRouter();
const mapboxAPIkey = 'pk.eyJ1Ijoic29mdHJhcGgiLCJhIjoiY2w3MHRzbDJ6MGh1YjN4czl2dnhydm51ZyJ9.S9VPQdd9-q_X0pqFX1ITIw'
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);
const previewCity = (searchResult) => {
  console.log(searchResult);

  const [city,state] = searchResult.place_name.split(",");
  router.push({
    name:"cityView",
    params:{state: state.replaceAll(" ","") , city: city},
    query:{
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    }
  })
};

const getSearchResults = () =>{
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout( async() => {
    if(searchQuery.value !=""){
      try{
          const result = await axios.get
      (`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIkey}&type=place`);
      mapboxSearchResults.value = result.data.features;
      // console.log(mapboxSearchResults.value)
      } catch{
        searchError.value = true
      }
      return;
    }
    mapboxSearchResults.value = null
  }, 300);
}
</script>
