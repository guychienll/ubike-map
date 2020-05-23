<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="city" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select v-model="select.city" id="city" class="form-control">
              <option :value="city.name" :key="city.name" v-for="city in cityName">
                {{ city.name }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="dist" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select v-model="select.dist" id="dist" class="form-control">
              <option
                :value="district.name"
                :key="district.name"
                v-for="district in cityName.find(city => city.name === select.city).districts"
              >
                {{ district.name }}
              </option>
            </select>
          </div>
        </div>
      </div>
      <!-- 顯示地圖和 UBike 站點 -->
      <div class="col-sm-9">
        <div id="map">
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import cityName from "./assets/cityName.json";

export default {
  name: "App",
  data: () => ({
    cityName,
    select: {
      city: "臺北市",
      dist: "中正區"
    },
    ubikes: [],
    OSMap: {}
  }),
  computed: {
    getUbikes() {
      return this.ubikes.filter((u) => (
        u.sarea === this.select.dist
      ));
    }
  },
  watch: {
    getUbikes() {
      this.updateMap();
    }
  },
  methods: {
    updateMap() {
      this.OSMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          this.OSMap.removeLayer(layer);
        }
      });
      this.getUbikes.forEach((u) => {
        L.marker([u.lat, u.lng]).bindPopup(`<p><strong style="font-size: 20px;">${u.sna}</strong></p>
 <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${u.sbi} 台</strong><br>
 可停空位剩餘: ${u.bemp}<br>
 <small>最後更新時間: ${u.mday}</small>`).addTo(this.OSMap);
      });
      this.cityName[0].districts.forEach(((dist) => {
        if (dist.name === this.select.dist) {
          this.OSMap.flyTo(new L.LatLng(dist.latitude, dist.longitude), 14);
        }
      }));
    }
  },
  created() {
    const url = "https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json";
    this.axios.get(url).then((response) => {
      this.ubikes = Object.keys(response.data.retVal).map((key) => (response.data.retVal[key]));
    });
  },
  mounted() {
    this.OSMap = L.map("map", {
      center: [25.041956, 121.508791],
      zoom: 18
    });

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "Map data &copy; <a href='https://www.openstreetmap.org/'>OpenStreetMap</a> contributors, <a href='https://creativecommons.org/licenses/by-sa/2.0/'>CC-BY-SA</a>, Imagery © <a href='  https://www.mapbox.com/'>Mapbox</a>",
      maxZoom: 18
    }).addTo(this.OSMap);
  }
};
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";

#map {
  height: 100vh;
  position: relative;
}
</style>
