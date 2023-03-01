<template>
    <div class="max-w-5xl mx-auto p-20">
        <h1 class="text-4xl mb-20">Weather App</h1>

        <input type="text" v-model="search" placeholder="Choose city..." @change="getWeather" class="px-4 py-2 border rounded-lg shadow-lg focus:outline-none ">

        <!-- result card -->
        <div v-if="weather" class="my-20 flex flex-col gap-2 p-8 w-fit bg-gray-50 rounded-lg shadow-lg">
            <h2 class="text-2xl font-semibold border-b-2 border-gray-500 mb-2">{{  weather.city }}</h2>
            <p>{{ weather.description?.description }}</p>
            <p>Temperature: {{ weather.main.temp }} st C</p>
            <p>Feels like: {{ weather.main.feels_like }} st C</p>
            <p>T min: {{ weather.main.temp_min }} st C</p>
            <p>T max: {{ weather.main.temp_max }} st C</p>
            <p>Huimdity: {{ weather.main.humidity }} st C</p>
        </div>
        <div v-if="err" class="text-red-500 mt-3 pl-4">{{ err }}</div>

    </div>
</template>

<script setup>
import { ref } from 'vue'

const weather = ref(null)
const search = ref('')
const err = ref(null)

// https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key}
const apiKey = import.meta.env.VITE_API_KEY

async function getWeather(){
    try {
        await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${search.value}&units=metric&appid=${apiKey}`)
                    .then(response => {
                        console.log(response)
                        if(response.ok){
                           return response.json()
                        } else {
                            throw new Error(response.statusText)
                        }
                    })
                    // .then(data => weather.value = data.main)
                    .then(data => {
                       console.log(data)
                       err.value = null
                       if(data) {
                           weather.value = []
                           weather.value['city'] = data.name
                           weather.value['main'] = data.main
                           weather.value['description'] = data.weather[0]
                       }
                    })

        console.log('weather', weather.value)


    } catch (error) {
        err.value = error.message
    }

}





</script>
