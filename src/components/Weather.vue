<script lang="ts">
    import axios from "axios";

    export default {
        name: 'Weather',
        data () {
            return {
                currentClimate: {},
                currentTime: null,
                weatherIcon: '10d@2x.png',
                weatherIconURL: 'https://openweathermap.org/img/wn/',
                searchQuery: '',
                showError: false,
                days: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
                errorText: 'Unknown Error',
                apiKey: '0dfa2691b0e082a2f2aa34390b28f499',
                apiEndpoint: 'https://api.openweathermap.org/data/2.5/'
            }
        },
        methods: {
            refreshWithNewClimateData (newClimateData: object): void {
                this.currentClimate = newClimateData;
                this.weatherIcon = newClimateData['weather'][0]['icon'] + '@2x.png';
                this.currentTime = this.formatTimestamp(newClimateData['timezone']);
            },

            hideErrorMessage (): void {
                this.showError = false;
            },

            displayErrorMessage (newErrorText: string = 'Error 404'): void {
                this.errorText = newErrorText;
                this.showError = true;
            },

            searchCityForClimateData (event: object): void {
                if (event['key'] === 'Enter') {
                    axios.get(`${this.apiEndpoint}weather?q=${this.searchQuery}&APPID=${this.apiKey}`).then(response => {
                        console.log(response);

                        // HIDE ERROR IF DISPLAYED AND REFRESH PAGE WITH NEW CLIMATE DATA
                        this.hideErrorMessage();
                        this.refreshWithNewClimateData(response['data']);
                    }).catch(error => {
                        console.log(error);

                        // SHOW ERROR ON PAGE
                        this.displayErrorMessage(error['message']);
                    });
                }
            },

            formatTimestamp (locationTimezone: number): string {
                const currentDate: Date = new Date();
                const locationDate: Date = new Date((currentDate.getTime() + currentDate.getTimezoneOffset() * 60000) + (1000 * locationTimezone));

                return `${this.days[locationDate.getDay()]} ${locationDate.toLocaleString()}`;
            }
        }
    }
</script>

<template>
    <div class='outercontainer'>
        <main class='innercontainer'>
            <div class='errortext_outercontainer' v-if="showError">
                <div class='errortext_innercontainer'>
                    <span class='errortext'>{{ errorText }}</span>
                </div>
            </div>

            <div class="searchbox_container">
                <input type="text" class="searchbox" placeholder="Enter city..." v-model="searchQuery" @keypress="searchCityForClimateData">
            </div>

            <div class="citydetails_container">
                <div class="cityname_container">
                    <h1 class="cityname">{{ currentClimate.name ?? 'Welcome!' }}</h1>
                </div>

                <div class="citydate_container">
                    <h3 class="citydate">{{ currentTime ?? 'Enter a city to get started!' }}</h3>
                </div>

                <div class="weathericon_container">
                    <img v-bind:src="weatherIconURL + weatherIcon" class="weathericon" draggable="false">
                </div>

                <div class="weathertype_container" v-if="currentClimate.weather !== undefined">
                    <h2 class="weathertype" v-if="currentClimate.weather[0] !== undefined">
                        {{ currentClimate.weather[0].main }}
                    </h2>
                </div>
            </div>

            <div class="climatedetails_container">
                <table class="climatedetails">
                    <tr>
                        <td>Wind: 10 MPH</td>
                        <td>Visibility: Low</td>
                        <td>Pressure: N/A</td>
                    </tr>
                </table>
            </div>
        </main>
    </div>
</template>

<style lang="scss" scoped>
    @import '../styles/Weather.scss';
</style>
