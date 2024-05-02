<template>
  <div>
    <div class="container" v-if="!isLoading">
      <nav>
        <div class="nav--left">
          <img src="@/assets/icons/Header_logo.svg" alt="" />
          <h1 class="primary-color">VUE WEATHER</h1>
        </div>
        <div class="nav--right">
          <img
            src="@/assets/icons/humidity.svg"
            alt=""
            @click="toggleDarkMode"
          />
          <input
            class="select--input"
            placeholder="Выбрать город"
            type="text"
            v-model="cityName"
            @keyup.enter="fetchWeatherData"
          />
        </div>
      </nav>
      <main v-if="weatherData">
        <div class="single--card">
          <div class="single--card-left">
            <div>
              <h1 class="primary-color">
                {{ formatTemperature(weatherData.list[0].temp.day) }}°
              </h1>
              <p>{{ getDayName(0) }}</p>
            </div>
            <div>
              <img
                :src="
                  getWeatherIcon(weatherData.list[0].weather[0].description)
                "
                style="width: 119px"
                alt=""
              />
            </div>
          </div>
          <div class="single--card-time gray--color">
            <p>Время: {{ getTimeFromOffset(weatherData.city.timezone) }}</p>
            <p>Город: {{ weatherData.city.name }}</p>
          </div>
        </div>
        <div class="info--card">
          <div class="info--card-right">
            <img src="@/assets/icons/temp.svg" alt="" />
            <p class="gray--color">Температура</p>
            <p>
              {{ formatTemperature(weatherData.list[0].temp.day) }}° - ощущается
              как {{ formatTemperature(weatherData.list[0].feels_like.day) }}°
            </p>
          </div>
          <div class="info--card-right">
            <img src="@/assets/icons/pressure.svg" alt="" />
            <p class="gray--color">Давление</p>
            <p>{{ weatherData.list[0].pressure }}</p>
          </div>
          <div class="info--card-right">
            <img src="@/assets/icons/precipitation.png" alt="" />
            <p class="gray--color">Осадки</p>
            <p>{{ weatherData.list[0].gust }}</p>
          </div>
          <div class="info--card-right">
            <img src="@/assets/icons/wind.svg" alt="" />
            <p class="gray--color">Ветер</p>
            <p>{{ weatherData.list[0].speed }} km</p>
          </div>
          <img
            src="@/assets/icons/Cloud image.png"
            alt="cloud-image"
            class="cloud--img"
          />
        </div>
      </main>
      <bottom>
        <div class="cards">
          <div class="cards--btns">
            <!-- <button class="week--btn">На неделю</button> -->
            <select v-model="selectedDuration" @change="changeDuration">
              <option value="7">На неделю</option>
              <option value="16">16 дней</option>
            </select>
            <button class="cancel--btn" @click="resetToDefault">
              Отменить
            </button>
          </div>
          <div class="card--group" v-if="weatherData">
            <div
              class="card"
              v-for="(day, index) in weatherData.list"
              :key="index"
            >
              <h4>{{ getDayName(index) }}</h4>
              <span class="gray--color card--date">{{ getDate(index) }}</span>
              <img :src="getWeatherIcon(day.weather[0].description)" alt="" />
              <span>{{ formatTemperature(day.temp.max) }}°</span>
              <span class="gray--color card--celsium"
                >{{ formatTemperature(day.temp.min) }}°</span
              >
              <p class="gray--color">{{ day.weather[0].description }}</p>
            </div>
          </div>
        </div>
      </bottom>
    </div>
    <div v-if="isLoading" class="loader">
      <img src="@/assets/icons/Header_logo.svg" alt="Loader" />
    </div>
  </div>
</template>

<script>
import axios from "axios";
import SunIcon from "@/assets/icons/sun.svg";
import RainIcon from "@/assets/icons/rain.svg";
import CloudsIcon from "/assets/icons/mainly_cloudy.svg";
import MainlyCloudyIcon from "@/assets/icons/mainly_cloudy.svg";
import SmallRainIcon from "@/assets/icons/small_rain.svg";

export default {
  data() {
    return {
      weatherData: "",
      isLoading: true,
      apiKey: "bd5e378503939ddaee76f12ad7a97608",
      cityName: "Tashkent",
      units: "metric",
      cnt: 7,
      selectedDuration: 7,
      isDarkMode: false,
      monthNames: [
        "Янв",
        "Фев",
        "Март",
        "Апрель",
        "Май",
        "Июнь",
        "Июль",
        "Авг",
        "Сен",
        "Окт",
        "Ноябрь",
        "Дек",
      ],
    };
  },
  created() {
    this.fetchWeatherData();
  },
  methods: {
    fetchWeatherData() {
      axios
        .get(
          `https://api.openweathermap.org/data/2.5/forecast/daily?q=${this.cityName}&units=${this.units}&cnt=${this.cnt}&appid=${this.apiKey}&lang=ru`
        )
        .then((response) => {
          this.weatherData = response.data;
          this.isLoading = false;
          console.log(this.weatherData);
        })
        .catch((error) => {
          this.isLoading = false;
          console.error("Error fetching weather data:", error);
        });
    },
    formatTemperature(temperature) {
      return Math.floor(temperature);
    },
    getDayName(index) {
      if (index === 0) return "Сегодня";
      else if (index === 1) return "Завтра";
      else {
        const days = [
          "Воскресенье",
          "Понедельник",
          "Вторник",
          "Среда",
          "Четверг",
          "Пятница",
          "Суббота",
        ];
        const today = new Date();
        const dayIndex = (today.getDay() + index) % 7;
        return days[dayIndex];
      }
    },
    getTimeFromOffset(offsetInSeconds) {
      let currentTimeMillis = Date.now();
      let offsetMillis = offsetInSeconds * 1000;
      let targetTimeMillis = currentTimeMillis + offsetMillis;
      let targetDate = new Date(targetTimeMillis);
      let hours = targetDate.getUTCHours();
      let minutes = targetDate.getUTCMinutes();
      let formattedHours = hours < 10 ? "0" + hours : hours;
      let formattedMinutes = minutes < 10 ? "0" + minutes : minutes;
      return formattedHours + ":" + formattedMinutes;
    },
    resetToDefault() {
      this.cnt = 7;
      this.selectedDuration = 7;
      this.fetchWeatherData();
    },
    changeDuration() {
      this.cnt = this.selectedDuration;
      this.fetchWeatherData();
    },
    getWeatherIcon(description) {
      const weatherIconMap = {
        "небольшой дождь": SmallRainIcon,
        дождь: RainIcon,
        пасмурно: MainlyCloudyIcon,
        "облачно с прояснениями": MainlyCloudyIcon,
        "небольшая облачность": CloudsIcon,
        ясно: SunIcon,
      };
      return weatherIconMap[description] || SunIcon;
    },
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode;
      if (this.isDarkMode) {
        document.body.classList.add("dark-mode");
      } else {
        document.body.classList.remove("dark-mode");
      }
    },
    getDate(index) {
      const currentDate = new Date();
      const nextDate = new Date(
        currentDate.getTime() + index * 24 * 60 * 60 * 1000
      );
      const day = nextDate.getDate();
      const monthIndex = nextDate.getMonth();
      return `${day} ${this.monthNames[monthIndex]}`;
    },
  },
};
</script>

<style lang="scss">
@import "/assets/scss/main.scss";
</style>
