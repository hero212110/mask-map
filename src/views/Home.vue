<template>
  <div style="height: 800px; width: 100%">
    <!-- <button @click="getLocation">Try It</button>
    <div style="height: 200px overflow: auto;">
      <p>First marker is placed at {{ withPopup.lat }}, {{ withPopup.lng }}</p>
      <p>Center is at {{ currentCenter }} and the zoom is: {{ currentZoom }}</p>
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
      style="height: 80%"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker :lat-lng="withPopup">
        <l-popup>
          <div @click="innerClick">
            I am a popup
            <p v-show="showParagraph">
              Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque
              sed pretium nisl, ut sagittis sapien. Sed vel sollicitudin nisi.
              Donec finibus semper metus id malesuada.
            </p>
          </div>
        </l-popup>
      </l-marker>
      <div v-if="Mask.length > 0">
        <l-marker
          v-for="item in items"
          :key="item.properties.id"
          :lat-lng="{
            lat: item.geometry.coordinates[1],
            lng: item.geometry.coordinates[0]
          }"
        >
          <l-popup>
            <div @click="innerClick">
              {{ item.properties.name }}
              <p>成人口罩:{{ item.properties.mask_adult }}</p>
              <p>兒童口罩:{{ item.properties.mask_child }}</p>

              <!-- <p v-show="showParagraph">
                Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque
                sed pretium nisl, ut sagittis sapien. Sed vel sollicitudin nisi.
                Donec finibus semper metus id malesuada.
              </p> -->
            </div>
          </l-popup>
        </l-marker>
      </div>
      <!-- <l-marker :lat-lng="withTooltip">
        <l-tooltip :options="{ permanent: true, interactive: true }">
          <div @click="innerClick">
            I am a tooltip
            <p v-show="showParagraph">
              Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque
              sed pretium nisl, ut sagittis sapien. Sed vel sollicitudin nisi.
              Donec finibus semper metus id malesuada.
            </p>
          </div>
        </l-tooltip>
      </l-marker> -->
    </l-map>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer, LMarker, LPopup, LTooltip } from "vue2-leaflet";
import axios from "axios";
export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup
    // LTooltip
  },
  data() {
    return {
      zoom: 16,
      center: latLng(47.41322, -1.219482),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      withPopup: latLng(47.41322, -1.219482),
      withTooltip: latLng(47.41422, -1.250482),
      currentZoom: 11.5,
      currentCenter: latLng(47.41322, -1.219482),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5
      },
      showMap: true,
      Mask: []
    };
  },
  mounted() {
    this.getLocation();
    this.getData();
  },
  watch: {
    // currentCenter(val) {
    //   console.log(val);
    //   this.centerUpdate(val);
    // }
  },
  computed: {
    items() {
      let tmp = [];
      for (let i = 0; i < 20; i++) {
        tmp.push(this.Mask[i]);
      }
      console.log(tmp);
      return tmp;
    }
  },
  methods: {
    async getData() {
      await axios
        .get(
          "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json"
        )
        .then(res => {
          console.log(res.data.features);
          this.Mask = res.data.features;
        });
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      console.log(center);
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      alert("Click!");
    },
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.showPosition);
      }
    },
    showPosition(position) {
      console.log(position.coords.latitude, position.coords.longitude);
      this.withPopup = {
        lat: position.coords.latitude,
        lng: position.coords.longitude
      };

      this.centerUpdate({
        lat: position.coords.latitude,
        lng: position.coords.longitude
      });

      this.center = {
        lat: position.coords.latitude,
        lng: position.coords.longitude
      };
    }
  }
};
</script>
