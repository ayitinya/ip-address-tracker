<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'
import leaflet from 'leaflet'
import "leaflet/dist/leaflet.css";
import locationIcon from "../assets/svgs/icon-location.svg"


interface IPAddress {
  ip: string,
  location: {
    country: string
    region: string
    city: string
    lat: number
    lng: number
    postalCode: string
    timezone: string
    geonameId: number
  },
  domains: string[]
  as: {
    asn: number
    name: string
    route: string
    domain: string
    type: string
  },
  isp: string
}

const ip = ref("")
const key = import.meta.env.VITE_GEOLOCATION_API_KEY
const data = ref<null | IPAddress>(null)
const mapRef = ref<HTMLInputElement | null>(null)

let map: leaflet.Map;
onMounted(() => {
  map = leaflet.map(mapRef.value!.id).setView([51.505, -0.09], 13);
  leaflet.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenStreetMap'
  }).addTo(map);
})


const searchByIp = async () => {

  const res = await axios.get<IPAddress>("https://geo.ipify.org/api/v2/country,city",
    { params: { apiKey: key, ipAddress: ip.value } })
  data.value = res.data
  map.setView([res.data.location.lat, res.data.location.lng], 15);
  const marker = leaflet.marker([res.data.location.lat, res.data.location.lng], {
    icon: leaflet.icon({
      iconUrl: locationIcon,
    })
  }).addTo(map)
  marker.bindPopup(`<b>${res.data.location.city}</b>`).openPopup()
}
</script>

<template>
  <main>
    <section class="header">
      <h1>IP Address Tracker</h1>
      <form v-on:submit.prevent="searchByIp">
        <fieldset>
          <label for="search-box" id="search-label">Search</label>
          <input type="text" id="search-box" placeholder="192.212.714.101" v-model="ip">
          <button type="submit"></button>
        </fieldset>
      </form>
    </section>
    <template v-if="!data">
      <section class="location-details">
        <div>
          <h4>IP ADDRESS</h4>
          <h2>192.123.45.23</h2>
        </div>
        <div>
          <h4>LOCATION</h4>
          <h2>Tamale</h2>
        </div>
        <div>
          <h4>TIMEZONE</h4>
          <h2>UTC -04:00</h2>
        </div>
        <div>
          <h4>ISP</h4>
          <h2>SpaceX Starlink</h2>
        </div>
      </section>
    </template>
    <template v-else>
      <section class="location-details">
        <div>
          <h4>IP ADDRESS</h4>
          <h2>{{ data.ip }}</h2>
        </div>
        <div>
          <h4>LOCATION</h4>
          <h2>{{ data.location.city }}</h2>
        </div>
        <div>
          <h4>TIMEZONE</h4>
          <h2>UTC {{ data.location.timezone }}</h2>
        </div>
        <div>
          <h4>ISP</h4>
          <h2>{{ data.isp }}</h2>
        </div>
      </section>
    </template>
    <section id="map" ref="mapRef"></section>
  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Rubik:wght@400;500;700&display=swap');


main {
  font-family: 'Rubik', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100vh;
  text-align: center;
  color: hsl(0, 0%, 17%);
}

.header {
  background-image: url(../assets/images/pattern-bg.png);
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2.5rem 0;
  padding-bottom: 10rem;
}

#search-box {
  padding: 1.125rem;
  flex: 1;
  border-radius: 10px 0px 0px 10px;
  border: none;
  font-size: 18px;
  font-weight: 400;
  font-family: 'Rubik', sans-serif;
}

form {
  width: 100%;
  display: flex;
  justify-content: center;
}

fieldset {
  border: none;
  width: 80%;
  display: flex;
  margin: 0;
  padding: 0;
}

#search-label {
  display: none;
}

input:active {
  border: none;
}

button {
  width: 50px;
  border: none;
  border-radius: 0px 10px 10px 0px;
  color: white;
  background: url(../assets/svgs/icon-arrow.svg) no-repeat center black;
}

.location-details {
  background-color: white;
  border-radius: 10px 10px 10px 10px;
  width: 80%;
  position: absolute;
  top: 11rem;
  z-index: 9999;
}

h4 {
  margin-bottom: 0;
  font-size: 0.9rem;
  font-weight: 400;
  color: hsl(0, 0%, 59%);
}

h1,
h2 {
  margin-top: 0.125rem;
  font-weight: 500;
}

h1 {
  color: white;
}

#map {
  flex: 1;
  width: 100%;
}

@media screen and (min-width: 1008px) {
  .location-details {
    display: flex;
    justify-content: space-evenly;
    padding: 2rem 0;
    top: 14rem;
    text-align: initial;
  }

  .location-details > div {
    border-left: 1px solid hsla(0, 0%, 17%, 0.1);
    padding-left: 1.5rem;
  }

  .location-details > div:first-child {
    border: none;
    padding: 0;
  }

  fieldset {
    width: 30%;
  }
}
</style>