<template>
  <div class="container">
        <div id="map" class=""></div>

  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet.markercluster/dist/leaflet.markercluster';

let osmMap = {};
const greenIcon = new L.Icon({
  iconUrl:
    'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-green.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});
const redIcon = new L.Icon({
  iconUrl:
    'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-red.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});
const yellowIcon = new L.Icon({
  iconUrl:
    'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-yellow.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});

export default {
  name: 'App',
  components: {},
  data() {
    return {
      rapidTestData: [],
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
      osmMap = L.map('map', {
        center: [this.location.latitude, this.location.longitude],
        zoom: 16,
      });
    },
    addMapLayer() {
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(osmMap);
    },
    getCurrentPosition() {
      return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(
          (position) => resolve(position),
          (error) => reject(error),
        );
      });
    },
    addMapMarkers() {
      const markers = new L.MarkerClusterGroup().addTo(osmMap);
      const myLocation = L.latLng(this.location.latitude, this.location.longitude);

      this.greenfliterData.forEach((pharmacy) => {
        const { geometry, properties } = pharmacy;
        const marker = L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
          icon: greenIcon,
        });
        markers.addLayer(
          marker.bindPopup(
            `<table class="table d-flex flex-column table-borderless">
              <tbody>
              <tr class="row g-0 mb-4">
                  <th class="col-12 text-shadow-white text-break px-0"><h3 class="d-flex fw-bold text-white mb-0">${
  properties.name
}<span class="badge bg-primary fs-7 align-self-end h-50 ms-1">${
  myLocation.distanceTo(marker.getLatLng()).toPrecision(2) / 1000
} 公里</span></h3></th>
              </tr>
              <tr class="row g-0">
                <td class="px-0 z-index-4">
                  <div class="badge rounded-circle bg-secondary d-flex justify-content-around align-items-center
                  fs-6 fw-blod text-primary text-shadow text-break w-100 position-relative z-index-4 px-0 py-2">
                    <span class="z-index-4">剩</span>
                    <span class="display-1 z-index-4">${properties.count}</span>
                    <span class="z-index-4 ms-2">組</span>
                    <div class="badge bg-warning display-6 fw-bolder text-shadow text-break opacity-75
                    position-absolute top-0 start-50 translate-middle z-index-4">
                    ${properties.brands[0].search('羅氏') >= 0 ? '羅氏快篩' : '品牌未知'}</div>
                </td>
              <tr class="row g-0">
                <td class="fs-7 fw-bolder text-white text-center text-shadow-white text-break px-0 z-index-4">${
  properties.note === null ? '無註備' : properties.note
}</td>
              </tr>
              </tr>
                <tr class="row g-0">
                  <td class="text-shadow text-break text-center d-flex flex-column align-items-center px-0"><a class="text-white text-decoration-none z-index-4" href="https://www.google.com.tw/maps/place/${
  properties.address
}" target="_blank">${properties.address}</a></td>
                </tr>
                <tr class="row g-0 mb-4">
                  <td class="text-shadow text-white text-break text-center d-flex flex-column align-items-center px-0 z-index-4">${
  properties.phone
}</td>
                </tr>
                <tr class="row g-0">
                  <td class="d-flex justify-content-center text-break fst-italic text-light p-0"><small class="opacity-50 z-index-4">最後更新時間： ${
  properties.updated_at
}</small></td>
                </tr>
              </tbody>
            </table>`,
          ),
        );
      });
      this.redfliterData.forEach((pharmacy) => {
        const { geometry, properties } = pharmacy;
        const marker = L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
          icon: redIcon,
        });
        markers.addLayer(
          marker.bindPopup(
            `<table class="table d-flex flex-column table-borderless">
              <tbody>
              <tr class="row g-0 mb-4">
                  <th class="col-12 text-shadow-white text-break px-0"><h3 class="d-flex fw-bold text-white mb-0">${
  properties.name
}<span class="badge bg-primary fs-7 align-self-end h-50 ms-1">${
  myLocation.distanceTo(marker.getLatLng()).toPrecision(2) / 1000
} 公里</span></h3></th>
              </tr>
              <tr class="row g-0">
                <td class="px-0 z-index-4">
                  <div class="badge rounded-circle bg-secondary d-flex justify-content-around align-items-center
                  fs-6 fw-blod text-primary text-shadow text-break w-100 position-relative z-index-4 px-0 py-2">
                    <span class="z-index-4">剩</span>
                    <span class="display-1 z-index-4">${properties.count}</span>
                    <span class="z-index-4 ms-2">組</span>
                    <div class="badge bg-warning display-6 fw-bolder text-shadow text-break opacity-75
                    position-absolute top-0 start-50 translate-middle z-index-4">
                    ${properties.brands[0].search('羅氏') >= 0 ? '羅氏快篩' : '品牌未知'}</div>
                </td>
              <tr class="row g-0">
                <td class="fs-7 fw-bolder text-white text-center text-shadow-white text-break px-0 z-index-4">${
  properties.note === null ? '無註備' : properties.note
}</td>
              </tr>
              </tr>
                <tr class="row g-0">
                  <td class="text-shadow text-break text-center d-flex flex-column align-items-center px-0"><a class="text-white text-decoration-none z-index-4" href="https://www.google.com.tw/maps/place/${
  properties.address
}" target="_blank">${properties.address}</a></td>
                </tr>
                <tr class="row g-0 mb-4">
                  <td class="text-shadow text-white text-break text-center d-flex flex-column align-items-center px-0 z-index-4">${
  properties.phone
}</td>
                </tr>
                <tr class="row g-0">
                  <td class="d-flex justify-content-center text-break fst-italic text-light p-0"><small class="opacity-50 z-index-4">最後更新時間： ${
  properties.updated_at
}</small></td>
                </tr>
              </tbody>
            </table>`,
          ),
        );
      });

      this.yellowfliterData.forEach((pharmacy) => {
        const { geometry, properties } = pharmacy;
        const marker = L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
          icon: yellowIcon,
        });
        markers.addLayer(
          marker.bindPopup(
            `<table class="table d-flex flex-column table-borderless">
              <tbody>
              <tr class="row g-0 mb-4">
                  <th class="col-12 text-shadow-white text-break px-0"><h3 class="d-flex fw-bold text-white mb-0">${
  properties.name
}<span class="badge bg-primary fs-7 align-self-end h-50 ms-1">${
  myLocation.distanceTo(marker.getLatLng()).toPrecision(2) / 1000
} 公里</span></h3></th>
              </tr>
              <tr class="row g-0">
                <td class="px-0 z-index-4">
                  <div class="badge rounded-circle bg-secondary d-flex justify-content-around align-items-center
                  fs-6 fw-blod text-primary text-shadow text-break w-100 position-relative z-index-4 px-0 py-2">
                    <span class="z-index-4">剩</span>
                    <span class="display-1 z-index-4">${properties.count}</span>
                    <span class="z-index-4 ms-2">組</span>
                    <div class="badge bg-warning display-6 fw-bolder text-shadow text-break opacity-75
                    position-absolute top-0 start-50 translate-middle z-index-4">
                    ${properties.brands[0].search('羅氏') >= 0 ? '羅氏快篩' : '品牌未知'}</div>
                </td>
              <tr class="row g-0">
                <td class="fs-7 fw-bolder text-white text-center text-shadow-white text-break px-0 z-index-4">${
  properties.note === null ? '無註備' : properties.note
}</td>
              </tr>
              </tr>
                <tr class="row g-0">
                  <td class="text-shadow text-break text-center d-flex flex-column align-items-center px-0"><a class="text-white text-decoration-none z-index-4" href="https://www.google.com.tw/maps/place/${
  properties.address
}" target="_blank">${properties.address}</a></td>
                </tr>
                <tr class="row g-0 mb-4">
                  <td class="text-shadow text-white text-break text-center d-flex flex-column align-items-center px-0 z-index-4">${
  properties.phone
}</td>
                </tr>
                <tr class="row g-0">
                  <td class="d-flex justify-content-center text-break fst-italic text-light p-0"><small class="opacity-50 z-index-4">最後更新時間： ${
  properties.updated_at
}</small></td>
                </tr>
              </tbody>
            </table>`,
          ),
        );
      });

      osmMap.addLayer(markers);
    },
  },
  computed: {
    filterData() {
      return this.rapidTestData.filter(
        (pharmacy) => !pharmacy.properties.address.search('高雄市三民區大昌二路２４８號'),
      );
    },
    redfliterData() {
      return this.rapidTestData.filter((pharmacy) => pharmacy.properties.count === 0);
    },
    yellowfliterData() {
      return this.rapidTestData.filter(
        (pharmacy) => pharmacy.properties.count > 0 && pharmacy.properties.count < 50,
      );
    },
    greenfliterData() {
      return this.rapidTestData.filter((pharmacy) => pharmacy.properties.count >= 50);
    },
  },
  created() {},
  mounted() {
    async function renderMap() {
      try {
        const position = await this.getCurrentPosition();
        this.location.latitude = position.coords.latitude;
        this.location.longitude = position.coords.longitude;
        this.rapidTestData = await this.getPharmacyData();
        this.initMap();
        this.addMapLayer();
        this.addMapMarkers();
      } catch (err) {
        console.log(err);
      }
    }
    renderMap.bind(this)();
  },
};
</script>

<style lang="scss">
@import './assets/scss/all';

#map {
  height: 100vh;
}
.marker-cluster-small {
  background-color: rgba(20, 134, 228, 0.2);
}
.marker-cluster-small div {
  background-color: rgba(20, 134, 228, 0.3);
}

.marker-cluster-medium {
  background-color: rgba(241, 149, 87, 0.2);
}
.marker-cluster-medium div {
  background-color: rgba(241, 149, 87, 0.3);
}

.marker-cluster-large {
  background-color: rgba(45, 3, 51, 0.2);
}
.marker-cluster-large div {
  background-color: rgba(45, 3, 51, 0.3);
}

.marker-cluster {
  background-clip: padding-box;
  border-radius: 20px;
}
.marker-cluster div {
  width: 2rem;
  height: 2rem;
  margin-left: 0.25rem;
  margin-top: 0.25rem;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-content: center;
}
.marker-cluster span {
  align-self: center;
}
.leaflet-popup-content-wrapper {
  padding: 8px 0px;
  position: relative;
  overflow: hidden;
  background: #3eb7ee;
  &::before {
    content: '';
    position: absolute;
    width: 200px;
    height: 200px;
    top: 221px;
    left: -88px;
    background: #03629c;
    border-radius: 50%;
    opacity: 0.3;
    z-index: 0;
  }
  &::after {
    content: '';
    position: absolute;
    width: 400px;
    height: 400px;
    top: 178px;
    left: -10px;
    background: #03629c;
    border-radius: 50%;
    opacity: 0.2;
    z-index: 1;
  }
}

.leaflet-popup-content {
  max-width: 220px;
  margin: 12px 20px 12px 20px;
}

.leaflet-popup-close-button {
  width: 44px !important;
  height: 44px !important;
  background: #ffffff !important;
  border-radius: 50%;
  color: #3eb7ee !important;
  transform: translate(10%, -10%);
  display: flex;
  justify-content: center;
  align-content: center;
  span {
    align-self: center;
  }
}
</style>
