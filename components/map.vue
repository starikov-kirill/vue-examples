<template>
  <Modal id="map-modal"
         :title="$t('components_Modal_Map.select_office')"
         :on-hide="onHide"
         :visible="$store.state.visibleModalName === 'map'">
    <div class="wrapper"
         :class="{sidebarVisible: office}">
      <div id="map"></div>
      <div v-if="office" class="sidebar">
        <div class="title">{{office.name}}</div>
        <div class="address">
          <p>{{ $t("components_Modal_Map.address") }}:</p>
          <p>{{office.street}}</p>
        </div>
        <div class="shedule">
          <p>{{ $t("components_Modal_Map.shedule") }}:</p>
          <p>{{office.workSchedule}}</p>
        </div>
        <Button @click="selectOffice"
                class="select">
          {{ $t("select") }}
        </Button>
      </div>
    </div>
  </Modal>
</template>

<script>
import { EventBus } from '@/plugins/event-bus';

import Modal from '#c/uikit/Modal.vue';
import Button from '#c/uikit/Button.vue';

let yandexMap = null;
const DEF_COORDINATES = [55.75, 37.6];

export default {
  name: 'modal-map',

  data() {
    return {
      userCoodinates: DEF_COORDINATES,
      office: null,
      isLocationLoaded: false,
    };
  },

  beforeDestroy() {
    yandexMap?.destroy();
  },

  watch: {
    '$store.state.visibleModalName': function (val) {
      if (val !== 'map') return;
      if (!this.isLocationLoaded) this.loadLocation();
      this.initMap();
      this.addGeoObjects();
    },
  },

  methods: {
    addGeoObjects() {
      for (let i = 0; i < this.$store.state.offices.length; i += 1) {
        const office = this.$store.state.offices[i];

        const myPlacemark = yandexMap.geoObjects.add(new ymaps.Placemark(
          [office.lat, office.lon],
          null,
          {
            preset: 'islands#circleIcon',
            iconColor: '#0079C2',
          },
        ));

        myPlacemark.events.add('click', () => {
          this.office = office;
        });
      }
    },
    selectOffice() {
      EventBus.$emit('modal-select-office', this.office);
      this.onHide();
    },
    onHide() {
      this.office = null;
      this.$store.commit('hideModal');
    },
    loadLocation() {
      this.isLocationLoaded = true;
      const location = ymaps.geolocation.get();
      location.then((result) => {
        const userCoodinates = result.geoObjects.get(0).geometry.getCoordinates();
        this.userCoodinates = userCoodinates;
        if (!yandexMap) return;
        yandexMap.setCenter(userCoodinates);
      }).catch((e) => {
        console.error(e);
      });
    },
    initMap() {
      yandexMap = new ymaps.Map('map', {
        center: this.userCoodinates,
        zoom: 12,
        controls: ['zoomControl'],
      });
    },
  },

  components: {
    Modal,
    Button,
  },
};
</script>
