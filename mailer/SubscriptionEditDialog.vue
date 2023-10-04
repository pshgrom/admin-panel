<template>
   <q-dialog v-model="showDialog"  @escape-key="cancelEdit">

      <q-card style="min-width: 500px;width: 500px">
         <q-form ref="form" @submit="save">
            <q-card-section class="dialog-header row items-center q-pb-none">
               <div class="text-h6">{{dialogTitle}}</div>
               <q-space/>
               <q-btn icon="close" flat round dense v-close-popup @click="cancelEdit"/>
            </q-card-section>
            <q-card-section class="q-pt-none">
               <div class="row">
                  <div class="col-6">
                     <q-input v-model="obj.code" label="Код уведомления *"
                              :rules="[val => getErrors('code'), val => !!val || '* Необходимо заполнить']" dense outlined/>
                  </div>
                  <div class="col-6">

                  </div>
               </div>
               <div class="row">
                  <div class="col-12">
                     <q-input v-model="obj.title" label="Название *"
                              :rules="[val => getErrors('title'), val => !!val || '* Необходимо заполнить']" dense outlined/>
                  </div>
               </div>
               <div class="row">
                  <div class="col-6">
                     <q-checkbox v-model="obj.is_group" label="Это группа подписок" dense/>
                  </div>
                  <div class="col-6">
                     <q-checkbox v-model="obj.is_test" label="Скрытый/тестовый тип" dense/>
                  </div>
               </div>
            </q-card-section>

            <q-card-actions class="bg-white text-primary justify-end">
               <q-btn label="Отмена" @click="cancelEdit" outline/>
               <q-btn label="Сохранить" @click="save" color="primary"/>
            </q-card-actions>
         </q-form>
      </q-card>

   </q-dialog>
</template>

<script>
    import {defineComponent} from 'vue';
    import Api from 'src/lib/mailer/api';
    import ErrorHints from 'src/components/ErrorHints';

    export default defineComponent({
        name: "SubscriptionEditDialog",
        props: ['obj'],
        emits: ['saved', 'cancel'],
        components: {ErrorHints},
        computed: {
            showDialog() {
                return this.obj != null;
            },
            dialogTitle() {
                if (this.obj.id) return 'Тип уведомления №' + this.obj.id;
                return 'Новый тип уведомления';
            }
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
            validateAll(){
              //подсвечивает ошибки на всех полях
              let res = true;
              this.$refs.form.getValidationComponents().forEach((comp)=> {
                const valid = comp.validate();
                res = res && valid;
              });
              return res;
            },
            cancelEdit() {
                this.$emit('cancel');
            },
            save() {
                this.errors = null;
                this.$refs.form.resetValidation();
                if(!this.validateAll()) return;

                this.isNew = this.obj.id === 0;
                Api.subscriptions.save(this.obj).then((data) => {
                    if (data._errors) {
                        this.errors = data._errors;
                        this.validateAll();
                        this.$q.notify({
                            message: 'Сохранено',
                            caption: '',
                            color: 'green'
                        });
                    } else {
                        this.$emit('saved', {obj: data, append: this.isNew});
                    }
                });

            }
        }

    });
</script>
<style>
   .myForm div.row > div {
      padding-left: 10px;
   }
</style>