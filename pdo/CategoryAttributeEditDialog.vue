<template>
   <q-dialog v-model="showDialog" v-if="this.obj">

      <q-card style="min-width: 900px;width: 900px">
         <q-form ref="form" @submit="save" class="user-form" dense>
            <q-card-section class="dialog-header row items-center q-pb-none">
               <div class="text-h6">{{dialogTitle}}</div>
               <q-space/>
               <q-btn icon="close" flat round dense v-close-popup @click="cancelEdit"/>
            </q-card-section>

            <q-card-section>

               <div class="row">
                  <div class="col-4">
                     <q-input outlined label="Код" v-model="obj.code"></q-input>
                  </div>

                  <div class="col-8">
                     <q-input outlined label="Название" v-model="obj.name" ></q-input>
                  </div>
               </div>

               <div class="row">
                  <div class="col-3">
                     <q-checkbox v-model="obj.can_be_virtual" label="Виртуальные объекты"/>
                  </div>
                  <div class="col-6">
                     <q-checkbox v-model="obj.can_add_object" label="Доступна форма об отсутствии объекта"/>
                  </div>


               </div>
               <div class="row">
                  <div class="col-3">
                     <q-checkbox v-model="obj.is_polygon" label="Полигональные"/>
                  </div>
                  <div class="col-3">
                     <q-checkbox v-model="obj.is_geo_required" label="Требуется геопозиция"/>
                  </div>
                  <div class="col-3">
                     <q-checkbox v-model="obj.is_only_in_moscow" label="Только в Москве"/>
                  </div>

               </div>
               <div class="row">
                  <div class="col-3">
                     <q-checkbox v-model="obj.is_district_required" label="Требуется район"/>
                  </div>
                  <div class="col-6">
                     <q-checkbox v-model="obj.show_in_map" label="Отображать на карте объектов"/>
                  </div>

               </div>
            </q-card-section>

            <q-card-actions class="bg-white text-primary justify-end">
               <q-btn label="Отмена" @click="cancelEdit"/>
               <q-btn label="Сохранить" @click="save" color="dark"/>
            </q-card-actions>
         </q-form>
      </q-card>

   </q-dialog>
</template>
<style>
   .user-form .col .row {
      margin-right: 10px;
   }
</style>
<script>
    import {defineComponent} from 'vue';
    import Api from 'src/lib/pdo/api';
    import ErrorHints from 'src/components/ErrorHints';
    import Helpers from 'src/lib/api/helpers';

    export default defineComponent({
        name: "CategoryAttributeEditDialog",
        props: ['obj'],
        emits: ['saved', 'cancel'],
        components: {ErrorHints},
        computed: {
            showDialog() {
                return this.obj != null;
            },
            dialogTitle() {
                if (this.obj.id !== 0) return this.obj.name === '' ? 'Нет имени' : this.obj.name;
                return 'Новый аттрибут';
            }
        },
        watch: {

        },
        data() {
            return {
                isNew: false,
                errors: null
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
                this.isNew = this.obj.id === 0;
                this.$refs.form.resetValidation();
                Api.categories.save(this.obj).then((data) => {
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