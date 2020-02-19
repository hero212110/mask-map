<template>
  <div :style="windowHeight">
    <!-- <button @click="getLocation">Try It</button>
          <div style="height: 200px overflow: auto;">
            <p>
              First marker is placed at {{ withPopup.lat }}, {{ withPopup.lng }}
            </p>
            <p>
              Center is at {{ currentCenter }} and the zoom is:
              {{ currentZoom }}
            </p>
            <button @click="showLongText">
              Toggle long popup
            </button>
            <button @click="showMap = !showMap">
              Toggle map
            </button>
          </div> -->
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 100%"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker :lat-lng="withPopup" :icon="icons.red">
        <l-popup>
          <div @click="innerClick">
            當前位置
            <p v-show="showParagraph">
              Lorem
            </p>
          </div>
        </l-popup>
      </l-marker>
      <div v-if="Mask.length > 0 && currMaskType == 'mdi-face'">
        <l-marker
          v-for="item in items"
          :key="item.properties.id"
          :icon="
            item.properties.mask_adult >= 100
              ? icons.green
              : item.properties.mask_adult >= 50 &&
                item.properties.mask_adult <= 99
              ? icons.orange
              : icons.grey
          "
          :lat-lng="{
            lat: item.geometry.coordinates[1],
            lng: item.geometry.coordinates[0]
          }"
        >
          <l-popup>
            <div style="font-family:Microsoft JhengHei">
              <p><v-icon>mdi-store-outline</v-icon>藥局:{{ item.properties.name }}</p>
              <p><v-icon>mdi-map-marker</v-icon>地址:{{ item.properties.address }}</p>
              <p><v-icon>mdi-phone</v-icon>電話:{{ item.properties.phone }}</p>
              <v-divider></v-divider>
              <p><v-icon>mdi-face</v-icon>成人口罩:{{ item.properties.mask_adult }} </p>
              <p><v-icon>mdi-baby-face-outline</v-icon>兒童口罩:{{ item.properties.mask_child }}</p>
            </div>
          </l-popup>
        </l-marker>
      </div>
      <div v-if="Mask.length > 0 && currMaskType == 'mdi-baby-face-outline'">
        <l-marker
          v-for="item in items"
          :key="item.properties.id"
          :icon="
            item.properties.mask_child >= 100
              ? icons.green
              : item.properties.mask_child >= 50 &&
                item.properties.mask_child <= 99
              ? icons.orange
              : icons.grey
          "
          :lat-lng="{
            lat: item.geometry.coordinates[1],
            lng: item.geometry.coordinates[0]
          }"
        >
          <l-popup>
            <div style="font-family:Microsoft JhengHei">
              <p><v-icon>mdi-store-outline</v-icon>藥局:{{ item.properties.name }}</p>
              <p><v-icon>mdi-map-marker</v-icon>地址:{{ item.properties.address }}</p>
              <p><v-icon>mdi-phone</v-icon>電話:{{ item.properties.phone }}</p>
              <v-divider></v-divider>
              <p><v-icon>mdi-face</v-icon>成人口罩:{{ item.properties.mask_adult }} </p>
              <p><v-icon>mdi-baby-face-outline</v-icon>兒童口罩:{{ item.properties.mask_child }}</p>
            </div>
          </l-popup>
        </l-marker>
      </div>

      <l-control>
        <v-btn fab dark small color="red" @click="changeMaskType">
          <v-icon>{{ currMaskType }}</v-icon>
        </v-btn>
      </l-control>

      <l-control position="bottomright">
        <v-speed-dial v-model="fab" right direction="top">
          <template v-slot:activator>
            <v-btn v-model="fab" color="blue darken-2" dark fab>
              <v-icon v-if="fab">mdi-close</v-icon>
              <v-icon v-else>mdi-cogs</v-icon>
            </v-btn>
          </template>
          <v-btn fab dark small color="green" @click="getData">
            <v-icon>mdi-reload</v-icon>
          </v-btn>

          <v-btn fab dark small color="red" @click="pageReload">
            <v-icon>mdi-crosshairs-gps</v-icon>
          </v-btn>
        </v-speed-dial>
      </l-control>
    </l-map>

    <div></div>
  </div>
</template>

<script>
import { latLng, icon } from "leaflet";
import {
  LMap,
  LTileLayer,
  LMarker,
  LPopup,
  LTooltip,
  LControl
} from "vue2-leaflet";

import axios from "axios";

export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LControl
    // LTooltip
  },
  data() {
    return {
      windowHeight: {
        height: window.innerHeight + "px"
      },
      currMaskType: "mdi-face",
      zoom: 16,
      center: latLng(47.41322, -1.219482),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      withPopup: latLng(47.41322, -1.219482),
      withTooltip: latLng(47.41422, -1.250482),
      currentZoom: 16,
      currentCenter: latLng(47.41322, -1.219482),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5
      },
      showMap: true,
      Mask: [],
      position: {
        latitude: "",
        longitude: ""
      },
      icons: {
        green: icon({
          iconUrl:
            "https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png",
          iconSize: [32, 37],
          iconAnchor: [16, 37]
        }),
        orange: icon({
          iconUrl:
            "https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-orange.png",
          iconSize: [32, 37],
          iconAnchor: [16, 37]
        }),
        grey: icon({
          iconUrl:
            "https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-grey.png",
          iconSize: [32, 37],
          iconAnchor: [16, 37]
        }),
        red: icon({
          iconUrl:
            "https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png",
          iconSize: [32, 37],
          iconAnchor: [16, 37]
        })
      },
      fab: false
    };
  },
  mounted() {
    this.getLocation();
    // this.getData();
  },
  watch: {
    currentCenter(val) {
      this.centerUpdate(val);
      this.getData();
    },
    // center(val) {
    //   console.log(val);
    // }
  },
  computed: {
    items() {
      let tmp = [];
      this.Mask.forEach(item => {
        if (
          Math.abs(item.geometry.coordinates[1] - this.currentCenter.lat) <
            0.01 &&
          Math.abs(item.geometry.coordinates[0] - this.currentCenter.lng) < 0.01
        ) {
          tmp.push(item);
        }
      });
      return tmp;
    }
  },
  methods: {
    pageReload() {
      location.reload();
    },
    changeMaskType() {
      this.currMaskType == "mdi-face"
        ? (this.currMaskType = "mdi-baby-face-outline")
        : (this.currMaskType = "mdi-face");
    },
    async getData() {
      await axios
        .get(
          "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json"
        )
        .then(res => {
          // console.log(res.data.features);
          this.Mask = res.data.features;
        });
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      // console.log(center);
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      // alert("Click!");
    },
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.showPosition);
      }
    },
    showPosition(position) {
      // console.log(position.coords.latitude, position.coords.longitude);
      this.position.latitude = position.coords.latitude;
      this.position.longitude = position.coords.longitude;
      let tmp = latLng(position.coords.latitude, position.coords.longitude);
      this.withPopup = tmp;
      this.centerUpdate(tmp);
      this.center = tmp;
    }
  }
};
</script>
<style></style>
