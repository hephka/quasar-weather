<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        borderless
        dark
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="
            `http://openweathermap.org/img/wn/${
              weatherData.weather[0].icon
            }@2x.png`
          "
          alt=""
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "46485d7ab521ae5ead07345f0d638ed2"
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        //determine day or night depending on icon suffix
        if (this.weatherData.weather[0].icon.endsWith("d")) return "bg-day";
        else return "bg-night";
      }
    }
  },
  methods: {
    getLocation() {
      //show loading icon
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        //with electron platform
        this.$axios.get("https://freegeoip.app/json/").then(res => {
          this.lat = res.data.latitude;
          this.lon = res.data.longitude;
          this.getWeatherByCoords();
        });
      } else {
        //get lat and lng
        navigator.geolocation.getCurrentPosition(position => {
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;

          //run next method to get weather data
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();

      //call API for weather data and insert data into weatherData
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${
          this.apiKey
        }&units=metric`
      ).then(res => {
        this.weatherData = res.data;
        this.$q.loading.hide();
      });
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
      )
        .then(res => {
          this.weatherData = res.data;
          this.$q.loading.hide();
        })
        .catch(err => {
          //catch errors
          this.$q.dialog({
            title: "Error",
            message: "Sorry, that location could not be found."
          });
          this.$q.loading.hide();
        });
    }
  }
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-day
    background: linear-gradient(to bottom, #2193b0, #6dd5ed)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../../public/skyline.png)
  background-size: contain
  background-position: center bottom
</style>
