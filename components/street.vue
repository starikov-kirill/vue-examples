<template>
  <SearchSelect :required="true"
          :autoBlur="true"
          label="Улица"
          @update="onUpdate"
          v-model="value"
          @searchQuery="onSearchQuery"
          :validationMessages="validationMessages"
          :validationState="validationState"
          :options="options" />
</template>

<script>
import _ from 'lodash'
import API from '@/util/api';
import { listForSelect, cloneObject } from '@/util/helpers';
import SearchSelect from '#c/uikit/StepByStepSelect.vue';

export default {
  name: 'Street',

  emits: ['update:modelValue'],

  model: {
    prop: 'modelValue',
    event: 'update:modelValue',
  },

  data() {
    return {
      value: null,
      options: [],
      query: '',
      seatchTimeout: null,
    };
  },

  props: {
    parent: Object,
    validationMessages: Object,
    validationState: Object,
  },

  watch: {
    parent() {
      this.reset();
      if (this.value?.label) this.loadStreets(this.value.label);
    },
  },

  methods: {
    onSearchQuery: _.debounce(function(query){
      this.query = query;
      this.search(query);
    }, 500),

    onUpdate(value) {
      this.value = value;
      this.$emit('update:modelValue', value);
    },

    reset(label) {
      let value = cloneObject(this.value);

      if (typeof label === 'undefined') {
        value?.id && delete value.id;
        this.$emit('update:modelValue', value);
        return;
      }

      if (label.trim()) {
        value === null ? value = { label } : value.label = label;
      } else {
        value = null;
      }

      this.$emit('update:modelValue', value);
    },

    search(query) {
      this.reset(query);
      this.loadStreets(query);
    },
    loadStreets(query) {
      if (query.trim().length < 3) return;
      if (!this.parent?.id) return;
      this.$http.post(API.getEndpoint('street'), {
        id_area_parent: this.parent.id,
        nm_street: query,
      }).then((response) => {
        this.options = listForSelect(response.data.data, 'key');
      });
    },
  },

  components: {
    SearchSelect,
  },
};
</script>
