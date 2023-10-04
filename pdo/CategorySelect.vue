<template>
   <q-select v-model="value" label="Категория"
             :outlined="outlined"
             :rules="[val => validate(val)]" :readonly="readonly"
             option-value="id"
             option-label="name"
             use-input
             clearable
             @filter="categoriesFilter"
             :options="categoryOptions" map-options emit-value dense>
      <template v-slot:option="scope">
         <q-item v-bind="scope.itemProps" dense>
            <q-item-section avatar v-if="techAdmin">
               {{ scope.opt.id }}
            </q-item-section>
            <q-item-section>
               <q-item-label v-html="scope.opt.name"/>
            </q-item-section>
         </q-item>
      </template>
   </q-select>
</template>
<script>
import {defineComponent} from 'vue';
import Api from 'src/lib/pdo/api';
import globalState from 'src/lib/state';

export default defineComponent({
   name: "CategorySelect",
   props: {
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
      readonly:{
         type: Boolean,
         default: false
      }
   },
   emits: ['update:modelValue'],
   watch: {
      modelValue() {
         this.value = this.modelValue;
      },
      value() {
         this.$emit('update:modelValue', this.value);
      }
   },
   data() {
      return {
         value: null,
         categories: [],
         categoryOptions: null
      };
   },
   created() {
      Api.categories.list({page:1, rowsPerPage:1000, sortBy:'name', descending:false}).then(data=>{
         this.categories = data.list;
         this.categoryOptions = data.list;
      });
   },
   computed:{
      techAdmin(){
         return globalState.hiddenMenu;
      }
   },
   methods: {
      validate(val) {
         if (!this.required) return true;
         return (val !== 0 && val != null) || 'Заполните категорию';
      },
      categoriesFilter(val, update, abort) {
         update(() => {
            const needle = val.toLowerCase();
            this.categoryOptions = this.categories.filter(v => v.name.toLowerCase().indexOf(needle) > -1)
         })
      }
   }

});
</script>