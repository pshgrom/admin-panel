<template>
    <q-dialog v-model="show" @escape-key="cancelEdit">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout" style="min-width: 1050px;width: 1050px;max-height: 450px">
            <q-header bordered>
                <q-toolbar>
                    <q-toolbar-title>Поиск пользователей</q-toolbar-title>
                    <q-btn flat v-close-popup round dense icon="close" @click="cancelEdit"/>
                </q-toolbar>
            </q-header>

            <q-footer bordered>
                <custom-button title="Сбросить всё" outline  type="light" @click="cleanVals"/>
                <custom-button title="Отменить" outline color="red" type="light" @click="cancelEdit"/>
                <custom-button title="Найти" type="purple" @click="save"/>
            </q-footer>


            <q-page-container>
                <q-page padding>
                    <q-form ref="form" @submit="save" class="user-form" dense>

                        <div class="row form-row" v-if="withSsoid">
                            <div class="col-6">
                                <q-input label="SSOID" dense v-model="data.ssoid" clearable></q-input>
                            </div>
                            <div class="col-6">
                                <q-input label="E-Mail" dense v-model="data.email" clearable></q-input>
                            </div>
                        </div>
                        <div class="row form-row" v-if="!withSsoid">
                            <div class="col-12">
                                <q-input label="E-Mail" dense v-model="data.email" clearable></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-4">
                                <q-input label="Фамилия" dense v-model="data.last_name" clearable></q-input>
                            </div>
                            <div class="col-4">
                                <q-input label="Имя" dense v-model="data.first_name" clearable></q-input>
                            </div>
                            <div class="col-4">
                                <q-input label="Отчество" dense v-model="data.middle_name" clearable></q-input>
                            </div>
                        </div>

                        <div class="row form-row">
                            <div class="col-6">
                                <q-input label="Дата регистрации с" v-model="data.regdate.from" clearable readonly>
                                    <template v-slot:append>
                                        <q-icon name="cancel" v-if="data.regdate.from!=null" @click="data.regdate.from=null" class="cursor-pointer" ></q-icon>
                                        <q-icon name="event" class="cursor-pointer">
                                            <q-popup-proxy transition-show="scale" transition-hide="scale" ref="p1">
                                                <q-date v-model="data.regdate.from" mask="DD.MM.YYYY" @update:model-value="$refs.p1.hide()">

                                                </q-date>
                                            </q-popup-proxy>
                                        </q-icon>
                                    </template>


                                </q-input>
                            </div>
                            <div class="col-6">
                                <q-input label="Дата регистрации по" v-model="data.regdate.to" clearable readonly>
                                    <template v-slot:append>
                                        <q-icon name="cancel" v-if="data.regdate.to!=null" @click="data.regdate.to=null" class="cursor-pointer" ></q-icon>
                                        <q-icon name="event" class="cursor-pointer">
                                            <q-popup-proxy transition-show="scale" transition-hide="scale" ref="p2">
                                                <q-date v-model="data.regdate.to" mask="DD.MM.YYYY" @update:model-value="$refs.p2.hide()">

                                                </q-date>
                                            </q-popup-proxy>
                                        </q-icon>
                                    </template>


                                </q-input>
                            </div>

                        </div>
                        <div class="row form-row" v-if="withActivity">
                            <div class="col-6">
                                <q-input label="Последняя активность с" v-model="data.activity.from" clearable readonly>
                                    <template v-slot:append>
                                        <q-icon name="cancel" v-if="data.activity.from!=null" @click="data.activity.from=null" class="cursor-pointer" ></q-icon>
                                        <q-icon name="event" class="cursor-pointer">
                                            <q-popup-proxy transition-show="scale" transition-hide="scale" ref="p3">
                                                <q-date v-model="data.activity.from" mask="DD.MM.YYYY" @update:model-value="$refs.p3.hide()">

                                                </q-date>
                                            </q-popup-proxy>
                                        </q-icon>
                                    </template>


                                </q-input>
                            </div>
                            <div class="col-6">
                                <q-input label="Последняя активность по" v-model="data.activity.to" clearable readonly>
                                    <template v-slot:append>
                                        <q-icon name="cancel" v-if="data.activity.to!=null" @click="data.activity.to=null" class="cursor-pointer" ></q-icon>
                                        <q-icon name="event" class="cursor-pointer">
                                            <q-popup-proxy transition-show="scale" transition-hide="scale" ref="p4">
                                                <q-date v-model="data.activity.to" mask="DD.MM.YYYY" @update:model-value="$refs.p4.hide()" >

                                                </q-date>
                                            </q-popup-proxy>
                                        </q-icon>
                                    </template>


                                </q-input>
                            </div>

                        </div>

                    </q-form>
                </q-page>
            </q-page-container>
        </q-layout>


    </q-dialog>
</template>
<script>
import {defineComponent} from 'vue';
import Helpers from 'src/lib/api/helpers';
import CustomButton from 'src/components/CustomButton';
import { ref } from 'vue';

export default defineComponent({
    name: "UserSearchDialog",
    props: ['trigger', 'withActivity', 'withSsoid'],
    emits: ['saved', 'cancel'],
    components: {CustomButton},
    computed: {

    },
    watch: {
        trigger(){
            this.show = this.trigger;
        }
    },
    data() {
        return {
            show:false,
            date:ref('01.02.2019'),
            data:{
                ssoid:null,
                email:null,
                first_name:null,
                last_name:null,
                middle_name:null,
                regdate:{from:null, to:null},
                activity:{from:null, to:null},
                extLabel:null
            }
        };
    },
    methods: {
        cleanVals(){
            this.data = {
                ssoid:null,
                email:null,
                first_name:null,
                last_name:null,
                middle_name:null,
                regdate:{from:null, to:null},
                activity:{from:null, to:null},
                extLabel:null
            };
        },
        getErrors(field) {
            //выдаёт ошибку для поля, которую возвращает Yii
            if (!this.errors || !this.errors[field]) return true;
            return this.errors[field].join(', ');
        },
        validateAll() {
            //подсвечивает ошибки на всех полях
            this.$refs.form.getValidationComponents().forEach((comp) => comp.validate());
        },
        cancelEdit() {
            this.$emit('cancel');
        },
        labelAdd(res, value, label) {
            if (value) {
                if (res !== '') res = res + ', ';
                res += label + ': ' + value;
            }
            return res;
        },
        save() {
            let label = '';
            label = this.labelAdd(label, this.data.last_name, 'Фамилия');
            label = this.labelAdd(label, this.data.first_name, 'Имя');
            label = this.labelAdd(label, this.data.middle_name, 'Отчество');
            label = this.labelAdd(label, this.data.email, 'E-Mail');
            label = this.labelAdd(label, this.data.ssoid, 'SSO ID');
            label = this.labelAdd(label, this.data.regdate.from, 'Дата регистрации с');
            label = this.labelAdd(label, this.data.regdate.to, 'Дата регистрации по');
            label = this.labelAdd(label, this.data.activity.from, 'Дата активности с');
            label = this.labelAdd(label, this.data.activity.to, 'Дата активности по');
            if (label=='') label = null;
            this.data.extLabel = label;
           this.$emit('saved', this.data);
        },
        ...Helpers
    }

});
</script>
<style>
.myForm div.row > div {
    padding-left: 10px;
}
</style>