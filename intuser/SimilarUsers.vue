<template>
    <q-markup-table v-if="users">

        <tbody>

        <tr v-for="user in users" :key="user.id">
            <td class="text-left">ID: {{user.id}}</td>
            <td class="text-left">{{user.name}}</td>
            <td class="text-left">({{user.nickname}})</td>
        </tr>
        </tbody>
    </q-markup-table>
</template>
<style>


</style>
<script>
import {defineComponent} from 'vue';

export default defineComponent({
    name: "SimilarUsers",
    emits: [],
    props: {
        users: {
            type: Object,
            default: null
        },
    },
    components: {},
    computed: {},
    watch: {},
    data() {
        return {
        };
    },
    mounted() {

    },
    methods: {

    }

});
</script>