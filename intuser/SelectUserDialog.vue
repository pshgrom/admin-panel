<template>
    <q-dialog v-model="show">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout"
                  style="min-width: 1000px;width: 1000px;max-height: 300px">
            <q-header bordered>
                <q-toolbar>
                    <q-toolbar-title>{{ title }}</q-toolbar-title>
                    <q-btn flat v-close-popup round dense icon="close" @click="cancelEdit"/>
                </q-toolbar>
            </q-header>

            <q-footer bordered>
                <custom-button title="Отмена" type="light" @click="cancelEdit"/>
                <custom-button :title="'Выбрать'" type="purple" @click="save"/>
            </q-footer>


            <q-page-container>
                <q-page padding>
                    <q-form ref="form" @submit="save" class="user-form" dense>

                        <div class="row form-row">
                            <div class="col-12">
                                <q-select input-debounce="500" :outlined="true" label="Выберите пользователя"
                                          type="search" v-model="user"
                                          dense
                                          use-input :options="options" map-options
                                          @filter-abort="abortFilterFn" @filter="search" clearable>
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
                            </div>
                        </div>

                    </q-form>
                </q-page>
            </q-page-container>
        </q-layout>
    </q-dialog>
</template>
<style>


</style>
<script>
import {defineComponent} from 'vue';
import Helpers from 'src/lib/api/helpers';
import Api from 'src/lib/auth/api';
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
    name: "SelectUserDialog",
    emits: ['confirmed', 'confirmed'],
    props: {
        show: {
            type: Boolean,
            default: 0
        },
        title: {
            type: String,
            default: 0
        },
    },
    components: {CustomButton},
    computed: {},
    watch: {},
    data() {
        return {
            user: null,
            options: []
        };
    },
    mounted() {

    },
    methods: {
        cancelEdit() {
            this.$emit('cancel');
        },
        save(stay) {
            this.$emit('confirmed', {id: this.user.value??0});
        },
        abortFilterFn() {

        },
        search(val, update, abort) {
            Api.intu.userSearch(val).then((list) => {
                update(() => {
                    this.options = list;
                });
            });

        },
        ...Helpers
    }

});
</script>