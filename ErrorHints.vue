<template>
   <q-banner v-if="shown" inline-actions class="text-white bg-red" dense>
      <ul>
         <li v-for="item in errorsList" :key="item">
            {{item}}
         </li>
      </ul>
   </q-banner>
</template>

<script>
    import {defineComponent} from 'vue';
    import Api from 'src/lib/api/admin-api';

    export default defineComponent({
        name: "ErrorHints",
        props: ['errors', 'name'],
        computed: {
            shown() {
                return this.errors && this.errors[this.name];
            },
            errorsList(){
                return this.errors[this.name];
            }
        },
        data() {
            return {
            };
        },
        methods: {

        }

    });
</script>

