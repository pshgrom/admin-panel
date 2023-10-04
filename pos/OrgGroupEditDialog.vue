<template>
    <q-dialog v-model="showDialog" @escape-key="cancelEdit" style="min-width: 500px;width: 500px">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout">
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
                            <div class="col-12">
                                <q-input outlined label="Название" dense v-model="obj.title"></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-input outlined label="Краткое название" dense v-model="obj.short_title"
                                        ></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-select outlined v-model="obj.source" label="Атрибут объекта сообщения"
                                          option-value="id"
                                          option-label="title"
                                          :options="sourceOptions" map-options emit-value dense/>
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
    name: "OrgGroupEditDialog",
    props: ['obj'],
    emits: ['saved', 'cancel'],
    components: {ErrorHints, CustomButton},
    computed: {
        showDialog() {
            return this.obj != null;
        },
        dialogTitle() {
            let str = '';
            if (this.obj.id>0){
                str += this.obj.id+': ';
                str +=  (this.obj.short_title ?? '') ;
                return str;
            }else {
                return 'Новая группа';
            }
        },
    },
    watch: {
        obj() {
            document.obj = this.obj;
        }
    },
    data() {
        return {
            isNew: false,
            birthDate: null,
            errors: null,
            sourceOptions:[{id: 'district', title: 'Округ'},
                {id: 'region', title: 'Район'},
                {id: 'object', title: 'Объект'}]
        };
    },
    methods: {


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
        save() {
            this.errors = null;
            this.$refs.form.resetValidation();
            Api.organization.groupSave(this.obj).then((data) => {
                if (data._errors) {
                    this.errors = data._errors;
                    this.validateAll();
                } else {
                    this.$q.notify({
                        message: 'Сохранено',
                        caption: '',
                        color: 'green'
                    });
                    this.$emit('saved', {obj: data, append: this.isNew});
                }
            });

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