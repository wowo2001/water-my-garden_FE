<template>
    <div>
        <!-- Show a loading spinner until weather data is loaded -->
        <div v-if="loading">
            <p>Loading...</p>
        </div>

        <!-- Show the calendar once weather data is loaded -->
        <el-calendar v-if="!loading">
            <template #date-cell="{ data }">
                <div @click="handleDateClick(data)">
                    <p :class="data.isSelected ? 'is-selected' : ''">
                        {{ data.day.split('-').slice(2).join('-') }}
                    </p>
                    <span v-if="getWeatherForDate(data.day)">
                        <el-icon>
                            <!-- Show 'Pouring' icon for rainy days -->
                            <Pouring v-if="getWeatherForDate(data.day) === 'Rainy'" />
                            <!-- Show 'Sunny' icon for sunny days -->
                            <Sunny v-if="getWeatherForDate(data.day) === 'Sunny'" />
                            <Check v-if="getWeatherForDate(data.day) === 'Check'" />
                        </el-icon>
                    </span>
                </div>
            </template>
        </el-calendar>

        <!-- Dialog for confirming watering action -->
        <el-dialog v-model="dialogVisible"
                   title="Do you water your garden this day?"
                   width="400px">
            <template v-slot:footer>
                <el-button :style="{ backgroundColor:'green', color: 'white' }"
                           @click="handleYesClick">Yes</el-button>

                <el-button :style="{ backgroundColor:'red', color: 'white' }"
                           @click="handleNoClick">No</el-button>
            </template>
        </el-dialog>
    </div>
</template>

<script>
    import axios from 'axios';
    const apiHost = 'http://13.210.14.154:3000'
    export default {
        name: 'App',
        data() {
            return {
                dialogVisible: false,
                weather: [],
                loading: true, // New flag to track loading state
                selectedDate: null, // Add selectedDate to store the clicked date
            };
        },
        async mounted() {
            await this.getWeather();  // Ensure weather data is fetched before rendering
        },
        methods: {
            async getWeather() {
                try {
                    const response = await axios.get(`${apiHost}/Weather/GetWeather`);
                    const weather = response.data.weather;
                    weather.forEach((day) => {
                        // Ensure each day has a 'wateringGarden' field with a default value of false
                        this.weather.push({
                            date: day.date,
                            weather: day.weather,
                            wateringGarden: day.wateringGarden || false // Default to false if undefined
                        });
                    });
                } catch (error) {
                    console.error("Error fetching weather data:", error);
                } finally {
                    this.loading = false; // Set loading to false when data is fetched
                }
            },

            handleDateClick(data) {
                console.log('Date clicked:', data.day);
                this.selectedDate = data.day; // Store the selected date
                this.dialogVisible = true; // Open the dialog
            },

            getWeatherForDate(date) {
                // Ensure data is loaded before calling this method
                if (this.loading) return null; // Avoid calling getWeatherForDate before data is loaded

                const weatherForDate = this.weather.find((day) => day.date === date);

                if (weatherForDate) {
                    // If wateringGarden is true, return "Check", otherwise return weather status
                    return weatherForDate.wateringGarden ? 'Check' : weatherForDate.weather;
                }

                return null; // Return null if no weather data found for the date
            },

            async handleYesClick() {
                const requestBody = {
                    date: this.selectedDate,
                    operate: "add"
                };
                try {
                    const response = await axios.post(`${apiHost}/Weather/RecordWatering`, requestBody)
                    if (response.status === 200) {
                        const selectedWeather = this.weather.find((day) => day.date === this.selectedDate);
                        if (selectedWeather) {
                            // Set wateringGarden to true for the selected date
                            selectedWeather.wateringGarden = true;

                            console.log(`Garden watered on ${this.selectedDate}`);
                        }
                        this.selectedDate = null;
                        this.dialogVisible = false;
                    }
                }
                catch (error) {
                    // Handle error (e.g., show an error message to the user)
                    console.log('Error updating the menu:', error);
                }

            },

            async handleNoClick() {
                const requestBody = {
                    date: this.selectedDate,
                    operate: "remove"
                };
                try {
                    const response = await axios.post(`${apiHost}/Weather/RecordWatering`, requestBody)
                    if (response.status === 200) {
                        const selectedWeather = this.weather.find((day) => day.date === this.selectedDate);
                        if (selectedWeather) {
                            // Set wateringGarden to true for the selected date
                            selectedWeather.wateringGarden = false;

                            console.log(`Garden watered on ${this.selectedDate}`);
                        }
                        this.selectedDate = null;
                        this.dialogVisible = false;
     
                    }
                }
                catch (error) {
                    // Handle error (e.g., show an error message to the user)
                    console.log('Error updating the menu:', error);
                }
            },
        },
    };
</script>
