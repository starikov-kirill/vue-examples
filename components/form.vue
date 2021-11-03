<template>
  <form class="shedule" @submit.prevent="onSubmit">
    <div class="title">Укажите отделение, тему и дату записи на прием:</div>
    <div class="office">
      <Select validation-model="office"
              label="Отделение"
              :required="true"
              @update="onOfficeUpdate"
              :options="$store.getters.officesForSelect">
      </Select>
      <Button class="input"
              @click="openModal('map')">
        <i class="icon-map"></i>
      </Button>
    </div>

    <Select validation-model="theme"
            label="Тема"
            :required="true"
            @update="onThemeUpdate"
            :disabled="!formData.office"
            :options="$store.getters.themesForSelect">
    </Select>

    <Datetime label="Дата и время записи"
              validation-model="slot"
              :disabled="!formData.office || !formData.theme"
              :required="true"
              :timeZone="officeTimeZone"
              @click="openModal('calendar')" />
    <button hidden></button>
    <Validation-message v-if="$v.$invalid" />
  </form>
</template>

<script>
import API from '@/util/api';
import { cloneObject } from '@/util/helpers';
import { EventBus } from '@/plugins/event-bus';

import validation from './validation';

import Select from '#c/uikit/Select.vue';
import Button from '#c/uikit/Button.vue';
import Datetime from '#c/uikit/Datetime.vue';
import ValidationMessage from '#c/layout/ValidationMessage.vue';

export default {
  name: 'Shedule',

  mixins: [validation],

  data() {
    return {
      formData: {
        reservation: null,
        office: '',
        theme: '',
        slot: '',
      },
    };
  },

  created() {
    if (this.data) {
      this.formData = cloneObject(this.data);
    }
    this.$store.dispatch('getOffices', this.regionData);
  },

  mounted() {
    EventBus.$on('modal-select-office', this.onSelectOffice);
    EventBus.$on('modal-select-date', this.onSelectDate);
  },

  beforeDestroy(){
    EventBus.off('modal-select-office', this.onSelectOffice);
    EventBus.off('modal-select-date', this.onSelectDate);
  },

  computed: {
    officeTimeZone() {
      if (!this.formData.office) return false;
      const office = this.$store.state.offices.find((office) => office.id === this.formData.office);

      return office?.timeZone ? office.timeZone : false;
    },
  },

  props: {
    data: Object,
    regionData: Object,
    isAccountKnown: Boolean,
  },

  methods: {
    onSelectOffice(office) {
      this.formData.office = office.id;
      this.$store.dispatch('getThemes', { id: office.id });
    },
    onSelectDate(slot) {
      this.formData.slot = slot;
    },
    onThemeUpdate(val) {
      if (val !== this.formData.theme) this.resetSlot();
    },
    onOfficeUpdate(val) {
      if (val === this.formData.office) return;

      this.resetSlot();
      this.$store.dispatch('getThemes', { id: val });
    },
    resetSlot() {
      this.formData.slot = null;
      this.$store.commit('slots', []);
    },
    openModal(val) {
      if (typeof ymaps === 'undefined' || ymaps.Map === undefined) return;

      if (val !== 'calendar') {
        this.$store.commit('visibleModalName', val);
        return;
      }

      if (this.formData.office && this.formData.theme) {
        EventBus.$emit('office-theme-date', this.formData);
        this.$store.commit('visibleModalName', val);
      }
    },
    onFormData() {
      if (this.formData.reservation?.slot?.id === this.formData.slot.id) {
        this.$emit('submitSuccess', this.formData);
        return;
      }

      this.$http.post(
        API.getEndpoint(`slots/${this.formData.slot.id}/lock`),
      ).then((response) => {
        if (!response.data.reservation) {
          return;
        }

        this.formData.reservation = response.data.reservation;
        this.$emit('submitSuccess', this.formData);
      }).catch((e) => {
        console.error(e);
      });
    }
  },

  components: {
    Select,
    Button,
    Datetime,
    ValidationMessage,
  },
};
</script>

<style scoped lang="stylus">
@import '~#a/style/config'
.validation-message
  margin-top: 30px
.shedule
  max-width: 335px
  margin: 0 auto

.office
  clearfix()
  .input-container, button
    float: left
  .input-container
    width: calc(100% - 56px)
    margin-right: 6px

.icon-map
  font-size: 20px
  top: 3px
  position: relative
  left: 1px
</style>
