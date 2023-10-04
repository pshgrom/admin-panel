<template>
    <q-dialog v-model="showDialog" v-if="this.obj">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout"
                  style="min-width: 1000px;width: 1000px;" :style="linked.count<0? 'max-height: 350px':'auto'">
            <q-header bordered>
                <q-toolbar>
                    <q-toolbar-title v-html="dialogTitle"></q-toolbar-title>
                    <q-btn flat v-close-popup round dense icon="close" @click="cancelEdit"/>
                </q-toolbar>
            </q-header>

            <q-footer bordered>
                <custom-button title="Отмена" type="light" @click="cancelEdit"/>
                <custom-button v-if="!readonly" :title="obj.resp_id===0?'Создать':'Сохранить'" type="purple"
                               @click="save"/>
            </q-footer>


            <q-page-container>
                <q-page padding>
                    <q-form ref="form" @submit="save" class="user-form" dense>
                        <div class="row form-row" v-if="obj.resp_id>0">
                            <div class="col-2">
                                <q-field outlined label="ПДО ID" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">{{
                                                obj.resp_id
                                            }}
                                        </div>
                                    </template>
                                </q-field>
                            </div>
                            <div class="col-10">
                                <q-field outlined label="Балансодержатель в ПДО" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">{{
                                                obj.resp_sname
                                            }}
                                        </div>
                                    </template>
                                </q-field>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <organization-select label="Ответственный в ПОС"
                                                     v-model="obj.org_id"
                                                     outlined
                                                     :readonly="readonly"></organization-select>
                            </div>
                        </div>

                        <div class="row form-row" v-if="linked.count>-1">
                            <div class="col-12">
                                <q-markup-table wrap-cells>
                                    <thead>
                                    <th colspan="4">Список объектов с данным балансодержателем в ПДО (Всего {{ linked.count }}, показано
                                        {{ linked.list.length }}):</th>
                                    </thead>
                                    <thead>
                                    <th>ID</th>
                                    <th>Внешний идентификатор</th>
                                    <th>Категория</th>
                                    <th>Наименование</th>
                                    </thead>
                                    <tbody>
                                    <tr v-for="item in linked.list" :key="item.id">
                                        <td class="text-left">{{ item.facility_id }}</td>
                                        <td class="text-left">{{ item.origin_global_id }}</td>
                                        <td class="text-left">{{ item.category_name }}</td>
                                        <td class="text-left"><a href="#" @click.prevent="$router.push('/pdo/object/'+item.facility_id)">{{
                                                item.facility_name
                                            }}</a></td>
                                    </tr>
                                    <tr>
                                        <td colspan="4" class="text-right">Всего {{ linked.count }}, показано
                                            {{ linked.list.length }}
                                        </td>
                                    </tr>
                                    </tbody>
                                </q-markup-table>

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
import PdoApi from 'src/lib/pdo/api';
import OrganizationSelect from "components/pos/OrganizationSelect";
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
    name: "OwnerCorrDialog",
    props: {
        obj: {type: Object, required: false},
        readonly: {type: Boolean, default: false}
    },
    emits: ['saved', 'cancel'],
    components: {OrganizationSelect, CustomButton},
    computed: {
        showDialog() {
            return this.obj != null;
        },
        dialogTitle() {
            if (!this.obj) return '';
            return 'Сопоставление организации';
        },
        editable() {
            return true;
        }
    },
    watch: {
        obj() {
            document.obj = this.obj;
            this.linked = {count: -1, list: []};
            if (this.obj) {
                PdoApi.owner.linkedRespObjects(this.obj.resp_id).then((data) => {
                    if (data._errors) {
                        this.errors = data._errors;
                        this.validateAll();
                    } else {
                        this.linked = data;
                    }
                });
            }
        }
    },
    data() {
        return {
            isNew: false,
            errors: null,
            linked: {count: -1, list: []}
        };
    },
    methods: {
        validationErrors(name) {
            let res = true;
            if (this.errors && this.errors[name] && Array.isArray(this.errors[name])) res = this.errors[name].join(', ');
            return res;
        },
        unixDate: Helpers.formatUnixDate,
        validateAll() {
            //подсвечивает ошибки на всех полях
            this.$refs.form.getValidationComponents().forEach((comp) => comp.validate());
        },
        cancelEdit() {
            this.$emit('cancel');
        },
        save(stay) {
            this.errors = null;
            this.isNew = this.obj.id === 0;
            this.$refs.form.resetValidation();
            PdoApi.owner.corrSave(this.obj).then((data) => {
                if (data._errors) {
                    this.errors = data._errors;
                    this.validateAll();
                } else {
                    this.$q.notify({
                        message: 'Сохранено',
                        caption: '',
                        color: 'green'
                    });
                    this.$emit('saved', {obj: data, append: this.isNew, stay: typeof stay == 'boolean' ? stay : false});
                }
            });

        },
        ...Helpers
    }

});
</script>