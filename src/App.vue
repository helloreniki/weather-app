<template>
    <div class="h-screen mx-auto flex justify-center items-center bg-gradient-to-tr from-sky-200 to-sky-400">
        <div class="px-12 py-20 max-w-2xl w-full flex flex-col items-center">
            <h1 class="text-4xl pb-12 text-center text-sky-900 leading-relaxed font-bold">Weather App</h1>

            <input type="text" v-model="search" placeholder="Choose city..." @change="getWeather" class="placeholder-sky-500 max-w-md mx-auto w-full px-4 py-2 border border-sky-300 opacity-70 rounded-lg shadow-sky-300 shadow-md text-sky-700 font-semibold focus:outline-none ">

            <!-- result card -->
            <div v-if="result" class="mt-20 flex flex-col gap-2 px-16 py-12 w-fit bg-white/50  rounded-lg shadow-lg shadow-sky-300 text-sky-600">
                <h2 class="text-3xl font-semibold border-b-2 border-sky-800 mb-2 text-sky-800">{{ result.name }}</h2>
                <p class="text-xl text-sky-800  ">{{ result.description }}</p>
                <p class="text-3xl font-bold text-sky-800 my-3"> {{ result.temp }} °C </p>
                <p>Feels like: {{ result.feels_like }} °C</p>
                <div class="flex gap-5 items-center">
                    <p>T min: {{ result.temp_min }} °C</p>
                    <p>T max: {{ result.temp_max }} °C</p>
                    <p>Humidity: {{ result.humidity }} %</p>
                </div>
            </div>
            <div v-if="err" class="text-red-500 mt-3 pl-4">{{ err }}</div>

            <!-- history results view -->
            <div v-if="historyResults.length > 0">
                <h3 class="mb-6 mt-12 font-semibold text-2xl text-sky-800">Recent Searches</h3>
                <div class="flex gap-4 items-center">
                    <div  v-for="city in historyResults" :key="city" c>
                        <h3 class="font-semibold text-sky-800 border border-sky-300 rounded-lg px-4 py-2 shadow-md shadow-sky-300 bg-white/50" @click="getCityWeather(city)">{{ city }}</h3>
                    </div>
                </div>
            </div>

        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const result = ref(null)
const search = ref('')
const err = ref(null)
const historyResults = ref([])

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
            .then(({ name, main, weather }) => {
                err.value = null
                result.value = main
                result.value['name'] = name
                result.value['description'] = weather[0].description
                result.value['description_short'] = weather[0].main

                if(!historyResults.value.includes(result.value.name)){
                    historyResults.value.unshift(result.value.name)
                } else {
                    // remove duplicate value (filter out duplicate)
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

</script>
