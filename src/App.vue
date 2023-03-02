<template>
    <div class="max-w-5xl mx-auto p-20">
        <h1 class="text-4xl mb-20">Weather App</h1>

        <input type="text" v-model="search" placeholder="Choose city..." @change="getWeather" class="px-4 py-2 border rounded-lg shadow-lg focus:outline-none ">

        <!-- result card -->
        <div v-if="result" class="mt-20 flex flex-col gap-2 p-8 w-fit bg-gray-50 rounded-lg shadow-lg">
            <h2 class="text-2xl font-semibold border-b-2 border-gray-500 mb-2">{{  result.name }}</h2>
            <p>{{ result.description }}</p>
            <p>Temperature: {{ result.temp }} st C</p>
            <p>Feels like: {{ result.feels_like }} st C</p>
            <p>T min: {{ result.temp_min }} st C</p>
            <p>T max: {{ result.temp_max }} st C</p>
            <p>Humidity: {{ result.humidity }} %</p>
        </div>
        <div v-if="err" class="text-red-500 mt-3 pl-4">{{ err }}</div>

        <!-- history -->
        <div v-if="historyResults.length > 0">
            <h3 class="mb-4 mt-6 font-semibold text-xl">Recent Searches</h3>
            <div class="flex gap-4 items-center">
                <div  v-for="city in historyResults" :key="city" c>
                    <h3 class="font-semibold border rounded-lg px-4 py-2 shadow-md" @click="getCityWeather(city)">{{ city }}</h3>
                </div>
            </div>
        </div>

    </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const result = ref(null)
const search = ref('')
const err = ref(null)
const historyResults = ref([])

// https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key}
const apiKey = import.meta.env.VITE_API_KEY

async function getWeather(){
    try {
        await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${search.value}&units=metric&appid=${apiKey}`)
                    .then(response => {
                        if(response.ok){
                           return response.json()
                        } else {
                            throw new Error(response.statusText)
                        }
                    })
                    // .then(data => result.value = data.main)
                    .then(({ name, main, weather }) => {
                    //    console.log('name', name)
                    //    console.log('main', main)
                    //    console.log('weather', weather)
                        err.value = null
                        result.value = main
                        result.value['name'] = name
                        result.value['description'] = weather[0].description
                        result.value['description_short'] = weather[0].main

                        if(!historyResults.value.includes(result.value.name)){
                            historyResults.value.unshift(result.value.name)
                        } else {
                            // remove duplicate (filter out duplicate)
                            historyResults.value = historyResults.value.filter(el => el !== result.value.name )
                            // then push value at the beginning of array
                            historyResults.value.unshift(result.value.name)

                        }

                        if(historyResults.value.length > 5){
                            // only get first 5
                            historyResults.value = historyResults.value.slice(0, 5)
                        }
                    })


    } catch (error) {
        err.value = error.message
    }

}

function getCityWeather(city) {
    search.value = city
    getWeather()
}

// add debounce, dont want for every key stroke
// watch(search, () => {
//     getWeather()
// } )


</script>
