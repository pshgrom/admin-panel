<template>
  <q-select input-debounce="500" :outlined="outlined" :label="label" type="search" v-model="organization"
            dense
            use-input :options="options" map-options
            @filter-abort="abortFilterFn" @filter="search" clearable
            :rules="[val => validate(val)]" :readonly="readonly"
            lazy-rules
  >
    <template v-slot:option="scope">
      <q-item v-bind="scope.itemProps" dense>
        <q-item-section avatar>
          {{ scope.opt.value }}
        </q-item-section>
        <q-item-section>
          <q-item-label v-html="scope.opt.label"/>
        </q-item-section>
      </q-item>
    </template>
  </q-select>
</template>
<script>
import {defineComponent} from 'vue';
import Api from 'src/lib/pos/api';

export default defineComponent({
  name: "OrganizationSelect",
  props: {
    label: {
      type: String,
      default: 'Название организации'
    },
    modelValue: {
      type: Number,
      default: false
    },
    outlined: {
      type: Boolean,
      default: false
    },
    required: {
      type: Boolean,
      default: false
    },
    readonly: {
      type: Boolean,
      default: false
    }
  },
  emits: ['update:modelValue'],
  watch: {
    modelValue() {
      this.modelChanged();
    },
    organization() {
      const value = this.organization ? this.organization.value : 0;
      this.$emit('update:modelValue', value);
    }
  },
  data() {
    return {
      options: [],
      organization: null
    };
  },
  mounted() {
    this.modelChanged();
  },
  methods: {
    validate(val) {
      console.log('validating', val);
      if (!this.required) return true;
      return (val !== 0 && val != null) || 'Заполните организацию';
    },
    abortFilterFn() {

    },
    addFilterVal() {

    },
    modelChanged() {
      if (this.organization && this.organization.id === this.modelValue) return;
      Api.organization.search(this.modelValue).then((list) => {
        this.organization = list.find(item => item.value == this.modelValue);
      });
    },
    search(val, update, abort) {
      // if (val.trim() === '') {
      //    update(() => {
      //       this.options = [];
      //    });
      //    return;
      // }
      console.log('searching', val);
      Api.organization.search(val).then((list) => {
        update(() => {
          console.log('setting options to', list);
          this.options = list;
        });
      });

    },
  }

});
</script>