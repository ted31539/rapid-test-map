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
            v-model="quantitySelect"
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
            v-model="quantitySelect"
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
            v-model="quantitySelect"
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
            v-model="quantitySelect"
            type="checkbox"
            name="quantitySelect"
            value="full"
          >
        </label>
      </div>
      <button
      @click.stop="search"
      type="button" class="btn  btn-secondary w-100 text-white
      text-shadow-white fw-bold">搜尋</button>
    </div>
    <div class="offcanvas-body">

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
      quantitySelect: ['all', 'none', 'low', 'full'],
      city: '全臺',
      area: '',
      road: '',
      mapData,
      cityData: [],
      areaData: [],
    };
  },
  methods: {
    openOffcanvas() {
      this.offcanvas.show();
    },
    closeOffcanvas() {
      this.offcanvas.hide();
    },
    search() {
      this.$emit('search', { quantitySelect: this.quantitySelect, locationText: this.locationText });
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
    quantitySelect(n, o) {
      if (n.includes('all')) {
        this.quantitySelect.push('none', 'low', 'full');
        return;
      }
      if (!n.includes('all') && o.includes('all')) {
        this.quantitySelect = [];
        return;
      }
      if (n.includes('none') && n.includes('low') && n.includes('full')) {
        this.quantitySelect.push('all');
      }
    },
  },
  computed: {
    locationText() {
      return this.city + this.area + this.road;
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
