<template>
    <div class="app">
        <section class="video-section">
            <video autoplay loop muted>
                <!-- <source src="@/assets/videos/spring_video.webm" type="video/webm" /> -->
                <!-- <source src="@/assets/videos/summer_video.webm" type="video/webm" /> -->
                <!-- <source src="@/assets/videos/winter_video.webm" type="video/webm" /> -->
                <source src="@/assets/videos/autumn_video.webm" type="video/webm" />
                Your browser does not support the video tag.
            </video>
        </section>
        <section class="content-wrapper">
            <section class="content-section">
                <section class="content-section_data">
                    <div class="weather-card">
                        <input
                            @input="setSearch()"
                            type="text"
                            v-model="searchValue"
                            class="weather-card_search--bar"
                            placeholder="cidade, Pais"
                        />

                        <div class="option-container" v-if="sugestion">
                            <option
                                @click="
                                    () => {
                                        this.searchValue = city.name + ', ' + city.country
                                        this.sugestion = false
                                        this.lat = city.lat
                                        this.lng = city.lng
                                        getWeather()
                                    }
                                "
                                v-for="city in filteredCity"
                                :key="city.lat + city.lng"
                            >
                                {{ city.name }}, {{ city.country }}
                            </option>
                        </div>

                        <div v-if="weatherCity" class="weather-card_local--weather">
                            <div class="weather-card_local--weather_view">
                                <p class="weather-card_local--weather_view--location">
                                    {{ weatherCity.location.name }},
                                    {{ weatherCity.location.country }}
                                </p>
                                <p class="weather-card_local--weather_view--current_temp">
                                    {{ weatherCity.current.temp_c }} <span><sup>o</sup>C</span>
                                </p>
                                <p class="weather-card_local--weather_view--feels_temp">
                                    Feels {{ weatherCity.current.feelslike_c }}
                                    <span><sup>o</sup>C</span>
                                </p>
                                <p class="weather-card_local--weather_view--condition">
                                    {{ weatherCity.current.condition.text }}
                                    <img class="weather-icon" :src="weatherIcon" alt="icone" />
                                </p>
                            </div>
                            <div class="weather-card_local--weather_chart">
                                <div class="button-container">
                                    <button @click="activateTemp()">Temperatura</button>
                                    <button @click="activateWind()">Vento</button>
                                    <button @click="activateHumidity()">Umidade</button>
                                </div>
                                <LineChart
                                    v-if="this.showChart && this.tempChart"
                                    label1="Temperatura"
                                    label2="Sensação Térmica"
                                    :maxTemp="this.maxTemp"
                                    :minTemp="this.feels"
                                    :labels="this.labels"
                                ></LineChart>
                                <LineChart
                                    v-if="this.showChart && this.windChart"
                                    label1="Vento (KM/h)"
                                    :maxTemp="this.wind"
                                    :labels="this.labels"
                                ></LineChart>
                                <LineChart
                                    v-if="this.showChart && this.humidityChart"
                                    label1="Umidade (%)"
                                    :maxTemp="this.humidity"
                                    :labels="this.labels"
                                ></LineChart>
                            </div>
                        </div>
                        <div v-else>
                            <p>oi</p>
                        </div>
                    </div>

                    <!-- este container terá informaçoes dos próximos 3 dias -->
                    <section class="next-days-container">
                        <div
                            v-for="(card, index) in nextDays"
                            :key="index"
                            class="next-days-container_card"
                        >
                            <p>
                                {{ card.date }}
                            </p>
                            <img :src="`http:${card.day.condition.icon}`" alt="tipo do clima" />
                            <div class="next-days_card--info">
                                <div class="left">
                                    <p>
                                        {{ card.day.maxtemp_c }}
                                        <span>{{ card.day.mintemp_c }}</span>
                                    </p>
                                    <p>{{ card.day.condition.text }}</p>
                                </div>
                                <div class="right">
                                    <p>Umidade {{ card.day.avghumidity }} %</p>
                                    <p>Vento {{ card.day.maxwind_kph }} Km/h</p>
                                </div>
                            </div>
                        </div>
                    </section>
                </section>
            </section>
            <section class="news-section">
                Weather news

                <div v-for="(article, index) in weatherNews" :key="index" class="news-card">
                    <div class="news-card_left">
                        {{ article.title }}
                        <div class="news-card_left_author">
                            {{ article.author }}
                        </div>
                    </div>
                    <div v-if="article.urlToImage" class="news-card_right">
                        <img :src="article.urlToImage" alt="foto da noticia" />
                    </div>
                </div>
            </section>
        </section>
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import cities from 'cities.json'
import axios from 'axios'
import LineChart from '@/components/LineChart.vue'

export default defineComponent({
    name: 'App',
    components: {
        LineChart
    },
    data() {
        return {
            searchValue: '',
            city: cities,
            searchedCity: '',
            searchedCountry: '',
            sugestion: false,
            lat: '',
            lng: '',
            weatherCity: null,
            weatherIcon: '',
            hourlyWeather: null,
            todayDate: '',
            labels: [] as string[],
            maxTemp: [] as number[],
            feels: [] as number[],
            humidity: [] as number[],
            wind: [] as number[],
            showChart: false,
            tempChart: true,
            windChart: false,
            humidityChart: false,
            nextDays: null,
            weatherNews: null
        }
    },
    computed: {
        filteredCity(): object[] | null {
            let citySearch = this.searchedCity
            let country = this.searchedCountry

            if ((citySearch || country) && this.searchValue.length > 2) {
                const search: object[] | null = this.city.filter((city) => {
                    if (city.name.toLowerCase() === citySearch.toLowerCase()) {
                        return city.name.toLowerCase() === citySearch.toLowerCase()
                    } else {
                        return city.name.toLowerCase().includes(citySearch.toLowerCase())
                    }
                })

                return search
            }
            return null
        },
        splitSearch(): string[] {
            return this.searchValue.split(',').map((item) => {
                return item.trim()
            })
        }
    },
    methods: {
        activateTemp(): void {
            this.tempChart = true
            this.windChart = false
            this.humidityChart = false
        },
        activateWind(): void {
            this.tempChart = false
            this.windChart = true
            this.humidityChart = false
        },
        activateHumidity(): void {
            this.tempChart = false
            this.windChart = false
            this.humidityChart = true
        },
        setSearch(): void {
            this.searchedCity = this.splitSearch[0]
            this.searchedCountry = this.splitSearch[1]
            if (this.searchValue.length > 2) {
                this.sugestion = true
            } else {
                this.sugestion = false
            }
        },
        getWeather(): void {
            // eslint-disable-next-line
            let self = this
            if (this.lat && this.lng) {
                // GET CURRENT WEATHER
                axios
                    .get(
                        `http://api.weatherapi.com/v1/current.json?key=79623225772a466e966223014220507&q=${this.lat},${this.lng}&aqi=no`
                    )
                    .then(function (response) {
                        // eslint-disable-next-line
                        self.weatherCity = response.data
                        self.weatherIcon = `http:${response.data.current.condition.icon}`

                        let formatedDate = response.data.location.localtime
                        let formatedDay = formatedDate.split(' ')
                        self.todayDate = formatedDay[0]
                    })
                    .then(() => {
                        //GET DAY HOURLY WEATHER AFTER CONVERTING TIMEZONE
                        axios
                            .get(
                                `http://api.weatherapi.com/v1/history.json?key=79623225772a466e966223014220507&q=${this.lat},${this.lng}&dt=${this.todayDate}`
                            )
                            .then(function (response) {
                                // eslint-disable-next-line
                                self.hourlyWeather = response.data.forecast

                                let allHours: [] | null = response.data.forecast.forecastday[0].hour

                                interface timeObject {
                                    time: string
                                    temp_c: number
                                    feelslike_c: number
                                    wind_kph: number
                                    humidity: number
                                }

                                let hourLabels: string[] = []
                                let tempMax: number[] = []
                                let feelsLike: number[] = []
                                let windNum: number[] = []
                                let humidityNum: number[] = []
                                allHours?.forEach((item: timeObject) => {
                                    let hour = item.time.split(' ')
                                    hourLabels.push(hour[1])
                                    tempMax.push(item.temp_c)
                                    feelsLike.push(item.feelslike_c)
                                    windNum.push(item.wind_kph)
                                    humidityNum.push(item.humidity)
                                })

                                self.labels = hourLabels
                                self.maxTemp = tempMax
                                self.feels = feelsLike
                                self.wind = windNum
                                self.humidity = humidityNum
                            })
                            .then(() => {
                                // GET NEXT 3 DAYS
                                axios
                                    .get(
                                        `http://api.weatherapi.com/v1/forecast.json?key=79623225772a466e966223014220507&q=${this.lat},${this.lng}&days=3&aqi=no&alerts=no`
                                    )
                                    .then(function (response) {
                                        // eslint-disable-next-line
                                        self.nextDays = response.data.forecast.forecastday
                                    })
                            })
                            .then(() => {
                                //GET NEWS
                                let splitSearchedCountry = self.searchValue.split(', ')
                                let searchedCountry = splitSearchedCountry[1].toLowerCase()

                                axios
                                    .get(
                                        `https://newsapi.org/v2/top-headlines?country=${searchedCountry}&apiKey=3dc6c71200d84dd3a30ca3c0a1254980`
                                    )
                                    .then(function (response) {
                                        // eslint-disable-next-line
                                        self.weatherNews = response.data.articles
                                    })
                            })
                            .then(() => {
                                self.showChart = true
                            })
                    })
            }
        }
    }
})
</script>

<style lang="scss">
@import '@/assets/scss/global_style';
</style>
