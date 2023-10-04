<template>
    <q-dialog v-model="showDialog" @escape-key="cancelEdit">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout" style="min-width: 900px;width: 900px">
            <q-header bordered>
                <q-toolbar>
                    <q-toolbar-title v-html="dialogTitle"></q-toolbar-title>
                    <q-btn flat v-close-popup round dense icon="close" @click="cancelEdit"/>
                </q-toolbar>
            </q-header>

            <q-footer bordered>
                <custom-button title="Отмена" type="light" @click="cancelEdit"/>
                <custom-button :title="obj.id===0?'Создать':'Сохранить'" type="purple" @click="save"/>
            </q-footer>


            <q-page-container>
                <q-page padding>
                    <q-form ref="form" @submit="save" class="user-form" dense>


                        <div class="row form-row">
                            <div class="col-4">
                                <q-input outlined label="Фамилия" dense v-model="obj.last_name"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input outlined label="Имя" dense v-model="obj.first_name"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input outlined label="Отчество" dense v-model="obj.middle_name"></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-input outlined label="Должность" dense v-model="obj.position_name"></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-8" style="padding-top:10px;text-align: right">
                                Дата начала действия
                            </div>
                            <div class="col-4">
                                <q-input outlined dense v-model="date">
                                    <template v-slot:prepend>
                                        <q-icon name="event" class="cursor-pointer">
                                            <q-popup-proxy cover transition-show="scale"
                                                           transition-hide="scale">
                                                <q-date v-model="date" mask="DD.MM.YYYY">
                                                    <div class="row items-center justify-end">
                                                        <custom-button title="Применить" type='light' v-close-popup
                                                        />
                                                    </div>
                                                </q-date>
                                            </q-popup-proxy>
                                        </q-icon>
                                    </template>
                                </q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-banner class="text-white bg-primary">
                                    Полная подпись:<br/>
                                    <span v-html='obj.sign?obj.sign.replaceAll("\n", "<br/>"):""'> </span>
                                </q-banner>
                            </div>


                        </div>

                    </q-form>
                </q-page>
            </q-page-container>
        </q-layout>


    </q-dialog>
</template>
<style>
.user-form .col .row {
    margin-right: 10px;
}
</style>
<script>
import {defineComponent} from 'vue';
import Api from 'src/lib/pos/api';
import ErrorHints from 'src/components/ErrorHints';
import Helpers from 'src/lib/api/helpers';
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
    name: "SignEditDialog",
    props: ['obj', 'org'],
    emits: ['saved', 'cancel'],
    components: {ErrorHints, CustomButton},
    computed: {
        showDialog() {
            return this.obj != null;
        },
        dialogTitle() {
            if (this.obj.id > 0) {
                return 'Редактирование подписи';
            } else {
                return 'Новая подпись';
            }
        },
    },
    watch: {
        obj: {
            // This will let Vue know to look inside the array
            deep: true,
            handler() {
                document.obj = this.obj;
                if (this.obj) {
                    this.date = Helpers.formatUnixDate(this.obj.started_at, false);
                    let orgName = '';
                    if (this.org) {
                        if (this.org.short_name == null || this.org.short_name == '') {
                            orgName = this.org.name;
                        } else {
                            orgName = this.org.short_name;
                        }
                    }
                    let first_name = this.obj.first_name ?? '';
                    let last_name = this.obj.last_name ?? '';
                    let middle_name = this.obj.middle_name ?? '';
                    let sign = last_name + ' '
                        + (first_name.length > 0 ? first_name[0] + '.' : '')
                        + (middle_name.length > 0 ? middle_name[0] + '.' : '')
                        + '\n';
                    if (this.obj.position_name && this.obj.position_name.length > 0) {
                        sign += 'Должность: ' + (this.obj.position_name ?? '') + '\n';
                    }
                    if (orgName !== '') {
                        sign += 'Организация: ' + orgName;
                    }
                    this.obj.sign = sign;
                }
            }
        }
    },
    data() {
        return {
            date: null,
            isNew: false,
            errors: null
        };
    },
    methods: {
        cancelEdit() {
            this.$emit('cancel');
        },
        save() {
            this.obj.started_at = this.date?Helpers.toUnixTime(this.date):null;
            this.$emit('saved', {obj: this.obj, append: !(this.obj.id > 0)});
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