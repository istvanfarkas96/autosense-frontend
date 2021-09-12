<template>
  <v-container class="grey lighten-5" fluid>
    <v-row>
      <v-col class="pr-2" cols="9">
        <GmapMap
            :center="{lat:47.37795601653243, lng:8.541298000076841}"
            :zoom="11"
            map-type-id="terrain"
            style="width: 100%; height: 90vh"
        >
          <GmapMarker
              :key="index"
              v-for="(m, index) in markers"
              :position="m.position"
              :clickable="true"
              v-on:click="toggleInfoWindow(m,index)"
          />
          <GmapInfoWindow
              :position="infoWindowPos"
              :opened="infoWinOpen"
              @closeclick="infoWinOpen=false"
          >
            <div class="card">
              <div class="card-content">
                <div class="media">
                  <div class="media-content">
                    <p class="title is-4">{{ selectedMarker.name }}</p>
                  </div>
                </div>
                <div class="content">
                  <h3>Prices</h3>
                  <div v-for="price in selectedMarker.prices">
                    <p>{{ price.product_id }} - {{ price.currency }} <strong>{{ price.price }}</strong></p>
                  </div>
                  <h3>Pump statuses</h3>
                  <div v-for="pump in selectedMarker.pumps">
                    <br>
                    <p class="font-weight-bold mb-2">{{ pump.product_id }}</p>
                    <div v-for="point in pump.points">
                      <p class="mb-1">Pomp nr.{{ point.id }}: {{ point.status }}</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </GmapInfoWindow>
        </GmapMap>
      </v-col>
      <v-col cols="3">
        <form>
          <h3 class="pb-5 text-center">Add new fuel station</h3>
          <v-text-field
              v-model="fuelStationName"
              label="Name"
              required
          ></v-text-field>
          <v-text-field
              v-model="fuelStationCity"
              label="City"
              required
          ></v-text-field>
          <v-text-field
              v-model="fuelStationAddress"
              label="Address"
              required
          ></v-text-field>
          <v-row class="mt-3">
            <v-col cols="6">
              <v-text-field
                  type="number"
                  class="ml-3 mr-3"
                  v-model="stationLatitude"
                  label="Latitude"
                  required
              ></v-text-field>
            </v-col>
            <v-col cols="6">
              <v-text-field
                  type="number"
                  class="mb-8 mr-3"
                  v-model="stationLongitude"
                  label="Longitude"
                  required
              ></v-text-field>
            </v-col>
          </v-row>
          <h3 class="pt-3 pb-0 text-center">Set prices</h3>
          <v-row v-for="item in prices" v-if="prices.length < 2">
            <v-col cols="6">
              <v-select
                  class="mr-3 ml-3"
                  v-model="item.product_id"
                  :items="items"
                  label="Fuel type"
                  required
              ></v-select>
            </v-col>
            <v-col cols="6">
              <v-text-field
                  type="number"
                  class="mb-8 mr-3"
                  v-model="item.price"
                  label="Price"
                  required
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="6">
              <v-select
                  class="mr-3 ml-3"
                  v-model="selectedFuelType"
                  :items="items"
                  label="Fuel type"
                  required
              ></v-select>
            </v-col>
            <v-col cols="6">
              <v-text-field
                  class="mb-8 mr-3"
                  v-model="selectedFuelPrice"
                  label="Price"
                  required
              ></v-text-field>
            </v-col>
          </v-row>
          <div v-if="prices.length < 1">
            <v-btn
                small
                class="ma-2"
                @click="addFuelType()"
            >
              <v-icon
                  left
              >
                mdi-plus
              </v-icon>
              Add more fuel type
            </v-btn>

          </div>
          <div class="mb-5">
            <h3>Pomps available</h3>
            <v-row v-for="i in pompCounter">
              <v-col class="">Pomp {{ i }}

              </v-col>
            </v-row>
          </div>
          <v-btn
              class="mr-4"
              @click="createNewFuelStation()"
          >
            submit
          </v-btn>
          <v-btn @click="">
            clear
          </v-btn>
        </form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Home',
  components: {},
  data() {
    return {
      fuelStationName: '',
      fuelStationCity: '',
      fuelStationAddress: '',
      stationLatitude: '',
      stationLongitude: '',
      selectedFuelType: '',
      selectedFuelPrice: '',
      pompCounter: 1,
      prices: [],
      markers: [],
      gasStations: [],
      infoContent: '',
      infoWindowPos: {
        lat: 50,
        lng: 0
      },
      infoWinOpen: false,
      currentMidx: null,
      infoOptions: {
        pixelOffset: {
          width: 0,
          height: -35
        }
      },
      selectedMarker: '',
      items: [
        'Diesel',
        'Benzin',
      ]
    }
  },

  created() {
    this.getGasStations();
  },

  methods: {
    addFuelType() {
      const item = {
        'price': this.selectedFuelPrice,
        'currency': 'CHF',
        'product_id': this.selectedFuelType
      }

      this.prices.push(item);
      this.selectedFuelPrice = '';
      this.selectedFuelType = '';
    },

    createNewFuelStation() {
      this.addFuelType();
      axios.post('http://127.0.0.1:3000/api/fuel-stations', {
        name: this.fuelStationName,
        city: this.fuelStationCity,
        address: this.fuelStationAddress,
        latitude: this.stationLatitude,
        longitude: this.stationLongitude,
        prices: this.prices
      })
    },

    toggleInfoWindow: function (marker, idx) {
      this.infoWindowPos = marker.position;
      this.selectedMarker = marker;

      if (this.currentMidx === idx) {
        this.infoWinOpen = !this.infoWinOpen;
      } else {
        this.infoWinOpen = true;
        this.currentMidx = idx;
      }
    },

    getGasStations() {
      axios.get('http://127.0.0.1:3000/api/fuel-stations').then(response => {
        if (response.data.length === 0) {
          return;
        }

        this.gasStations = response.data
        this.setupMarkers();
      })
    },

    setupMarkers() {
      this.gasStations.forEach(station => {
        const marker = {
          position: {
            lat: station.latitude,
            lng: station.longitude
          },
          name: station.name,
          prices: station.prices,
          pumps: station.products
        }

        this.markers.push(marker);
      })
    },
  }
}
</script>
