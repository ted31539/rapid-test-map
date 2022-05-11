<template>
  <div class="container-fluid">
    <div
      id="map"
      class="position-relative"
    >
      <div class="d-flex flex-column position-absolute bottom-3 end-3 z-index-999">
        <button
          type="button"
          @click.stop="toMyLocation"
          id="myLocationBtn"
          class="btn btn-primary rounded-circle shadow hvr-pulse-grow mb-4"
          data-bs-toggle="tooltip"
          data-bs-placement="top"
          title="前往我的位置"
        >
          <i class="bi bi-pin-map-fill fs-3 text-danger hvr-pulse-grow"></i>
        </button>
        <button
          type="button"
          @click.stop="openSidebar"
          id="sidebarOpenBtn"
          class="btn btn-primary rounded-circle shadow hvr-pulse-grow a"
          data-bs-toggle="tooltip"
          data-bs-placement="top"
          title="查詢"
        >
          <i class="bi bi-grid-3x3-gap fs-3 text-danger"></i>
        </button>
      </div>
      <Offcanvas
      class=""
        @click.stop=""
        @search="search"
        @to-my-location="toMyLocation"
        :pharmacyNum="pharmacyNum"
        ref="sidebar"
      />
      <MessageModal
        @click.stop=""
        :message="message"
        ref="modal"
      />
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet.markercluster/dist/leaflet.markercluster';
import 'leaflet.markercluster.layersupport';
import Offcanvas from './components/Offcanvas.vue';
import MessageModal from './components/MessageModal.vue';

let osmMap = {};
let markers = null;
let layerControl = null;
let fullMarkerAry = [];
let lowMarkerAry = [];
let noneMarkerAry = [];
let fullLayerGroup = null;
let lowLayerGroup = null;
let noneLayerGroup = null;
// const full = [];
// const low = [];

const greenIcon = new L.Icon({
  iconUrl:
    'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-green.png',
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
const redIcon = new L.Icon({
  iconUrl:
    'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-red.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});
const blueIcon = new L.Icon({
  iconUrl:
    'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-blue.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
  iconSize: [37.5, 61.5],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});

export default {
  name: 'App',
  components: { Offcanvas, MessageModal },
  data() {
    return {
      rapidTestData: [],
      location: {
        latitude: '',
        longitude: '',
      },
      myLocationMarker: null,
      message: [],
      searchData: {
        full: [],
        low: [],
        none: [],
      },
      pharmacyNum: {},
    };
  },
  methods: {
    search(searchContent) {
      this.searchData = {
        full: [],
        low: [],
        none: [],
      };
      const { locationText, pharmacySelect } = searchContent;
      // console.log(locationText, pharmacySelect);

      let tempSearchData = [];
      let outcomeCaculte = [];
      let pharmacyLocation = [];

      if (locationText.city === '全臺') {
        if (pharmacySelect.includes('full')) {
          this.searchData.full = this.fullfliterData;
          const text = `快篩充足的藥局共 ${this.searchData.full.length} 家`;
          outcomeCaculte.push(text);
        }
        if (pharmacySelect.includes('low')) {
          this.searchData.low = this.lowfliterData;
          const text = `快篩少量的藥局共 ${this.searchData.low.length} 家`;
          outcomeCaculte.push(text);
        }
        if (pharmacySelect.includes('none')) {
          this.searchData.none = this.nonefliterData;
          const text = `快篩不足的藥局共 ${this.searchData.none.length} 家`;
          outcomeCaculte.push(text);
        }
        // console.log('全臺', this.searchData);
      }

      if (locationText.city !== '全臺') {
        tempSearchData = this.rapidTestData.filter(
          (pharmacy) => pharmacy.properties.address.includes(locationText.city),
        );
        // console.log('城市', tempSearchData);
        if (locationText.area.length) {
          tempSearchData = tempSearchData.filter(
            (pharmacy) => pharmacy.properties.address.includes(locationText.area),
          );
          // console.log('鄉鎮市區', tempSearchData);
          if (locationText.road.length) {
            tempSearchData = tempSearchData.filter(
              (pharmacy) => pharmacy.properties.address.includes(locationText.road),
            );
            // console.log('路巷村莊', tempSearchData);
          }
        }

        // console.log('分配資料了');
        if (pharmacySelect.includes('full')) {
          this.searchData.full = tempSearchData.filter(
            (pharmacy) => pharmacy.properties.count >= 38,
          );
          const text = `快篩充足的藥局共 ${this.searchData.full.length} 家`;
          outcomeCaculte.push(text);
        }
        if (pharmacySelect.includes('low')) {
          this.searchData.low = tempSearchData.filter(
            (pharmacy) => pharmacy.properties.count > 0 && pharmacy.properties.count < 38,
          );
          const text = `快篩少量的藥局共 ${this.searchData.low.length} 家`;
          outcomeCaculte.push(text);
        }
        if (pharmacySelect.includes('none')) {
          this.searchData.none = tempSearchData.filter(
            (pharmacy) => pharmacy.properties.count === 0,
          );
          const text = `快篩不足的藥局共 ${this.searchData.none.length} 家`;
          outcomeCaculte.push(text);
        }
      }

      // console.log(pharmacyLocation);

      // console.log('分類完成', this.searchData);
      if (
        !this.searchData.full.length
        && !this.searchData.low.length
        && !this.searchData.none.length
      ) {
        outcomeCaculte = ['拍謝，沒有符合條件的藥局!!'];
        this.message = outcomeCaculte;
        this.$refs.modal.openModal();
      } else {
        if (pharmacySelect.includes('none')) {
          if (this.searchData.none.length) {
            pharmacyLocation = [];
            pharmacyLocation = [
              this.searchData.none[0].geometry.coordinates[1],
              this.searchData.none[0].geometry.coordinates[0],
            ];
            // console.log('找到沒有塊篩的藥局位置', pharmacyLocation);
          }
        }
        if (pharmacySelect.includes('low')) {
          if (this.searchData.low.length) {
            pharmacyLocation = [];
            pharmacyLocation = [
              this.searchData.low[0].geometry.coordinates[1],
              this.searchData.low[0].geometry.coordinates[0],
            ];
          }
        }
        if (pharmacySelect.includes('full')) {
          if (this.searchData.full.length) {
            pharmacyLocation = [];
            pharmacyLocation = [
              this.searchData.full[0].geometry.coordinates[1],
              this.searchData.full[0].geometry.coordinates[0],
            ];
          }
        }
        // console.log(pharmacyLocation);
        // console.log(outcomeCaculte);
        this.message = outcomeCaculte;
        this.pharmacyNum = {
          full: this.searchData.full.length,
          low: this.searchData.low.length,
          none: this.searchData.none.length,
        };
        this.addMapMarkers(this.searchData);
        this.panToLocation(pharmacyLocation);
        this.$refs.modal.openModal();
      }
    },
    getPharmacyData() {
      const url = 'https://raw.githubusercontent.com/SiongSng/Rapid-Antigen-Test-Taiwan-Map/data/data/antigen_open_street_map.json';
      return this.$http
        .get(url)
        .then((res) => res.data.features)
        .catch(() => {
          this.message = ['抱歉，發生錯誤，請重新整理'];
          this.$refs.modal.openModal();
        });
    },
    sortPharmacyData() {
      this.searchData.full = this.fullfliterData;
      this.searchData.low = this.lowfliterData;
      this.searchData.none = this.nonefliterData;
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
      L.control.scale().addTo(osmMap);
    },
    getCurrentPosition() {
      let res;
      if ('geolocation' in navigator) {
        res = new Promise((resolve, reject) => {
          navigator.geolocation.getCurrentPosition(
            (position) => resolve(position),
            (error) => reject(error),
          );
        });
      } else {
        this.message = ['抱歉，無法取得您的所在，請檢查您的裝置'];
        this.$refs.modal.openModal();
      }
      return res;
      /* geolocation IS NOT available */
    },
    addMapMarkers(
      filterAry = {
        full: this.fullfliterData,
        low: this.lowfliterData,
        none: this.nonefliterData,
      },
    ) {
      if (markers) {
        markers.checkOut(fullLayerGroup);
        markers.checkOut(lowLayerGroup);
        markers.checkOut(noneLayerGroup);
        osmMap.removeLayer(fullLayerGroup);
        osmMap.removeLayer(lowLayerGroup);
        osmMap.removeLayer(noneLayerGroup);
        osmMap.removeControl(layerControl);
        layerControl = null;
        markers = null;
        fullLayerGroup = null;
        lowLayerGroup = null;
        noneLayerGroup = null;
        fullMarkerAry = [];
        lowMarkerAry = [];
        noneMarkerAry = [];
      }

      markers = L.markerClusterGroup.layerSupport();
      layerControl = L.control.layers().addTo(osmMap);
      const myLocation = L.latLng(this.location.latitude, this.location.longitude);

      this.myLocationMarker = L.marker([this.location.latitude, this.location.longitude], {
        icon: blueIcon,
      }).addTo(osmMap).bindPopup(`
      <div><p class="fs-5 text-danger fw-bold text-shadow-white pt-4"><i class="bi bi-emoji-laughing-fill me-1"></i>我的位置</p></div>`);

      function addFullMarkers() {
        filterAry.full.forEach((pharmacy) => {
          const { geometry, properties } = pharmacy;
          const marker = L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
            icon: greenIcon,
          });
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
          );
          fullMarkerAry.push(marker);
        });
        fullLayerGroup = L.layerGroup(fullMarkerAry);
        markers.checkIn(fullLayerGroup);
        fullLayerGroup.addTo(osmMap);
        layerControl.addOverlay(
          fullLayerGroup,
          '<p class="text-full text-shadow fw-bold fs-5 mb-0">快篩充足藥局<p>',
        );
      }
      function addLowMarkers() {
        filterAry.low.forEach((pharmacy) => {
          const { geometry, properties } = pharmacy;
          const marker = L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
            icon: yellowIcon,
          });

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
          );
          lowMarkerAry.push(marker);
        });
        lowLayerGroup = L.layerGroup(lowMarkerAry);
        markers.checkIn(lowLayerGroup);
        lowLayerGroup.addTo(osmMap);
        layerControl.addOverlay(
          lowLayerGroup,
          '<p class="text-low text-shadow-white fw-bold fs-5 mb-0">快篩少量藥局<p>',
        );
      }
      function addNoneMarkers() {
        filterAry.none.forEach((pharmacy) => {
          const { geometry, properties } = pharmacy;
          const marker = L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
            icon: redIcon,
          });

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
          );
          noneMarkerAry.push(marker);
        });
        noneLayerGroup = L.layerGroup(noneMarkerAry);
        markers.checkIn(noneLayerGroup);
        noneLayerGroup.addTo(osmMap);
        layerControl.addOverlay(
          noneLayerGroup,
          '<p class="text-none text-shadow fw-bold fs-5 mb-0">快篩不足藥局<p>',
        );
      }
      markers.addTo(osmMap);
      addNoneMarkers.bind(this)();
      addLowMarkers.bind(this)();
      addFullMarkers.bind(this)();
    },
    toMyLocation() {
      this.getCurrentPosition().then((position) => {
        this.location.latitude = position.coords.latitude;
        this.location.longitude = position.coords.longitude;
        osmMap.setView([this.location.latitude, this.location.longitude], 16);
        this.myLocationMarker.openPopup();
      });
    },
    panToLocation(location) {
      osmMap.setView(location, 16);
    },
    openSidebar() {
      document.querySelector('#sidebarOpenBtn').classList.add('roll-away');
      setTimeout(() => {
        this.$refs.sidebar.openOffcanvas();
      }, 400);
    },
    welcomeMessage() {
      this.message = ['歡迎使用快篩地圖',
        '右上角圖層圖示可選擇要顯示的藥局',
        '右下角搖桿圖示可前往您的位置',
        '最右下角方塊圖示可開啟尋列'];
      this.$refs.modal.openModal();
    },
  },
  computed: {
    nonefliterData() {
      return this.rapidTestData.filter((pharmacy) => pharmacy.properties.count === 0);
    },
    lowfliterData() {
      return this.rapidTestData.filter(
        (pharmacy) => pharmacy.properties.count > 0 && pharmacy.properties.count < 38,
      );
    },
    fullfliterData() {
      return this.rapidTestData.filter((pharmacy) => pharmacy.properties.count >= 38);
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
        this.sortPharmacyData();
        this.initMap();
        this.addMapLayer();
        this.addMapMarkers();
        this.welcomeMessage();
      } catch (err) {
        this.message = ['抱歉，發生錯誤，請重新整理'];
        this.$refs.modal.openModal();
      }
    }
    renderMap.bind(this)();
  },
};
</script>

<style lang="scss">
@import '../node_modules/leaflet.markercluster/dist/MarkerCluster.Default.css';
@import '../node_modules/leaflet.markercluster/dist/MarkerCluster.css';
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

/* layers control */

.leaflet-control-layers {
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.4);
  background: #108ECE;
  border-radius: 5px;
}
.leaflet-control-layers-toggle {
  background-image: url(../public/image/stack.svg);
  width: 49px;
  height: 49px;
}
.leaflet-touch .leaflet-control-layers-toggle {
  width: 49px;
  height: 49px;
}
.leaflet-control-layers-expanded {
  padding: 16px;
  color: #333;
  background: #fff;
}
.leaflet-control-layers label {
  display: block;
  font-size: 13px;
  font-size: 1.08333em;
  span {
    display: flex;
  }
  margin-bottom: 8px;
}
</style>
