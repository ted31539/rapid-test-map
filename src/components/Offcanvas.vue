<template>
  <div
    ref="sidebarOffcanvas"
    class="offcanvas offcanvas-end bg-info"
    tabindex="-1"
    id="offcanvasRight"
    aria-labelledby="offcanvasRightLabel"
  >
    <div class="offcanvas-header bg-success flex-column">
      <div class="d-flex justify-content-between w-100 mb-3">
        <div>
          <button
          @click="removeSidebarOpenBtnClass"
            type="button"
            class="btn text-white hvr-float p-0 me-2"
            data-bs-dismiss="offcanvas"
            aria-label="Close"
            data-bs-toggle="tooltip"
            data-bs-placement="top"
            title="收起搜尋欄"
          >
            <i class="bi bi-arrow-right-circle-fill fs-3"></i></button>
          <button
            @click.stop="toMyLocation"
            type="button"
            id="myLocationBtn"
            class="btn btn-primary shadow text-white"
            data-bs-toggle="tooltip"
            data-bs-placement="top"
            title="前往我的位置"
          >
            <i class="bi bi-pin-map-fill"></i>
            前往我的位置
          </button>
        </div>
        <h5
          class="text-white text-shadow-white d-flex align-items-center mb-0"
          id="offcanvasRightLabel"
        >快篩地圖</h5>
      </div>
      <div class="d-flex w-100 mb-3">
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
      <div class="d-flex w-100 mb-3">
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
      <div class="d-flex justify-content-around flex-wrap w-100 mb-3">
        <label class="text-white w-100 mb-2">快篩數量</label>
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
          <p class="text-white mb-1">全選</p>
          <p class="text-white mb-1">all</p>
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
          <p class="text-white mb-1">不足</p>
          <p class="text-white mb-1">0</p>
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
          <p class="text-white mb-1">少量</p>
          <p class="text-white mb-1"> &lt; 38</p>
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
          <p class="text-white mb-1">充足</p>
          <p class="text-white mb-1">&gt;=38</p>
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
    <div class="offcanvas-body px-1">
      <h3 class="text-center text-white text-shadow-white fw-bold mb-4">搜尋結果</h3>
      <ul class="list-unstyled">
        <li class="d-flex bg-success text-white shadow hvr-float px-4 py-2 mb-3">
          <div class="d-flex align-items-center justify-content-center w-100 mb-1">
            <p class="fs-5 mb-0">快篩充足的藥局：</p>
            <span class="badge bg-full fs-5 ms-1">{{ pharmacyNumData?.full }} 間</span>
          </div>
        </li>
        <li class="d-flex bg-success text-white shadow hvr-float px-4 py-2 mb-3">
          <div class="d-flex align-items-center justify-content-center w-100 mb-1">
            <p class="fs-5 mb-0">快篩少量的藥局：</p>
            <span class="badge bg-low fs-5 ms-1">{{ pharmacyNumData?.low }} 間</span>
          </div>
        </li>
        <li class="d-flex bg-success text-white shadow hvr-float px-4 py-2 mb-3">
          <div class="d-flex align-items-center justify-content-center w-100 mb-1">
            <p class="fs-5 mb-0">快篩不足的藥局：</p>
            <span class="badge bg-none fs-5 ms-1">{{ pharmacyNumData?.none }} 間</span>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
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
      pharmacyNumData: [],
    };
  },
  props: {
    pharmacyNum: {
      type: Object,
      default() {
        return {
          full: {},
          low: {},
          none: {},
        };
      },
    },
  },
  emits: ['search', 'to-my-location'],
  methods: {
    openOffcanvas() {
      this.offcanvas.show();
    },
    closeOffcanvas() {
      this.offcanvas.hide();
    },
    removeSidebarOpenBtnClass() {
      document.querySelector('#sidebarOpenBtn').classList.remove('roll-away');
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
    toMyLocation() {
      this.$emit('to-my-location');
    },
  },
  watch: {
    city() {
      if (this.city === '全臺') {
        this.cityData = [];
        this.area = '';
        return;
      }
      this.cityData = mapData.filter((city) => city.CityName === this.city);
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
    pharmacyNum() {
      this.pharmacyNumData = { ...this.pharmacyNum };
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
