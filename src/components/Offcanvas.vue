<template>
  <div
    ref="sidebarOffcanvas"
    class="offcanvas offcanvas-end bg-info"
    tabindex="-1"
    id="offcanvasRight"
    aria-labelledby="offcanvasRightLabel"
  >
    <div class="offcanvas-header bg-success flex-column">
      <div class="d-flex justify-content-between w-100 mb-4">
        <button
          type="button"
          class="btn text-white hvr-float p-0"
          data-bs-dismiss="offcanvas"
          aria-label="Close"
        >
          <i class="bi bi-arrow-right-circle-fill fs-3"></i></button>
        <h5
          class="text-white text-shadow-white d-flex align-items-center mb-0"
          id="offcanvasRightLabel"
        >快篩地圖</h5>
      </div>
      <div class="d-flex w-100 mb-4">
        <div class="w-50 me-8">
          <label
            for="citySelected"
            class="text-white"
          >城市</label>
          <select
            @change="area = '', road = ''"
            v-model="city"
            id="citySelected"
            class="form-select"
            aria-label="Default select example"
          >
            <option
              selected
              value="全臺"
            >全臺</option>
            <option
              v-for="city in mapData"
              :key="city?.CityName"
              :value="city?.CityName"
            >{{city?.CityName}}</option>
          </select>
        </div>
        <div class="w-50">
          <label
            for="citySelected"
            class="text-white"
          >鄉/市/鎮/區/鄉</label>
          <select
            @change="road = ''"
            v-model="area"
            id="citySelected"
            class="form-select"
            aria-label="Default select example"
          >
            <option
              value=""
              disabled
              selected
            >請選擇</option>
            <option
              v-for="area in cityData[0]?.AreaList"
              :key="area?.AreaName"
              :value="area?.AreaName"
            >{{area?.AreaName}}</option>
          </select>
        </div>
      </div>
      <div class="d-flex w-100 mb-4">
        <div class="w-100">
          <label
            for="citySelected"
            class="text-white"
          >路/巷/村/莊</label>
          <select
            v-model="road"
            id="citySelected"
            class="form-select"
            aria-label="Default select example"
          >
            <option
              value=""
              disabled
              selected
            >請選擇</option>
            <option
              v-for="road in areaData[0]?.RoadList"
              :key="road?.RoadName"
              :value="road?.RoadName"
            >{{road?.RoadName}}</option>
          </select>
        </div>
      </div>
      <div class="d-flex justify-content-around w-100 mb-4">
        <label
          for="quantitySelect-all"
          class="d-flex flex-column align-items-center
          hvr-float w-25"
        >
          <img
            class="mb-1"
            src="https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-black.png"
            alt="radioImage"
          >
          <p class="text-white mb-2">全選</p>
          <input
            id="quantitySelect-all"
            v-model="pharmacySelect"
            type="checkbox"
            name="quantitySelect"
            value="all"
          >
        </label>
        <label
          for="quantitySelect-none"
          class="d-flex flex-column align-items-center
          hvr-float w-25"
        >
          <img
            class="mb-1"
            src="https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-red.png"
            alt="radioImage"
          >
          <p class="text-white mb-2">不足</p>
          <input
            id="quantitySelect-none"
            v-model="pharmacySelect"
            type="checkbox"
            name="quantitySelect"
            value="none"
          >
        </label>
        <label
          for="quantitySelect-low"
          class="d-flex flex-column align-items-center
          hvr-float w-25"
        >
          <img
            class="mb-1"
            src="https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-yellow.png"
            alt="radioImage"
          >
          <p class="text-white mb-2">少量</p>
          <input
            id="quantitySelect-low"
            v-model="pharmacySelect"
            type="checkbox"
            name="quantitySelect"
            value="low"
          >
        </label>
        <label
          for="quantitySelect-full"
          class="d-flex flex-column align-items-center
          hvr-float w-25"
        >
          <img
            class="mb-1"
            src="https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-green.png"
            alt="radioImage"
          >
          <p class="text-white mb-2">充足</p>
          <input
            id="quantitySelect-full"
            v-model="pharmacySelect"
            type="checkbox"
            name="quantitySelect"
            value="full"
          >
        </label>
      </div>
      <button
        @click.stop="search"
        type="button"
        class="btn  btn-secondary w-100 text-white
      text-shadow-white fw-bold"
      >搜尋</button>
    </div>
    <div class="offcanvas-body overflow-y-scroll px-1">
      <ul class="list-unstyled">
        <li
        v-for="pharmacy in searchData.full" :key="pharmacy.properties.latitude"
        class="d-flex bg-success text-white shadow hvr-float px-4 py-2 mb-3">
          <div class="w-70">
            <div class="d-flex align-items-center mb-1">
              <h3 class="mb-0">{{pharmacy.properties.name}}</h3>
              <span class="badge bg-primary ms-1">
                {{ myLocation.distanceTo(
                  [pharmacy.properties.latitude, pharmacy.properties.longitude])
                  .toPrecision(2) / 1000}}
                  公里</span>
            </div>
            <div>
              <a class="text-decoration-none text-white mb-1">
                {{pharmacy.properties.address}}</a>
              <p class="mb-1">{{pharmacy.properties.phone}}</p>
            </div>
          </div>
          <p class="badge bg-full rounded-circle d-flex flex-column justify-content-center
          position-relative ms-1 mb-0 w-30">
            <span class="d-block fs-7 position-absolute top-5 start-50 translate-middle-x">
              羅氏:</span>
            <span class="d-block fs-1 text-shadow-white">
              {{pharmacy.properties.count}} <small class="fs-6">份</small>
            </span>
          </p>
        </li>
        <li class="d-flex bg-success text-white shadow hvr-float px-4 py-2 mb-1">
          <div>
            <div class="d-flex align-items-center mb-1">
              <h3 class="mb-0">我我藥局</h3>
              <span class="badge bg-primary ms-1">0.5 公里</span>
            </div>
            <div class="">
              <a class="text-decoration-none text-white mb-1">高雄市鳳山區光復路二段269之8號9樓</a>
              <p class="mb-1">077999125</p>
            </div>
          </div>
          <p class="badge bg-low rounded-circle d-flex flex-column ms-1 mb-0">
            <span class="d-block fs-7 mt-1 mb-3">羅氏:</span>
            <span class="d-block flex-grow-1 fs-1 text-white text-shadow-white">0.5 份</span>
          </p>
        </li>
        <li class="d-flex bg-success text-white shadow hvr-float px-4 py-2 mb-1">
          <div>
            <div class="d-flex align-items-center mb-1">
              <h3 class="mb-0">我我藥局</h3>
              <span class="badge bg-primary ms-1">0.5 公里</span>
            </div>
            <div class="">
              <a class="text-decoration-none text-white mb-1">高雄市鳳山區光復路二段269之8號9樓</a>
              <p class="mb-1">077999125</p>
            </div>
          </div>
          <p class="badge bg-none rounded-circle d-flex flex-column ms-1 mb-0">
            <span class="d-block fs-7 mt-1 mb-3">羅氏:</span>
            <span class="d-block flex-grow-1 fs-1 text-white text-shadow-white">0.5 份</span>
          </p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import Offcanvas from 'bootstrap/js/dist/offcanvas';
import mapData from '../../public/mapData/AllData.json';

export default {
  data() {
    return {
      offcanvas: null,
      pharmacySelect: ['all', 'none', 'low', 'full'],
      city: '全臺',
      area: '',
      road: '',
      mapData,
      cityData: [],
      areaData: [],
      filterData: [],
    };
  },
  props: {
    searchData: {
      type: Object,
      default() {
        return {};
      },
    },
    location: {
      type: Object,
      default() {
        return {};
      },
    },
  },
  methods: {
    openOffcanvas() {
      this.offcanvas.show();
    },
    closeOffcanvas() {
      this.offcanvas.hide();
    },
    search() {
      if (!this.pharmacySelect.length) {
        this.pharmacySelect = ['all', 'none', 'low', 'full'];
      }
      this.$emit('search', {
        pharmacySelect: this.pharmacySelect,
        locationText: this.locationText,
      });
    },
  },
  watch: {
    city() {
      if (this.city === '全臺') {
        this.cityData = [];
        this.area = '';
        return;
      }
      this.cityData = this.mapData.filter((city) => city.CityName === this.city);
    },
    area() {
      if (this.city === '全臺') {
        this.areaData = [];
        this.road = '';
        return;
      }
      this.areaData = this.cityData[0].AreaList.filter((area) => area.AreaName === this.area);
    },
    pharmacySelect(n, o) {
      if (n.includes('all')) {
        this.pharmacySelect.push('none', 'low', 'full');
        return;
      }
      if (!n.includes('all') && o.includes('all')) {
        this.pharmacySelect = [];
        return;
      }
      if (n.includes('none') && n.includes('low') && n.includes('full')) {
        this.pharmacySelect.push('all');
      }
    },
    searchData() {
      this.filterData = [...this.searchData];
    },
  },
  computed: {
    locationText() {
      return {
        city: this.city,
        area: this.area,
        road: this.road,
      };
    },
    myLocation() {
      return L.latLng(this.location.latitude, this.location.longitude);
    },
  },
  mounted() {
    this.offcanvas = new Offcanvas(this.$refs.sidebarOffcanvas, {
      backdrop: false,
    });
  },
};
</script>

<style scoped lang="scss">
</style>
