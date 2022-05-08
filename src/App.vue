<template>
  <div class="btn btn-primary">ji3</div>
  <div class="container">
    <div id="map"></div>
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet.markercluster/dist/leaflet.markercluster';

export default {
  name: 'App',
  components: {},
  data() {
    return {
      rapidTestData: [],
      osmMap: null,
      location: {
        latitude: 22.6417929,
        longitude: 120.3320082,
      },
    };
  },
  methods: {
    getPharmacyData() {
      const url = 'https://raw.githubusercontent.com/SiongSng/Rapid-Antigen-Test-Taiwan-Map/data/data/antigen_open_street_map.json';
      return this.$http
        .get(url)
        .then((res) => res.data.features)
        .catch((err) => err);
    },
    initMap() {
      this.osmMap = L.map('map', {
        center: [this.location.latitude, this.location.longitude],
        zoom: 16,
      });
    },
    addMapLayer() {
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(this.osmMap);
    },
    getCurrentPosition() {
      return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(
          (position) => resolve(position),
          (error) => reject(error),
        );
      });
    },
    addMapMarker() {
      const markers = new L.MarkerClusterGroup().addTo(this.osmMap);
      this.rapidTestData.forEach((pharmacy) => {
        const { geometry } = pharmacy;
        markers.addLayer(L.marker([geometry.coordinates[1], geometry.coordinates[0]]));
      });
      this.osmMap.addLayer(markers);
    },
  },
  computed: {
    filterData() {
      return this.rapidTestData.filter(
        (pharmacy) => !pharmacy.properties.address.search('高雄市鳳山區'),
      );
    },
  },
  created() {},
  mounted() {
    async function renderMap() {
      try {
        // const position = await this.getCurrentPosition();
        this.rapidTestData = await this.getPharmacyData();
        // this.location.latitude = position.coords.latitude;
        // this.location.longitude = position.coords.longitude;
        this.initMap();
        this.addMapLayer();
        this.addMapMarker();
      } catch (err) {
        console.log(err);
      }
    }
    renderMap.bind(this)();
  },
};
</script>

<style lang="scss">
// @import 'node_modules\leaflet.markercluster\dist\MarkerCluster';
// @import 'node_modules\leaflet.markercluster\dist\MarkerCluster.Default';
@import './assets/scss/all';

#map {
  height: 50vh;
}
.marker-cluster-small {
  background-color: rgba(181, 226, 140, 0.6);
}
.marker-cluster-small div {
  background-color: rgba(110, 204, 57, 0.6);
}

.marker-cluster-medium {
  background-color: rgba(241, 211, 87, 0.6);
}
.marker-cluster-medium div {
  background-color: rgba(240, 194, 12, 0.6);
}

.marker-cluster-large {
  background-color: rgba(253, 156, 115, 0.6);
}
.marker-cluster-large div {
  background-color: rgba(241, 128, 23, 0.6);
}

.marker-cluster {
  background-clip: padding-box;
  border-radius: 20px;
}
.marker-cluster div {
  width: 30px;
  height: 30px;
  margin-left: 5px;
  margin-top: 5px;

  text-align: center;
  border-radius: 15px;
  font: 12px 'Helvetica Neue', Arial, Helvetica, sans-serif;
}
.marker-cluster span {
  line-height: 30px;
}
</style>
