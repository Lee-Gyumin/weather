<template>
  <div class="rightContainer">
    <div id="cityNameBox">
      <div class="cityName">
        <p>{{ cityName }}</p>
        <p>{{ currentTime }}</p>
      </div>
    </div>
    <div id="contentsBox">
      <div class="buttonBox">
        <div class="buttonBackground">
          <button class="forecast">Forecast</button>
          <button class="airquality">Air Quality</button>
        </div>
      </div>
      <div class="weatherBox">
        <div class="airCondition">
          <p>{{ feeling }}</p>
        </div>
        <div class="detail">
          <div class="title">
            <p>πμμΈν λ μ¨ λ°μ΄ν°π</p>
          </div>
          <div class="data" v-for="(detailData, index) in subWeatherData" :key="index">
            <div class="dataName">
              <p></p>
              <p>{{ detailData.name }}</p>
            </div>
            <div class="dataValue">
              <p>{{ detailData.value }}<span></span></p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Map />
    <nav>
      <i class="fas fa-home"></i>
      <i class="fas fa-search-location"></i>
      <i class="fas fa-chart-line"></i>
      <i class="fas fa-cog"></i>
    </nav>
  </div>
</template>

<script>
import Map from "~/components/Map.vue";
import { ref } from "vue";
import axios from "axios";
import dayjs from "dayjs";
import "dayjs/locale/ko";
dayjs.locale("ko"); // globalλ‘ νκ΅­μ΄ locale μ¬μ©

export default {
  components: {
    Map,
  },
  setup() {
    // νλ©΄μμ λ³΄μ¬μ§ λ°μ΄ν°
    let currentTime = dayjs().format("YYYY. MM. DD. ddd"); // νμ¬ μκ°

    let cityName = ref(""); // λμ μ΄λ¦
    let feeling = ref(""); // νμ¬ μ¨λμ λν μ²΄κ°μ λνλ΄λ λ°μ΄ν°
    let subWeatherData = ref([]); // μμΈ λ μ¨ λ°μ΄ν°

    // νμμ€ν¬νλ‘ λ³ν
    const Unix_timestamp = (dt) => {
      let date = new Date(dt * 1000);
      // padStart() λ©μλλ νμ¬ λ¬Έμμ΄μ μμμ λ€λ₯Έ λ¬Έμμ΄λ‘ μ±μ, μ£Όμ΄μ§ κΈΈμ΄λ₯Ό λ§μ‘±νλ μλ‘μ΄ λ¬Έμμ΄μ λ°νν©λλ€.
      // μ±μλ£κΈ°λ λμ λ¬Έμμ΄μ μμ(μ’μΈ‘)λΆν° μ μ©λ©λλ€.
      let hour = date.getHours().toString().padStart(2, "0");
      return hour.substring(-2) + "μ";
    };

    // OpenWeatherAPI νΈμΆ ν¨μ
    const fetchOpenWeatherApi = async () => {
      // API νΈμΆμ μν νμ λ°μ΄ν°
      const API_KEY = "65c0b42cff7ecf92e9832ae7e8ee9669"; // env
      let initialLat = 36.5683;
      let initialLon = 126.9778;

      try {
        const res = await axios.get(`https://api.openweathermap.org/data/2.5/onecall?lat=${initialLat}&lon=${initialLon}&appid=${API_KEY}&units=metric`);

        let isInitialData = res.data.current; // μ΄κΈ°λ°μ΄ν°
        let isInitialCityName = res.data.timezone; // μ΄κΈ° λμμ΄λ¦ λ°μ΄ν°
        let isFeelLikeTemp = isInitialData.feels_like; // μ΄κΈ° μ²΄κ°μ¨λ λ°μ΄ν°
        let isTimeOfSunrise = isInitialData.sunrise; // μΌμΆμκ° λ°μ΄ν°
        let isTimeOfSunSet = isInitialData.sunset; // μΌλͺ°μκ° λ°μ΄ν°
        let isLineOfSight = isInitialData.visibility; // κ°μκ±°λ¦¬ λ°μ΄ν°

        // κΈ°μ€μ μ λ°λ₯Έ Arrayλ₯Ό νλ λ§λ€κ³ 
        // κΈ°μ€μ λ°λ₯Έ λ©μμ§μ λ°λ₯Έ Arrayλ₯Ό νλ λ§λ€μ΄μ
        // μ²΄κ°μ¨λ λ°μ΄ν°κ° νμμ ν΄μ
        // μνλ κ°μ λ½λ λ‘μ§μΌλ‘ κ΅¬μ±

        const pivots = [0, 10, 15, 20, 25, 30];
        const labels = ["λ§€μ° μΆμ", "μΆμ", "μμν¨", "μ μ ν¨", "λ³΄ν΅", "λμ", "λ§€μ° λμ"];

        let index = 0;
        for (const p of pivots) {
          if (isFeelLikeTemp <= p) break;
          index++;
        }
        feeling.value = labels[index];

        // κ°κ³΅ν λ°μ΄ν°λ₯Ό κ°μ§κ³  μλ‘μ΄ λ°°μ΄ μμ±
        let isProcessedData = [
          { name: "μΌμΆμκ°", value: Unix_timestamp(isTimeOfSunrise) },
          {
            name: "μΌλͺ°μκ°",
            value: Unix_timestamp(isTimeOfSunSet),
          },
          {
            name: "κ°μκ±°λ¦¬",
            value: isLineOfSight.toString().replace(/\B(?<!\.\d*)(?=(\d{3})+(?!\d))/g, ",") + "M",
          },
        ];

        // Composition APIμμ AJAXμμ²­κ³Ό λ°μ΄ν° λ³κ²½μ νλ €λ©΄ λ°μ΄ν°.valueλ‘ μ κ·Όν΄μΌνλ€.
        cityName.value = isInitialCityName.split("/")[1];
        subWeatherData.value = isProcessedData;
      } catch (error) {
        console.log(error);
      }
    };

    fetchOpenWeatherApi();

    return {
      cityName,
      currentTime,
      feeling,
      subWeatherData,
    };
  },
};
</script>

<style lang="scss" scoped>
@import "~/scss/main.scss";
@import "~/scss/subview.scss";
</style>
