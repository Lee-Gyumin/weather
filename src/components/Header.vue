<template>
  <header>
    <div class="headerWrap">
      <div class="info">
        <p>{{ currentTime }}</p>
        <h2>{{ cityName }}</h2>
        <div class="weatherDegree">
          <p>{{ currentTemp }}&deg;</p>
        </div>
      </div>
      <div class="menu"></div>
    </div>
  </header>
</template>

<script>
import dayjs from "dayjs";
import "dayjs/locale/ko";
export default {
  data() {
    return {
      currentTime: dayjs().format("YYYY.MM.DD.ddd"),
    };
  },
  computed: {
    cityName() {
      return this.$store.state.openWeatherApi.cityName;
    },
    // 현재 시간에 따른 현재 온도 데이터
    currentTemp() {
      const { currentTemp } = this.$store.state.openWeatherApi.currentWeather;
      return currentTemp;
    },
  },
  async created() {
    await this.$store.dispatch("openWeatherApi/FETCH_OPENWEATHER_API");
    const { currentTemp } = this.$store.state.openWeatherApi.currentWeather;
    this.currentTemp = currentTemp; // 현재시간에 대한 현재온도
  },
};
</script>

<style lang="scss" scoped>
@import "~/scss/main.scss";
@import "~/scss/mainview.scss";
</style>
