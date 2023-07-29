<script lang="ts">
    import axios from "axios";
    import Credits from './Credits.vue';

    export default {
        name: 'Weather',
        data () {
            return {
                currentClimate: {} as any,
                currentTime: '',
                currentFahrenheitTemperature: '',
                currentFahrenheitFeelsLike: '',
                currentWindInMPH: '',
                currentWindInCompassPoint: '',
                currentPressureInHg: '',
                weatherIcon: '10d@2x.png',
                weatherIconURL: 'https://openweathermap.org/img/wn/',
                searchQuery: '',
                showError: false,
                days: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
                compassPoints: ["N", "NNE", "NE", "ENE", "E", "ESE", "SE", "SSE", "S", "SSW", "SW", "WSW", "W", "WNW", "NW", "NNW", "N"],
                errorText: 'Unknown Error',
                apiKey: '0dfa2691b0e082a2f2aa34390b28f499',
                apiEndpoint: 'https://api.openweathermap.org/data/2.5/'
            }
        },
        components: {
            Credits
        },
        methods: {
            /**
                * A function that is triggered when the API request returns a successful callback and provides climate data of the inputted location.
                * @param newClimateData `any` - The data of the climate within the inputted location.
                * @returns `void` - This returns nothing.
            */
            refreshWithNewClimateData (newClimateData: any): void {
                this.currentClimate = newClimateData;
                this.weatherIcon = newClimateData.weather[0].icon + '@2x.png';

                this.currentTime = this.formatTimestamp(newClimateData.timezone);
                this.currentFahrenheitTemperature = this.getTemperatureInFahrenheit(newClimateData.main.temp);
                this.currentFahrenheitFeelsLike = this.getTemperatureInFahrenheit(newClimateData.main.feels_like);

                this.currentWindInMPH = this.getWindInMilesPerHour(newClimateData.wind.speed);
                this.currentWindInCompassPoint = this.getWindDegreeInCompassPoint(newClimateData.wind.deg);
                this.currentPressureInHg = this.getPressureInHg(newClimateData.main.pressure);
            },

            /**
                * A function that is triggered to hide the error alert box.
                * @returns `void` - This returns nothing.
            */
            hideErrorMessage (): void {
                this.showError = false;
            },

            /**
                * A function that is triggered when an error with the API request occurs to show an alert box with a customized message.
                * @param newErrorText `string` - The text to show in the error alert box.
                * @returns `void` - This returns nothing.
            */
            displayErrorMessage (newErrorText: string = 'Error 404'): void {
                this.errorText = newErrorText;
                this.showError = true;
            },

            /**
                * A function that is triggered when the user presses 'Enter' on the search input.
                * @param event `any` - The DOM event from the input trigger.
                * @returns `void` - This returns nothing.
            */
            async searchCityForClimateData (event: any): Promise<void> {
                if (event['key'] === 'Enter') {
                    const spinner: any = document.querySelector('.spinner_container');

                    // SHOW LOADER
                    spinner.style.display = 'flex';

                    // GET DATA FROM OPEN WEATHER API
                    await axios.get(`${this.apiEndpoint}weather?q=${this.searchQuery}&APPID=${this.apiKey}`).then(response => {
                        console.log(response);

                        // HIDE ERROR IF DISPLAYED AND REFRESH PAGE WITH NEW CLIMATE DATA
                        this.hideErrorMessage();
                        this.refreshWithNewClimateData(response['data']);
                    }).catch(error => {
                        console.log(error);

                        // SHOW ERROR ON PAGE
                        this.displayErrorMessage(error['message']);
                    });

                    // HIDE LOADER
                    spinner.style.display = 'none';
                }
            },

            /**
                * A function that is triggered when new climate data is received and the time is needed to be converted to a standard display.
                * @param locationTimezone `number` - The timezone of the location relaying the new climate data.
                * @returns `string` - This returns the standard display of the location's time. (Ex: Tuesday 7/29/2023, 7:01:40 AM)
            */
            formatTimestamp (locationTimezone: number): string {
                const currentDate: Date = new Date();
                const locationDate: Date = new Date((currentDate.getTime() + currentDate.getTimezoneOffset() * 60000) + (1000 * locationTimezone));

                return `${this.days[locationDate.getDay()]} ${locationDate.toLocaleString()}`;
            },

            /**
                * A function that is triggered when new climate data is received and a temperature unit is needed to be converted to fahrenheit. 
                * @param kelvinTemperature `number` - The kelvin temperature of the unit provided.
                * @returns `string` - This returns the fahrenheit temperature number of what was previously the kelvin temperature number.
            */
            getTemperatureInFahrenheit (kelvinTemperature: number): string {
                const fahrenheitTemperature: number = ((Math.round(kelvinTemperature) - 273.15) * 9 / 5) + 32;
                return fahrenheitTemperature.toFixed(0);
            },

            /**
                * A function that is triggered when new climate data is received and the wind is needed to be converted to miles per hour (MPH) rather than meters per second. 
                * @param metersPerSecond `number` - The meters per second provided.
                * @returns `string` - This returns the miles per hour of the meters per second provided.
            */
            getWindInMilesPerHour (metersPerSecond: number): string {
                return (metersPerSecond * 2.23694).toFixed(0);
            },

            /**
                * A function that is triggered when new climate data is received and the wind degree is needed to be converted to a compass point. 
                * @param windDegrees `number` - The original unit for the wind degree provided.
                * @returns `string` - This returns the compass point of the converted wind degree.
            */
            getWindDegreeInCompassPoint (windDegrees: number): string {
                return this.compassPoints[Number((windDegrees / 22.5).toFixed(0))];
            },

            /**
                * A function that is triggered when new climate data is received and the pressure in millibar is needed to be converted to mercury.
                * @param windDegrees `number` - The original unit for the pressure provided. Usually in millibar.
                * @returns `string` - This returns the mercury value of the pressure in millibar.
            */
            getPressureInHg (pressureInMillibar: number): string {
                return (pressureInMillibar / 33.864).toFixed(2);
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
                    <h3 class="citydate">{{ currentTime === '' ? 'Enter a city to get started!' : currentTime }}</h3>
                </div>

                <div class="weathericon_container" v-if="currentClimate.weather !== undefined">
                    <img v-bind:src="weatherIconURL + weatherIcon" alt="current weather icon" class="weathericon" draggable="false">
                </div>

                <div class="weathertype_container" v-if="currentClimate.weather !== undefined">
                    <h2 class="weathertype" v-if="currentClimate.weather[0] !== undefined">
                        {{ currentClimate.weather[0].main }}
                    </h2>
                </div>
            </div>

            <div class="climatedetails_container" v-if="currentClimate.weather !== undefined">
                <table class="climatedetails">
                    <tr>
                        <td>
                            Temperature:
                            <br>
                            <span class="detail">{{ currentFahrenheitTemperature }}F</span>
                        </td>

                        <td>
                            Wind:
                            <br>
                            <span class="detail">{{ currentWindInMPH }} MPH ({{ currentWindInCompassPoint }})</span>
                        </td>

                        <td>
                            Pressure:
                            <br>
                            <span class="detail">{{ currentPressureInHg }} inHg</span>
                        </td>
                    </tr>
                </table>
            </div>

            <div class="climatedetails_container" style="margin-top: 13.5px;" v-if="currentClimate.weather !== undefined">
                <table class="climatedetails">
                    <tr>
                        <td>
                            Humidity:
                            <br>
                            <span class="detail">{{ currentClimate.main.humidity }}%</span>
                        </td>

                        <td>
                            Feels Like:
                            <br>
                            <span class="detail">{{ currentFahrenheitFeelsLike }}F</span>
                        </td>
                    </tr>
                </table>
            </div>

            <div class="climatedetails_container" style="margin-top: 13.5px;" v-if="currentClimate.weather !== undefined">
                <table class="climatedetails">
                    <tr>
                        <td>
                            Sunrise:
                            <br>
                            <span class="detail">
                                {{ new Date(currentClimate.sys.sunrise * 1000).toLocaleTimeString().replace(/([\d]+:[\d]{2})(:[\d]{2})(.*)/, "$1$3") }}
                            </span>
                        </td>

                        <td>
                            Sunset:
                            <br>
                            <span class="detail">
                                {{ new Date(currentClimate.sys.sunset * 1000).toLocaleTimeString().replace(/([\d]+:[\d]{2})(:[\d]{2})(.*)/, "$1$3") }}
                            </span>
                        </td>
                    </tr>
                </table>
            </div>

            <Credits />
        </main>
    </div>
</template>

<style lang="scss" scoped>
    @import '../styles/Weather.scss';
</style>
