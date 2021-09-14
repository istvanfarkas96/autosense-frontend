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
                <div class="editable-content">
                  <p v-if="!showEditForm" class="title is-4">{{ selectedMarker.name }}</p>
                  <v-text-field v-if="showEditForm" v-model="selectedMarker.name"></v-text-field>
                  <div class="content pt-4">
                    <h3>Prices</h3>
                    <div v-for="price in selectedMarker.prices">
                      <p v-if="!showEditForm">{{ price.product_id }} - {{ price.currency }} <strong>{{ price.price }}</strong></p>
                      <p v-if="showEditForm">{{ price.product_id }} - {{ price.currency }} <v-text-field v-model="price.price"></v-text-field></p>
                    </div>
                  </div>
                </div>
                <h3>Pump statuses</h3>
                <div v-for="pump in selectedMarker.pumps">
                  <br>
                  <p class="font-weight-bold mb-2">{{ pump.product_id }}</p>
                  <div v-for="point in pump.points">
                    <p class="mb-1">pump nr.{{ point.id }}: {{ point.status }}</p>
                  </div>
                </div>
              </div>
              <v-btn v-if="!showEditForm" class="mt-4 mb-3 justify-center" v-on:click="showEditForm = true">
                Edit
                <v-icon
                    left
                    class="pl-5"
                >
                  mdi-account-edit-outline
                </v-icon>
              </v-btn>
              <v-btn v-if="showEditForm" class="mt-4 mb-3 justify-center" v-on:click="updateStation()">
                Save
              </v-btn>
              <v-btn class="mt-4 mb-3 ml-3 justify-center" v-on:click="deleteFuelStation()">
                Delete
              </v-btn>
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
            <h3>Pumps available</h3>
            <v-row class="pt-3" v-for="i in pumpCounter">
              <v-col cols="5">
                Pump {{ i }}
                <v-switch
                    v-model="pumpAvailability[i-1].status"
                    value="available"
                ></v-switch>
              </v-col>
              <v-col>
                <v-select
                    class="mr-3 ml-3"
                    v-model="pumpAvailability[i-1].product_id"
                    :items="items"
                    label="Fuel type"
                    required
                ></v-select>
              </v-col>
            </v-row>
            <v-btn
                small
                class="ma-2 mt-8"
                @click="addPump()"
            >
              <v-icon
                  left
              >
                mdi-plus
              </v-icon>
              Add pump
            </v-btn>
          </div>
          <v-btn
              class="mr-4"
              @click="createNewFuelStation()"
          >
            submit
          </v-btn>
        </form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import config from '../config/index.ts'

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
      pumpCounter: 0,
      showEditForm: false,
      prices: [],
      markers: [],
      gasStations: [],
      infoContent: '',
      products: [],
      pumpAvailability: [],
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
    updateStation() {
      this.showEditForm = false;
      this.selectedMarker.prices.forEach(item => {
        item.price = parseFloat(item.price)
      })
      axios.put(config.apiBaseUrl + '/fuel-stations/' + this.selectedMarker.id, {
        id: this.selectedMarker.id,
        name: this.selectedMarker.name,
        address: this.selectedMarker.address,
        city: this.selectedMarker.city,
        latitude: this.selectedMarker.position.lat,
        longitude: this.selectedMarker.position.lng,
        prices: this.selectedMarker.prices,
        products: this.selectedMarker.pumps
      }, {
        auth: config.auth
      })
    },

    addPump() {
      this.pumpAvailability[this.pumpCounter] = {id: this.pumpCounter + 1, status: 'not available'};
      this.pumpCounter++;
    },

    addFuelType() {
      const item = {
        'price': parseFloat(this.selectedFuelPrice),
        'currency': 'CHF',
        'product_id': this.selectedFuelType
      }

      this.prices.push(item);
      this.selectedFuelPrice = '';
      this.selectedFuelType = '';
    },

    buildProductsObject() {
      this.pumpAvailability.forEach((item, index) => {
        this.products[index] = {
          "product_id": item.product_id,
          "points": [
            {
              "id": item.id.toString(),
              "status": item.status
            }]
        };
      })
    },

    createNewFuelStation() {
      this.addFuelType();
      this.buildProductsObject();
      axios.post(config.apiBaseUrl + '/fuel-stations', {
        name: this.fuelStationName,
        city: this.fuelStationCity,
        address: this.fuelStationAddress,
        latitude: parseFloat(this.stationLatitude),
        longitude: parseFloat(this.stationLongitude),
        prices: this.prices,
        products: this.products
      }, {
        auth: config.auth
      }).then(response => {
        this.clearInputs();
        this.getGasStations();
      })
    },

    clearInputs() {
      this.fuelStationName = '';
      this.fuelStationCity = '';
      this.fuelStationAddress = '';
      this.stationLatitude = '';
      this.stationLongitude = ''
      this.prices = [];
      this.pumpAvailability = [];
      this.pumpCounter = 0;
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
      this.gasStations = [];
      axios.get(config.apiBaseUrl + '/fuel-stations', {
        auth: config.auth
      }).then(response => {
        if (response.data.length === 0) {
          return;
        }

        this.gasStations = response.data
        this.setupMarkers();
      })
    },

    setupMarkers() {
      this.markers = [];
      this.gasStations.forEach(station => {
        const marker = {
          id: station.id,
          name: station.name,
          address:station.address,
          city: station.city,
          position: {
            lat: station.latitude,
            lng: station.longitude
          },
          prices: station.prices,
          pumps: station.products
        }

        this.markers.push(marker);
      })
    },

    deleteFuelStation() {
      this.infoWinOpen = false;
      axios.delete(config.apiBaseUrl + '/fuel-stations/' + this.selectedMarker.id, {
        auth: config.auth
      }).then(response => {
        this.markers.forEach((item, index) => {
          if (item.id === this.selectedMarker.id) {
            this.markers.splice(index, 1)
          }
        })
      })
    }
  }
}
</script>
