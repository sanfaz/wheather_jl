<template>
  <div id="app" class="app">
    <header class="header">
      <h1>🌤 Weather Finder</h1>
      <p>Search for any city to see its current weather and forecast</p>
    </header>

    <main class="content">
      <div class="search-container">
        <input
          v-model="locationQuery"
          type="text"
          placeholder="Enter city name"
          class="search-input"
        />
        <button @click="searchLocation" class="search-button">Search</button>
      </div>

      <div v-if="locations.length" class="locations">
        <h2>Results:</h2>
        <ul>
          <li
            v-for="location in locations"
            :key="location.id"
            class="location-item"
          >
            <button @click="selectLocation(location)" class="location-button">
              {{ location.name }}, {{ location.country }}
            </button>
          </li>
        </ul>
      </div>

      <div v-if="weatherData" class="weather">
        <h2>Weather for {{ selectedLocation.name }}</h2>
        <ul class="weather-list">
          <li v-for="(day, index) in weatherData.temperature_2m_max" :key="index">
            <strong>{{ getLocalDate(index) }}</strong>: 
            {{ day }}°C (Max) / {{ weatherData.temperature_2m_min[index] }}°C (Min)
          </li>
        </ul>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      locationQuery: '',
      locations: [],
      selectedLocation: null,
      weatherData: null,
    };
  },
  methods: {
    async searchLocation() {
      if (!this.locationQuery) return;
      const response = await fetch(
        `https://geocoding-api.open-meteo.com/v1/search?name=${this.locationQuery}&count=10&language=en&format=json`
      );
      const data = await response.json();
      this.locations = data.results || [];
    },
    async selectLocation(location) {
      this.selectedLocation = location;
      const today = new Date().toISOString().split('T')[0];
      const tomorrow = new Date(Date.now() + 86400000).toISOString().split('T')[0];
      const response = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${location.latitude}&longitude=${location.longitude}&daily=temperature_2m_max,temperature_2m_min&timezone=auto&start_date=${today}&end_date=${tomorrow}`
      );
      const data = await response.json();
      this.weatherData = data.daily;
    },
    getLocalDate(index) {
      const date = new Date();
      date.setDate(date.getDate() + index);
      return date.toLocaleDateString();
    },
  },
};
</script>

<style>

body {
  margin: 0;
  font-family: 'Arial', sans-serif;
  background: linear-gradient(to bottom, #4facfe, #00f2fe);
  color: #333;
}

.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 20px;
  color: white;
}

.header h1 {
  font-size: 2.5em;
  margin: 0;
}

.header p {
  margin: 5px 0 15px 0;
}


.search-container {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.search-input {
  padding: 10px;
  font-size: 1em;
  border-radius: 5px;
  border: 1px solid #ccc;
  width: 60%;
}

.search-button {
  padding: 10px 20px;
  font-size: 1em;
  border: none;
  border-radius: 5px;
  background-color: #4facfe;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

.search-button:hover {
  background-color: #00c6ff;
}

.locations {
  margin-top: 20px;
}

.locations ul {
  list-style: none;
  padding: 0;
}

.location-item {
  margin-bottom: 10px;
}

.location-button {
  padding: 10px;
  font-size: 1em;
  border: none;
  border-radius: 5px;
  background-color: #00f2fe;
  color: white;
  cursor: pointer;
  width: 100%;
  text-align: left;
}

.location-button:hover {
  background-color: #4facfe;
}

.weather {
  margin-top: 20px;
  padding: 20px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.weather h2 {
  margin-top: 0;
  font-size: 1.5em;
}

.weather-list {
  list-style: none;
  padding: 0;
}

.weather-list li {
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.weather-list li:last-child {
  border-bottom: none;
}
</style>
