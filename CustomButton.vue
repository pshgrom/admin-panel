<template>
  <q-btn :label="title" :class="className">

  </q-btn>
</template>

<script>
  import { defineComponent } from 'vue';

  export default defineComponent({
    name: 'CustomButton',
    props: ['type', 'title', 'class'],
    computed: {
      className() {
        let className = 'customButton';
        if (this.type) {
          className = `${className} customButton-${this.type}`;
        }

        if (this.class) {
          className = `${className} ${this.class}`;
        }

        return className;
      }
    },

  })
</script>

<style lang='scss'>
    .customButton {
        letter-spacing: 1.35px;
        
        &-purple {
            background-color: $primary;
            color: #fff;
            border: 1px solid $primary;

            &:hover {
                background-color: $dark;
                border: 1px solid $dark;
            }

            &:disabled {
                color: $disabled-text-gray;
                background-color: $background-gray;
            }
        }

        &-red {
            background-color: #fff;
            color: $red;
            border: 1px solid $red;

            &:hover {
                background-color: $background-gray;
            }

            &:disabled {
                background-color: #fff;
                color: $disabled-text-red;
                border-color: $disabled-text-red;
            }
        }

        &-green {
            background-color: $gorod-ligth-green3;
            color: #fff;
            border: 1px solid $gorod-ligth-green3;

             &:hover {
                background-color: $green;
                border: 1px solid $green;
            }

            &:disabled {
                color: $disabled-text-gray;
                background-color: $background-gray;
            }
        }

        &-light {
            background-color: #fff;
            color: $primary;
            border: 1px solid $borders-gray;

            &:hover {
                background-color: $background-gray;
                color: $dark;
                border: 1px solid $background-gray;
            }

            &:disabled {
                color: $disabled-text-gray;
                background-color: $background-gray;
            }
        }
    }
</style>