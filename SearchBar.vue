<template>
   <div :class="wrapperClass">
      <div :class="width" style="padding-right: 24px">
         <q-input
         @keydown.enter.prevent="prevent"
         v-model="modelValue"
         :dense="dense"
         :placeholder="title"
         debounce="500"
         style="font-size: 16px;"
         class="searchBar"
         type="search">
            <template v-slot:prepend>
              <q-icon name="search"/>
            </template>
            <template v-slot:append>
              <q-btn
              v-if="modelValue"
              @click="modelValue = null"
              icon="img:icons/clear-24px.svg"
              flat round dense/>
            </template>
         </q-input>

      </div>
      <div class="text-right search-right-bar" :class="restWidth">
      <slot>

      </slot>
      </div>
      <slot name="bottom"></slot>
   </div>
</template>
<script>
    export default {
        name: "SearchBar",
        props: {
            modelValue: String,
            title: { type: String, required: true },
            dense: { type: Boolean, default: true },
            fullwidth: { type: Boolean, default: false },
            width23: { type: Boolean, default: false },
            nested: { type: Boolean, required: false, default: false },
            customWidth: String,
        },
        emits: ['update:modelValue'],
        watch: {
            modelValue(){
                this.$emit('update:modelValue', this.modelValue);
            }
        },
       computed:{
           width(){
              if (this.customWidth) {
                 return this.customWidth;
              }

              if (this.fullwidth){
                 return 'col-12';
              }
              if (this.width23){
                 return 'col-8';
              }
              return 'col-md-4 col-sm-12';
           },
          restWidth(){
             if (this.fullwidth){
                return 'col-12';
             }
             if (this.width23){
                return 'col-4';
             }
             return 'col-md-8';

          },
          wrapperClass() {
             if (this.nested) {
                return this.customWidth || '';
             }

             return "row top_content_panel";
          }
       },
        methods: {
          prevent (event) {
            event.preventDefault();
            event.stopPropagation();
            return false;
          },
        }
    }
</script>

<style lang="scss">
   .searchBar {
      &:hover {
         background-color: $background-gray;

         & input {
            cursor: pointer;
         }
      }
   }
</style>