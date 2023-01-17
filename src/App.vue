<script>
import axios from "axios";
import Chart from "chart.js/auto";

export default {
  name: "App",
  data() {
    return {
      weathers: [],
      idx: 0,
      weatherData: {},
      time: this.currentTime(),
      searchCity: "",
      favoriteCities: JSON.parse(localStorage.getItem("favorites")) || [],
      haveFavorite: false,
      usualCity: true,
      svgFill: false,

      cityIsExist: false,
      cityIsUnknown: false,
      manyCities: false,

      activeColor: "red",
      filter: "",
      showModal: false,
      selectedCity: "",
      cityToDelete: "",

      isNotEmpty: false,
      loading: false,
    };
  },
  methods: {
    async clickWeatherButton() {
      await this.fetchCity();
      if (!this.checkingCorrectness()) return;
      this.pushNewCity();
    },

    async fetchCity() {
      const apiKey = "e81c2a0f775eeea447f275a781828331";
      const city = this.searchCity;

      try {
        await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`
        )
          .then((response) => {
            return response.json();
          })
          .then((data) => {
            this.weatherData = data;
          });

      } catch (error) {
        this.weatherData = {};
      }
    },

    selectCity(city) {
      this.searchCity = city.name;
      this.showList = false;
    },

    checkingCorrectness() {
      if ((this.searchCity = "")) {
        this.emptyInput();
        return false;
      }
      if (this.weatherData.cod == "404") {
        this.unknownCity();
        this.searchCity = "";
        return false;
      }
      if (
        this.weathers.find((city) => city.city == this.weatherData.name) !==
        undefined
      ) {
        this.existedCity();
        return false;
      }
      if (this.weathers.length >= 5) {
        this.toManyCities();
        return false;
      }

      return true;
    },

    pushNewCity() {
      const newWeather = {
        index: this.idx + 1,
        date: this.currentDate(),
        city: this.searchCity,
        degree: NaN,
        favorite: "none",
      };

      newWeather.degree = `${Math.round(this.weatherData.main.temp)} °C`;

      newWeather.city = this.weatherData.name;

      this.weathers.push(newWeather);
      this.filter = "";
      this.searchCity = "";

      // localStorage.setItem(
      //   "weatherInChoosenCity",
      //   JSON.stringify(this.weathers)
      // );
    },

    deleteBlock() {
      this.showModal = false;
    
    },

    unknownCity() {
      this.cityIsUnknown = true;
      setTimeout(() => {
        this.cityIsUnknown = false;
      }, 3000);
      return;
    },

    emptyInput() {
      this.isNotEmpty = true;
      setTimeout(() => {
        this.isNotEmpty = false;
      }, 1000);
      return;
    },

    existedCity() {
      this.cityIsExist = true;
      setTimeout(() => {
        this.cityIsExist = false;
      }, 1000);
      this.searchCity = "";
      return;
    },

    toManyCities() {
      this.manyCities = true;
      setTimeout(() => {
        this.manyCities = false;
      }, 2000);
      this.searchCity = "";
      return;
    },

    deleteCity() {

      if (this.usualCity) {
        this.weathers = this.weathers.filter(
          (weather) => weather.city !== this.cityToDelete
        );
        this.favoriteCities = this.favoriteCities.filter(
          (weather) => weather.city !== this.cityToDelete
        );
      }
      if (!this.usualCity) {
        this.favoriteCities = this.favoriteCities.filter(
          (weather) => weather.city !== this.cityToDelete
          
        );
        
        localStorage.setItem("favorites", JSON.stringify(this.favoriteCities));
      }

      // localStorage.setItem("weatherInChoosenCity", JSON.stringify(this.weathers))
      this.showModal = false;
      this.cityToDelete = "";
    },

    normalizeCityName(city) {
      const str = city.replace(/( |^)[а-яёa-z]/g, function (x) {
        return x.toUpperCase();
      });
      return str;
    },

    currentDate() {
      const current = new Date();
      const date = current
        .toLocaleDateString("en-GB", { day: "numeric", month: "long" })
        .replace(/ /g, " ");
      return date;
    },

    isInFavorites(city) {
      if (this.favoriteCities.some((f) => f.city == city.city)) {
        return true;
      }
      return false;
    },

    addToFavorite(city) {
      if (this.favoriteCities.length >= 5) {
        return this.toManyCities();
      }

      city.favorite = "black";
      this.favoriteCities.push(city);
      localStorage.setItem("favorites", JSON.stringify(this.favoriteCities));


    },
    removeFromFavorites(city) {
      city.favorite = "none";
      this.favoriteCities = this.favoriteCities.filter(
        (weather) => weather != city
      );
      localStorage.setItem("favorites", JSON.stringify(this.favoriteCities));
    },

    currentTime() {
      const today = new Date();
      const hour = today.getHours();
      return hour;
    },

    showFavorites() {
      if (this.favoriteCities.length > 0) {
        this.svgFill = !this.svgFill;
        this.usualCity = !this.usualCity;
      } else {
        this.notHaveFavorite();
      }
    },

    notHaveFavorite() {
      this.haveFavorite = true;
      setTimeout(() => {
        this.haveFavorite = false;
      }, 2000);
      return;
    },

    filteredCities() {
      return this.weathers.filter((searchCity) =>
        searchCity.city.includes(this.filter)
      );
    },

    colorAttr(city) {
      this.svgFill = "black";
    },
  },

  mounted() {
    this.loading = true;
    axios
      .get("http://ip-api.com/json")
      .then((response) => {
        const city = response.data.city;
        const apiKey = "e81c2a0f775eeea447f275a781828331";
        return axios.get(
          `http://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`
        );
      })
      .then((response) => {
        this.weatherData = response.data;
        this.pushNewCity();
        this.loading = false;
      })
      .catch((error) => {
        console.log(error);
        this.loading = false;
      })

  },

  watch: {
    favoriteCities: {
      immediate: true,
      deep: true,
      handler(newVal) {
        if (newVal.length == 0) {
          this.usualCity = true;
          this.svgFill = false;
        }
      },
    },
  },

  created() {},
};
</script>

<template>
  <div class="wrapper">
    <div class="big-image">
      <img src="./images/world.svg" alt="" class="bgImage" />
    </div>
    <div class="header">
      <div class="header-spreadsheet-wrapper">
        <div class="header-title">
          <p>World</p>
          <p>weather</p>
          <p>forecast</p>
        </div>
        <div class="button-wrapper" v-if="usualCity">
          <input
            v-model="searchCity"
            type="text"
            placeholder=""
            @keyup.enter="clickWeatherButton()"
            class="form-search"
            id="select_city"
            list="inputCities"
            @input="fetchCity"
            ref="cityInput"
            autocomplete="on"
            spellcheck="false"
            aria-label="Select city"
            data-bs-toggle="dropdown"
            aria-expanded="false"
          />
          <button
            v-on:click="clickWeatherButton()"
            data-glow
            class="button"
            type="button"
          >
            <span class="button-text">Add city</span>
          </button>
        </div>
        <Transition>
          <p class="alert" v-if="cityIsExist" :style="{ color: activeColor }">
            This city is already exist
          </p>
        </Transition>
        <Transition>
          <p class="alert" v-if="cityIsUnknown" :style="{ color: activeColor }">
            Unknown city
          </p>
        </Transition>
        <Transition>
          <p class="alert" v-if="manyCities" :style="{ color: activeColor }">
            You can only add 5 cities
          </p>
        </Transition>
        <Transition>
          <p class="alert" v-if="isNotEmpty" :style="{ color: activeColor }">
            Add some city
          </p>
        </Transition>
        <div class="filter-favorite">
          <div class="filter">
            Filter: <input type="text" v-model="filter" />
          </div>
          <div class="favorite" @click="showFavorites()">
            <p class="favorite-text">Favorite</p>
            <div class="favorite-icon">
              <svg
                aria-hidden="true"
                v-bind:fill="[this.svgFill ? 'black' : 'none']"
                stroke="currentColor"
                src="/src/images/Heart.svg"
              >
                <path
                  d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                ></path>
              </svg>
            </div>
          </div>
          <Transition>
            <p
              class="alert"
              v-if="this.haveFavorite"
              :style="{ color: activeColor }"
              style="margin-left: 1vw"
            >
              You don't have favorite cities
            </p>
          </Transition>
        </div>
        <div v-if="this.loading" class="loader">
          <div class="loading">Loaging...</div>
        </div>
        <div class="header-spreadsheet head" v-if="this.loading">
          <div class="head-date">Date:</div>
          <div class="head-city">Your city:</div>
          <div class="head-degree">Degrees:</div>
          <div class="head-icon">Weather:</div>
        </div>
        <div v-if="usualCity">
          <div
            class="spreadsheet-wrapper"
            v-for="weather in filteredCities()"
            v-bind:key="weather"
          >
            <div class="header-spreadsheet spreadsheet">
              <div class="date">{{ weather.date }}</div>
              <div class="city">{{ weather.city }}</div>
              <div class="degree">{{ weather.degree }}</div>
              <div class="icon-favorite">
                <button
                  class="button-favorite"
                  @click="
                    [
                      isInFavorites(weather)
                        ? removeFromFavorites(weather)
                        : addToFavorite(weather),
                    ]
                  "
                >
                  <svg
                    aria-hidden="true"
                    v-bind:fill="weather.favorite"
                    stroke="currentColor"
                    stroke-width="1.5"
                    viewBox="0 0 24 24"
                    src="/src/images/Heart.svg"
                  >
                    <path
                      d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    ></path>
                  </svg>
                </button>
              </div>
              <div
                class="icon-delete"
                v-if="!showModal"
                @click="this.cityToDelete = weather.city"
              >
                <button
                  @click="showModal = true"
                  class="button-delete"
                  type="button"
                >
                  <div><img src="./images/delete.svg" alt="" /></div>
                </button>
              </div>
              <div v-if="showModal">
                <div class="modal-overlay"></div>
                <div class="modal-content">
                  <p>Are you sure you want to remove this city?</p>
                  <div class="modal-buttons">
                    <button @click="showModal = false">Cancel</button>
                    <button @click="deleteCity">Delete</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-if="!usualCity">
          <div
            class="spreadsheet-wrapper"
            v-for="favorite in filteredCities()"
            v-bind:key="favorite"
          >
            <div class="header-spreadsheet spreadsheet">
              <div class="date">{{ favorite.date }}</div>
              <div class="city">{{ favorite.city }}</div>
              <div class="degree">{{ favorite.degree }}</div>
              <div class="icon-favorite">
                <button
                  class="button-favorite"
                  @click="
                    [
                      this.isInFavorites(weather)
                        ? this.removeFromFavorites(weather)
                        : this.addToFavorite(weather),
                    ]
                  "
                >
                  <svg
                    aria-hidden="true"
                    fill="black"
                    stroke="currentColor"
                    stroke-width="1.5"
                    viewBox="0 0 24 24"
                    src="/src/images/Heart.svg"
                  >
                    <path
                      d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    ></path>
                  </svg>
                </button>
              </div>
              <div
                class="icon-delete"
                v-if="!showModal"
                @click="this.cityToDelete = favorite.city"
              >
                <button
                  @click="showModal = true"
                  class="button-delete"
                  type="button"
                >
                  <div><img src="./images/delete.svg" alt="" /></div>
                </button>
              </div>
              <div v-if="showModal">
                <div class="modal-overlay"></div>
                <div class="modal-content">
                  <p>
                    Are you sure you want to remove this city from your
                    favorites?
                  </p>
                  <div class="modal-buttons">
                    <button @click="showModal = false">Cancel</button>
                    <button @click.stop="deleteCity(favorite)">Delete</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped >
.wrapper {
  width: 100vw;
  height: auto;
  margin: 0 auto;
  align-items: center;
  color: black;
}

.big-image {
  width: 100vw;
  position: sticky;

  height: auto;
}

.bgImage {
  width: 100vw;
  height: 25vw;
}

.header {
  display: flex;
  justify-content: left;
  align-items: flex-start;
}

.alert {
  display: flex;
  justify-content: start;
  margin-left: 20vw;
  position: absolute;
  font-family: "Montserrat", sans-serif;
}

.filter {
  font-family: "Montserrat", sans-serif;
  font-family: "Manrope", sans-serif;
  font-size: 1.5vw;
  margin: 1vw 0 0 1vw;

  display: flex;
  justify-content: start;
  align-items: center;

  cursor: default;
}

.filter > input {
  border-radius: 40px;
  border: 1px solid black;
  width: 10vw;
  margin: 0 0 0 1vw;
  padding: 0 7px 0;
}

.favorite {
  cursor: pointer;
  display: flex;
  align-items: center;
  margin: 1vw 0 0 1vw;
  width: 7vw;
}

.favorite-text {
  font-family: "Manrope", sans-serif;
  font-size: 1.5vw;
  z-index: 1;
}

.favorite:hover .favorite-text {
  color: #8a8686;
}

.favorite-icon > svg {
  width: 30px !important;
  height: 30px !important;
}
.favorite-icon {
  margin: 1vw 0 0 0;
}

.filter-favorite {
}

.modal-content {
  color: black;
}

.header-spreadsheet-wrapper {
  margin: 0vw auto 10vw auto;
  background-size: cover;
  border-radius: 10px;
  width: 1200px;

  height: auto;
  text-align: center;
  align-items: center;
}

.header-title {
  font-family: "Julius Sans One", sans-serif;
  text-transform: uppercase;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  line-height: 1vw;
  font-size: 60px;
  color: white;
  position: absolute;
  top: 3vw;
  margin: 0 auto;
}
.header-title > p {
  padding-bottom: 3vw;
}

.header-spreadsheet {
  display: flex;
  justify-content: space-between;
  text-align: center;
  align-items: center;

  font-family: "Montserrat", sans-serif;
  font-weight: bold;
  font-size: 16px;
  margin: 6vw 0 0vw 0;
  padding: 0 0 1vw 0;
}

.header-spreadsheet > div {
  max-width: 11vw;
}

.head > div {
  margin: 0 1vw 0 17vw;
}

.spreadsheet {
  font-family: "Montserrat", sans-serif;
  font-weight: 400;
  font-size: 20px;
  height: 5vw;

  background: rgb(8, 73, 98);
  background: linear-gradient(
    90deg,
    rgba(8, 73, 98, 1) 20%,
    rgba(42, 96, 117, 0.9753035003063726) 42%,
    rgba(255, 255, 255, 1) 85%
  );
  border-radius: 33px;
  box-sizing: border-box;

  margin: 2vw 1vw 0 1vw;
  padding: 0.3vw 0;

  color: white;
}

.date {
  margin: 0 0 0 4vw !important;
}

.head-date {
  margin: 0 0 0 4vw !important;
}

.form-search {
  background: rgb(255, 255, 255);
  border: 1px solid black;
  border-right: none;
  border-radius: 25px 0 0 30px;
  color: black;
  font-family: "Manrope", sans-serif;
  font-size: 16px;
  padding: 0 0.5em;
  position: relative;
  height: 39px;
  width: 30vw;
}

input[type="text"] {
  font-size: 22px;
}

.weatherIcon {
  width: 3vw;
  height: 3vw;
  z-index: 10;
}

.button-wrapper {
  position: relative;
  align-items: center;
  justify-content: center;
  display: flex;
  flex-direction: row;
  margin: 1vw 0 0 0;
}

.spreadsheet-wrapper {
  position: relative;
  padding-bottom: 1vw;
  overflow: visible;
}

.button {
  cursor: pointer;
  height: 39px;
  width: 11vw;
  margin: 0;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 1);
  border-radius: 0 40px 40px 0;
}

.button-text {
  padding: 0.5vw 1vw;
  font-family: "Montserrat", sans-serif;
  font-weight: 400;
  color: black;
  text-transform: uppercase;
  position: relative;
  z-index: 2;
  /* box-shadow: 0 0 5px rgba(0, 0, 0, 0.4); */
}

.icon-delete {
  position: absolute;
  right: 2vw;
  top: 20%;
  align-self: center;
}

.button-favorite {
  position: absolute;
  top: 1vw;
  right: 6vw;
  width: 1.3vw;
  padding: 0 0px;
  border: none;
  background-color: transparent;
  cursor: pointer;
}

.button-delete {
  width: 1.2vw;
  height: auto;

  border: none;
  background: transparent;
  cursor: pointer;
  z-index: 2;
  padding: 0px 0px;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 1;
}

.modal-content {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  padding: 1rem;
  z-index: 2;
  border-radius: 1.5em;
}

.modal-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 1em;
}
.modal-buttons > button {
  text-decoration: none;
  border: none;
  background-color: #e7e7e7;
  color: black;
  border-radius: 1em;
  padding: 0 1em;
}

.modal-buttons > button:hover {
  background-color: #b3b2b2;
  cursor: pointer;
}

button,
button:active,
button:focus {
  outline: none;
}

/* Breakpoint for screens between 360px and 480px */
@media (min-width: 360px) and (max-width: 480px) {
  .bgImage {
    width: 100vw;
    height: 50vw;
  }

  .button-text {
    font-size: 15px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .button-wrapper {
    margin-top: 4vw;
  }
  .filter > input {
    width: 20vw;
    height: 8vw;
    font-size: 10px;
  }

  .header-title {
    font-size: 40px;
    top: 7vw;
    left: 4vw;
  }
  .header-title > p {
    padding-bottom: 8vw;
  }

  .spreadsheet {
    font-size: 16px;
    height: 64px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-right: 10vw;
    border-radius: 30px;
  }

  input[type="text"] {
    font-size: 18px;
  }

  .form-search {
    height: 40px;
    width: 50vw;
  }
  .button {
    height: 40px;
    width: 30vw;
  }
  .favorite-icon > svg {
    width: 6vw !important;
    height: 6vw !important;
    scale: 80%;
    margin-top: 1.8vw;
    margin-left: 0vw;
  }

  .filter {
    font-size: 5vw;
    margin: 5vw 0 0 3vw;
  }
  .favorite-text {
    font-size: 5vw;
    margin: 1vw 0 0 2vw;
  }

  .favorite {
  }

  .button-favorite {
    top: 5.2vw;
    right: 14vw;
    width: 5vw;
  }

  .button-delete {
    width: 5vw;
    margin-top: 5px;
  }

  .modal-content {
    width: 70%;
  }

  .spreadsheet-wrapper {
    height: 4vw;
    max-width: 100%;
    border-radius: 33px;
    margin: 2vw 2vw 0 2vw;
    padding-bottom: 20vw;
  }
}

/* Breakpoint for screens between 480px and 768px */
@media (min-width: 480px) and (max-width: 768px) {
  .bgImage {
    width: 100vw;
    height: 30vw;
  }

  .button-text {
    font-size: 18px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .button-wrapper {
    margin-top: 2vw;
  }
  .filter > input {
    width: 20vw;
    height: 6vw;
    font-size: 10px;
  }

  .header-title {
    font-size: 45px;
    top: 7vw;
    left: 4vw;
  }
  .header-title > p {
    padding-bottom: 6vw;
  }

  .spreadsheet {
    font-size: 20px;
    height: 64px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-right: 10vw;
    border-radius: 30px;
  }

  input[type="text"] {
    font-size: 22px;
  }

  .form-search {
    height: 45px;
    width: 50vw;
  }
  .button {
    height: 45px;
    width: 30vw;
  }
  .favorite-icon > svg {
    width: 6vw !important;
    height: 6vw !important;
    scale: 80%;
    margin-top: 1.8vw;
    margin-left: 0vw;
  }

  .filter {
    font-size: 4vw;
    margin: 5vw 0 0 3vw;
  }
  .favorite-text {
    font-size: 4vw;
    margin: 1vw 0 0 2vw;
  }

  .favorite {
  }

  .button-favorite {
    top: 20px;
    right: 14vw;
    width: 4vw;
  }

  .button-delete {
    width: 4vw;
    margin-top: 0px;
  }

  .modal-content {
    width: 70%;
  }

  .spreadsheet-wrapper {
    height: 4vw;
    max-width: 100%;
    border-radius: 33px;
    margin: 2vw 2vw 0 2vw;
    padding-bottom: 15vw;
  }
}

/* Breakpoint for screens between 768px and 1000px */
@media (min-width: 768px) and (max-width: 1000px) {
  .bgImage {
    width: 100vw;
    height: 30vw;
  }

  .button-text {
    font-size: 18px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .button-wrapper {
    margin-top: 2vw;
  }
  .filter > input {
    width: 20vw;
    height: 6vw;
    font-size: 10px;
  }

  .header-title {
    font-size: 50px;
    top: 7vw;
    left: 4vw;
  }
  .header-title > p {
    padding-bottom: 5vw;
  }

  .spreadsheet {
    font-size: 20px;
    height: 74px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-right: 10vw;
    border-radius: 30px;
  }

  input[type="text"] {
    font-size: 22px;
  }

  .form-search {
    height: 45px;
    width: 50vw;
  }
  .button {
    height: 45px;
    width: 30vw;
  }
  .favorite-icon > svg {
    width: 6vw !important;
    height: 6vw !important;
    scale: 90%;
    margin-top: 4.4vw;
    margin-left: 0vw;
  }

  .filter {
    font-size: 3vw;
    margin: 5vw 0 0 3vw;
  }
  .favorite-text {
    font-size: 3vw;
    margin: 1vw 0 0 2vw;
  }

  .favorite {
  }

  .button-favorite {
    top: 20px;
    right: 14vw;
    width: 3vw;
  }

  .button-delete {
    width: 3vw;
    margin-top: 5px;
  }

  .modal-content {
    width: 70%;
  }

  .spreadsheet-wrapper {
    height: 4vw;
    max-width: 100%;
    border-radius: 33px;
    margin: 2vw 2vw 0 2vw;
    padding-bottom: 10vw;
  }
}

/* Breakpoint for screens between 1000px and 1200px */
@media (min-width: 1000px) and (max-width: 1200px) {
  .header-title {
    font-size: 60px;
    top: 4vw;
    left: 2vw;
  }

  .header-title > p {
    padding-bottom: 4vw;
  }
  .filter {
    font-size: 2.5vw;
    margin: 1vw 0 0 3vw;
  }

  .filter > input {
    width: 12vw;
    height: 3.5vw;
    font-size: 10px;
  }

  input[type="text"] {
    font-size: 28px;
  }
  .form-search {
    height: 45px;
    width: 30vw;
  }
  .button {
    height: 45px;
    width: 15vw;
  }

  .button-text {
    font-size: 1.5vw;
  }

  .favorite-text {
    font-size: 2.5vw;
    margin: 1vw 0 0 2vw;
  }

  .favorite-icon > svg {
    height: 2vw !important;
    margin-top: 1vw;
  }

  .button-favorite {
    top: 2.1vw;
    right: 12vw;
    width: 2.8vw;
  }
  .button-delete {
    width: 2.6vw;
    margin-top: 5px;
  }

  .spreadsheet {
    font-size: 24px;
    height: 84px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-right: 10vw;
    border-radius: 35px;
  }
  .spreadsheet-wrapper {
    height: 4vw;
    max-width: 100%;
    border-radius: 33px;
    margin: 0 2vw 0 2vw;
    padding-bottom: 9vw;
  }
}

/* Breakpoint for screens large than 1200px*/
@media (min-width: 1200px) {
  .spreadsheet {
    height: 4vw;
    border-radius: 33px;
  }
  .spreadsheet {
    font-size: 20px;
    height: 84px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-right: 10vw;
    border-radius: 30px;
  }

  .button-favorite {
    top: 1.5vw;
    right: 12vw;
    width: 2vw;
  }
  .button-delete {
    width: 1.8vw;
    margin-top: 5px;
  }
  .favorite-icon > svg {
    height: 2vw !important;
    margin: 0vw 0 0 0.5vw;
  }
}
</style>
