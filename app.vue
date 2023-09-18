<template>
  <div class="row body">
    <div class="col-md-6 col-sm-12 container">
      <div class="search-box">
        <i class="fa fa-map-marker" aria-hidden="true"></i>
        <input
          type="text"
          @keyup.enter="searchData()"
          placeholder="Enter your location"
        />
        <button type="submit" @click="searchData()" onEnter>
          <i class="fa fa-search" aria-hidden="true"></i>
        </button>
      </div>

      <div class="not-found">
        <img src="/404.jpg" />
        <p>Invalid location !! Please enter again.</p>
      </div>

      <div class="loading">
        <div class="d-flex justify-content-center loading-detail">
          <div class="spinner-border" role="status">
            <span class="sr-only">Loading...</span>
          </div>
        </div>
      </div>

      <div class="weather-box">
        <img src="" />
        <p class="temperature"></p>
        <p class="description"></p>
      </div>

      <div class="weather-details">
        <div class="humidity">
          <i class="fa fa-thermometer-empty"></i>
          <div class="text">
            <span></span>
            <p>Humidity</p>
          </div>
        </div>
        <div class="wind">
          <i class="fa fa-pagelines" aria-hidden="true"></i>
          <div class="text">
            <span></span>
            <p>Wind Speed</p>
          </div>
        </div>
      </div>
    </div>

    <div class="col-md-6 col-sm-12 container week">
      <div class="search-box weekly">
        <i class="fa fa-calendar-o" aria-hidden="true"></i>
        <span>weather forecast</span>
      </div>

      <div class="loading-week">
        <div class="d-flex justify-content-center loading-detail">
          <div class="spinner-border" role="status">
            <span class="sr-only">Loading...</span>
          </div>
        </div>
      </div>

      <div class="detail">
        <div class="detail-list border-bottom" v-for="(data, index) in arr">
          <div class="list list-day">
            <span>{{ data.dt_txt }}</span>
          </div>
          <div class="list">
            <img :src="data.weather" />
          </div>
          <div class="list bar-min">
            <span class="min">{{ data.min }}</span>
          </div>
          <div class="list bar-line">
            <div class="line"></div>
          </div>
          <div class="list">
            <span>{{ data.max }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      defaultData: "",
      defaultWeekData: "",
      APIKey: "c1b556ca210bfa70be806d737a9d4818",
      arr: [],
    };
  },
  methods: {
    async searchData() {
      const container = document.querySelector(".container");
      const weatherBox = document.querySelector(".weather-box");
      const weatherDetails = document.querySelector(".weather-details");
      const error404 = document.querySelector(".not-found");
      const loading = document.querySelector(".loading");
      const week = document.querySelector(".week");
      const detail = document.querySelector(".detail");

      const city = document.querySelector(".search-box input").value;
      if (city === "") return;

      fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${this.APIKey}`
      )
        .then((response) => response.json())
        .then(async (json) => {
          if (json.cod === "404") {
            container.style.height = "590px";
            weatherBox.style.display = "none";
            weatherDetails.style.display = "none";
            week.style.display = "none";
            week.classList.remove("fadeInweek");
            loading.style.display = "none";
            loading.classList.remove("fadeInloading");
            error404.style.display = "block";
            error404.classList.add("fadeIn");
            return;
          }

          if (this.defaultData !== "") {
            weatherBox.style.display = "none";
            weatherDetails.style.display = "none";
            error404.style.display = "none";
            error404.classList.remove("fadeIn");
            loading.style.display = "block";
            loading.classList.add("fadeInloading");
            detail.style.display = "none";

            await new Promise((resolve, reject) => {
              setTimeout(() => {
                resolve();
              }, 1000);
            });
          }

          loading.style.display = "none";
          loading.classList.remove("fadeInloading");
          error404.style.display = "none";
          error404.classList.remove("fadeIn");

          setTimeout(() => {
            this.getWeek(city);
          }, 100);

          const image = document.querySelector(".weather-box img");
          const temperature = document.querySelector(
            ".weather-box .temperature"
          );
          const description = document.querySelector(
            ".weather-box .description"
          );
          const humidity = document.querySelector(
            ".weather-details .humidity span"
          );
          const wind = document.querySelector(".weather-details .wind span");

          let temp = parseInt(json.main.temp) - 273.15;
          image.src = this.getWeather(json.weather[0].main);
          temperature.innerHTML = `${parseInt(temp)} <span>°C</span>`;
          description.innerHTML = `${json.weather[0].description}`;
          humidity.innerHTML = `${json.main.humidity} %`;
          wind.innerHTML = `${parseInt(json.wind.speed)} Km/h`;

          week.style.display = "block";
          weatherBox.style.display = "";
          weatherDetails.style.display = "";
          weatherBox.classList.add("fadeIn");
          weatherDetails.classList.add("fadeIn");
          container.style.height = "590px";
          this.defaultData = city;
        });
    },

    getWeek(data) {
      fetch(
        `https://api.openweathermap.org/data/2.5/forecast?q=${data}&appid=${this.APIKey}`
      )
        .then((response) => response.json())
        .then(async (json) => {
          const week = document.querySelector(".week");
          const detail = document.querySelector(".detail");

          detail.style.display = "";
          if (this.defaultWeekData !== "") detail.classList.add("fadeIn");
          else detail.classList.add("fadeInFast");

          this.arr = [];
          let dayArr = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"];
          for (let index = 0; index < json.list.length; index++) {
            if (json.list[index].dt_txt.includes("00:00:00") || index === 0) {
              this.arr.push({
                dt_txt:
                  index === 0
                    ? `TODAY`
                    : dayArr[new Date(json.list[index].dt_txt).getDay()],
                weather: this.getWeather(json.list[index].weather[0].main),
                min: `${parseInt(
                  parseInt(json.list[index].main.temp_min) - 273.15
                )} °C`,
                max: `${parseInt(
                  parseInt(json.list[index].main.temp_max) - 273.15
                )} °C`,
              });
            }
          }
          week.classList.add("fadeInweek");
          this.defaultWeekData = this.defaultData;
        });
    },

    getWeather(data) {
      switch (data) {
        case "Clear":
          return "/clear.png";
        case "Rain":
          return "/rain.png";
        case "Snow":
          return "/snow.png";
        case "Clouds":
          return "/cloud.png";
        case "Mist":
          return "/mist.png";
        default:
          return "";
      }
    },
  },
};
</script>

<style scoped>
.body {
  margin: 0;
  padding: 0;
  border: 0;
  outline: none;
  box-sizing: border-box;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #06283d;
}

.week {
  height: 590px !important;
  background-color: #eaeff1 !important;
  display: none;
  scale: 0;
  opacity: 0;
  transition: 0.2s ease-out;
}

.fadeInweek {
  display: block;
  animation: 1.5s fadeIn forwards;
  animation-delay: 0.2s;
}

.weekly {
  border-radius: 5px;
  padding: 15px 30px;
  background-color: #fff;
  text-transform: uppercase;
  height: 80px !important;
  box-shadow: 1px 1px 10px #aaaaaa;
}

.weekly i {
  margin-left: -0.5rem !important;
}

.weekly span {
  margin-left: 34px;
}

.detail {
  height: 425px;
  margin-top: 24px;
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 1px 1px 10px #aaaaaa;
}

.detail hr {
  margin: auto;
  width: 90%;
}

.detail-list {
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding: 19px;
}

.detail-list .list-day,
.detail-list .bar-line {
  width: 20%;
}

.detail-list .bar-min {
  margin-left: 10%;
}

.detail-list .list {
  display: flex;
  align-items: center;
}

.detail-list .list .line {
  height: 4px;
  width: 100%;
  border-radius: 4px;
  background-color: #ee7a32;
}

.detail-list img {
  margin-top: -8px;
  margin-left: 20%;
  width: 40px;
  top: 7px;
}

.container {
  position: relative;
  width: 460px;
  height: 105px;
  background: #eaeff1;
  padding: 28px 32px;
  overflow: hidden;
  border-radius: 18px;
  font-family: "Roboto", sans-serif;
  transition: 0.6s ease-out;
}

.loading-detail {
  margin-top: 180px;
}

.spinner-border {
  width: 60px;
  height: 60px;
}

.search-box {
  width: 100%;
  height: min-content;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.search-box input {
  color: #06283d;
  width: 80%;
  font-size: 24px;
  font-weight: 500;
  text-transform: uppercase;
  border: none;
  padding: 10px 0px 8px 60px;
}

.search-box input:focus {
  outline: none;
}

.search-box input::placeholder {
  font-size: 20px;
  font-weight: 500;
  /* color: #06283d; */
  text-transform: capitalize;
}

.search-box button {
  cursor: pointer;
  width: 50px;
  height: 50px;
  color: #06283d;
  background: #dff6ff;
  border-radius: 50%;
  font-size: 22px;
  transition: 0.4s ease;
  border: none;
}

.search-box button:hover {
  background: #bae0ef;
}

.search-box i {
  position: absolute;
  color: #06283d;
  font-size: 28px;
  margin-left: 1.5rem;
}

.search-box button i {
  display: contents;
}

.search-box button i:hover {
  font-size: 32px;
  transition: 0.2s;
}

.weather-box {
  text-align: center;
}

.weather-box img {
  width: 170px;
  margin-top: 30px;
  height: 160px;
}

.weather-box .temperature {
  position: relative;
  color: #06283d;
  font-size: 4rem;
  font-weight: 800;
  margin-top: 30px;
  margin-left: -16px;
}

.weather-box .temperature span {
  position: absolute;
  margin-left: 4px;
  font-size: 1.5rem;
}

.weather-box .description {
  color: #06283d;
  font-size: 22px;
  font-weight: 500;
  text-transform: capitalize;
}

.weather-details {
  width: 100%;
  display: flex;
  justify-content: space-between;
  margin-top: 30px;
}

.weather-details .humidity,
.weather-details .wind {
  display: flex;
  align-items: center;
  width: 50%;
  height: 100px;
}

.weather-details .humidity {
  padding-left: 20px;
  justify-content: flex-start;
}

.weather-details .wind {
  padding-right: 20px;
  justify-content: flex-end;
}

.weather-details i {
  color: #06283d;
  font-size: 32px;
  margin-right: 16px;
  margin-top: -12px;
}

.weather-details span {
  color: #06283d;
  font-size: 22px;
  font-weight: 500;
}

.weather-details p {
  color: #06283d;
  font-size: 14px;
  font-weight: 500;
}

.not-found,
.loading,
.loading-week {
  width: 100%;
  text-align: center;
  margin-top: 1px;
  scale: 0;
  opacity: 0;
  display: none;
}

.not-found img {
  margin-top: 50px;
  border-radius: 5px;
  width: 90%;
}

.not-found p {
  color: #06283d;
  font-size: 22px;
  font-weight: 500;
  margin-top: 5px;
}

.weather-box,
.weather-details,
.detail {
  scale: 0;
  opacity: 0;
}

.fadeIn {
  animation: 0.5s fadeIn forwards;
  animation-delay: 0.5s;
}

@keyframes fadeIn {
  to {
    scale: 1;
    opacity: 1;
  }
}

.fadeInFast {
  animation: 0.2s fadeIn forwards;
  animation-delay: 0.2s;
}

@keyframes fadeInFast {
  to {
    scale: 1;
    opacity: 1;
  }
}

.fadeInloading {
  animation: 0.3s fadeIn forwards;
  animation-delay: 0.3s;
}

@keyframes fadeInloading {
  to {
    scale: 1;
    opacity: 1;
  }
}

@media (max-width: 920px) {
  .body {
    padding: 2rem;
    height: 220vh;
  }

  .week {
    margin-top: 2rem;
  }

  .detail-list,
  .search-box,
  .weather-details {
    font-size: 0.8rem;
  }
}

@media (max-width: 400px) {
  .wind i,
  .humidity i {
    display: none;
  }
}
</style>
