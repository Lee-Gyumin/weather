<template>
  <div class="leftContainer afternoon">
    <div id="contentsBox">
      <div class="weatherBox">
        <div class="weatherIcon">
          <div class="city-info">
            <div class="cityName">
              <h2>{{ cityName }}</h2>
              <p>{{ currentTime }}</p>
            </div>
            <div class="weatherDegree">
              <p>{{ currentTemp }}&deg;</p>
            </div>
          </div>
          <img :src="images[0]" alt="MainLogo" />
        </div>
        <div class="weatherData">
          <div
            v-for="temporary in temporaryData"
            :key="temporary.title"
            class="detailData"
          >
            <p>{{ temporary.title }}</p>
            <p v-html="temporary.value"></p>
          </div>
        </div>
      </div>
    </div>
    <div class="search">
      <input type="text" placeholder="도시를 입력해주세요." />
    </div>
    <div class="weatherWrap">
      <div class="textBox">
        <p>시간대별 날씨 정보</p>
        <!-- <p>이번주 날씨 보기</p> -->
      </div>
      <div class="timelyWeatherBox">
        <div
          class="timelyWeather"
          v-for="(temp, index) in arrayTemps"
          :key="index"
        >
          <div class="icon">
            <p class="time">
              {{ Unix_timestamp(temp.dt) }}
            </p>
            <img :src="images[index]" alt="" />
          </div>
          <div class="data">
            <!-- <p class="time">
              {{ Unix_timestamp(temp.dt) }}
            </p> -->
            <p class="currentDegree">{{ Math.round(temp.temp) }}&deg;</p>
            <div>
              <img src="~/assets/images/drop.png" alt="" />
              <p class="fall">{{ Math.round(temp.pop * 100) }}%</p>
            </div>
            <div>
              <p class="rain">
                <!-- {{ temp.rain ? Object.values(temp.rain)[0] : "" }} -->
                {{
                  temp.rain ? Math.round(Object.values(temp.rain)[0]) : 0
                }}mm/h
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="weatherWrap">
      <div class="textBox">
        <p>주간 날씨 정보</p>
        <!-- <p>이번주 날씨 보기</p> -->
      </div>
      <div class="weeklyWeatherBox">
        <div
          class="weeklyWeather"
          v-for="(temp, index) in weeklyTemps"
          :key="index"
        >
          <div class="weeklyInfo" :class="{ today: index === 0 }">
            <div class="dateInfo">
              <div class="time">
                <p v-show="index === 0">오늘</p>
                {{ Unix_timestamp2(temp.dt) }}
              </div>
              <div class="temp">
                {{ Math.round(temp.temp.morn) }}&deg;/
                <strong>{{ Math.round(temp.temp.max) }}&deg;</strong>
              </div>
            </div>
            <div class="allDay">
              <div class="morn">
                <span>오전</span>
                <!-- <span>오전 {{ Math.round(temp.pop * 100) }}%</span> -->
                <img
                  :src="`src/assets/images/${temp.weather[0].icon}.png`"
                  alt=""
                />
              </div>
              <div class="night">
                <span>오후</span>
                <img :src="images[index]" alt="" />
              </div>
            </div>
            <!-- <img :src="images[index]" alt="" /> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from "dayjs";
import "dayjs/locale/ko";
dayjs.locale("ko"); // global로 한국어 locale 사용

export default {
  data() {
    return {
      // 현재 시간을 나타내기 위한 Dayjs 플러그인 사용
      currentTime: dayjs().format("YYYY. MM. DD. ddd"),
      getHeight: null,
    };
  },
  async created() {
    // 초기데이터 선언을 위한 코드 작성
    // https://api.openweathermap.org/data/2.5/onecall?lat={lat}&lon={lon}&exclude={part}&appid={API key}
    // Vuex Store의 Mutations를 실행할 때는 commit() 메서드를
    // Vuex Store의 Actions를 실행할 때는 dispatch() 메서드를 사용한다.
    await this.$store.dispatch("openWeatherApi/FETCH_OPENWEATHER_API");
    const { currentTemp, currentHumidity, currentWindSpeed, currentFeelsLike } =
      this.$store.state.openWeatherApi.currentWeather;
    this.currentTemp = currentTemp; // 현재시간에 대한 현재온도
    this.temporaryData[0].value = currentHumidity + "%"; // 현재시간에 대한 습도
    this.temporaryData[1].value = currentWindSpeed + "m/s"; // 현재시간에 대한 풍속
    this.temporaryData[2].value = Math.round(currentFeelsLike) + "&deg"; // 현재시간에 대한 체감온도
    this.arrayTemps = this.$store.state.openWeatherApi.hourlyWeather;
    this.weeklyTemps = this.$store.state.openWeatherApi.daily;
    this.images = this.$store.state.openWeatherApi.imagePath;
    console.log("images", this.images);
    console.log("weekly", this.weeklyTemps);
  },
  mounted() {
    let ele = document.getElementById("contentsBox");
    console.log("height", ele.scrollHeight);
  },
  watch: {},
  computed: {
    // 마커를 선택했을 때, 레이아웃에 보여지는 도시 이름
    cityName() {
      return this.$store.state.openWeatherApi.cityName;
    },
    // 현재 시간에 따른 현재 온도 데이터
    currentTemp() {
      const { currentTemp } = this.$store.state.openWeatherApi.currentWeather;
      return currentTemp;
    },
    arrayTemps() {
      return this.$store.state.openWeatherApi.hourlyWeather;
    },
    weeklyTemps() {
      return this.$store.state.openWeatherApi.daily;
    },
    // 상세 날씨 데이터를 받아주는 데이터 할당
    temporaryData() {
      const { currentHumidity, currentWindSpeed, currentFeelsLike } =
        this.$store.state.openWeatherApi.currentWeather;
      return [
        {
          title: "습도",
          value: currentHumidity + "%",
        },
        {
          title: "풍속",
          value: currentWindSpeed + "m/s",
        },
        {
          title: "체감온도",
          value: Math.round(currentFeelsLike) + "&deg",
        },
      ];
    },
    // 시간별 날씨 데이터에 대한 아이콘 이미지
    images() {
      return this.$store.state.openWeatherApi.images;
    },
  },
  methods: {
    // 타임스탬프로 변환
    Unix_timestamp(dt) {
      let date = new Date(dt * 1000);
      let day = date.getDate().toString().padStart(2, "0");
      let hour = date.getHours().toString().padStart(2, "0");
      return day + "일 " + hour + "시";
    },
    Unix_timestamp2(dt) {
      const WEEKDAY = ["일", "월", "화", "수", "목", "금", "토"];
      let date = new Date(dt * 1000);
      let day = date.getDate().toString().padStart(1, "0");
      let month = date.getMonth() + 1;
      return month + "." + day + "(" + WEEKDAY[date.getDay()] + ")";
      // return dayjs(date).format("YYYY. MM. DD. ddd");
    },
  },
};
</script>

<style lang="scss" scoped>
@import "~/scss/main.scss";
@import "~/scss/mainview.scss";
</style>
