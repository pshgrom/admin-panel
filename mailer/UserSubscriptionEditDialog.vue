<template>
   <q-dialog v-model="showDialog"  @escape-key="cancelEdit">

      <q-card style="min-width: 700px;">
         <q-form ref="form" @submit="save">
            <q-card-section class="dialog-header row items-center q-pb-none no-wrap">
               <div class="text-h6">{{dialogTitle}}</div>
               <q-space/>
               <q-btn icon="close" flat round dense v-close-popup @click="cancelEdit"/>
            </q-card-section>
            <q-card-section class="q-pt-none" v-if="subscription!=null">
               <div class="row">
                  <div class="col-8">
                     Не ранее: {{subscription.time_start}}
                  </div>
                  <div class="col-4">
                     Не позднее: {{subscription.time_end}}
                  </div>
               </div>
               <div class="row">
                  <div class="col-8 bg-primary text-white">
                     Рассылка
                  </div>
                  <div class="col-2 bg-primary text-white">
                     E-Mail
                  </div>
                  <div class="col-2 bg-primary text-white">
                     Push
                  </div>
               </div>
               <div class="row" v-for="item in names" :key="item">
                  <div class="col-8">
                     {{mailName(item)}}
                  </div>
                  <div class="col-2">
                     {{getFlag('mail', item)}}
                  </div>
                  <div class="col-2">
                     {{subscription.push[item]?'да':'нет'}}
                  </div>
               </div>

            </q-card-section>

            <q-card-actions class="bg-white text-primary justify-end">
                <custom-button title="Отмена" type="light" @click="cancelEdit" />
                <!-- <custom-button title="Сохранить" type="purple" @click="save" /> -->
            </q-card-actions>
         </q-form>
      </q-card>

   </q-dialog>
</template>
<script>
    import {defineComponent} from 'vue';
    import Api from 'src/lib/mailer/api';
    import ErrorHints from 'src/components/ErrorHints';
    import CustomButton from "src/components/CustomButton";

    export default defineComponent({
        name: "UserSubscriptionEditDialog",
        props: ['obj'],
        emits: ['saved', 'cancel'],
        components: { ErrorHints, CustomButton },
        computed: {
            showDialog() {
                return this.obj != null;
            },
            dialogTitle() {
                if (this.obj.id) return 'Подписка №' + this.obj.ssoid;
                return 'Новая подписка';
            }
        },
        data() {
            return {
                isNew: false,
                errors: null,
                subscription:null,
                names:['info','volonteer','issue_er_decision','issue_moderation','issue_response']
            };
        },
        watch:{
            obj(){
                this.loadSubscription();
            }
        },
        methods: {
            getFlag(code, item){
                return this.subscription[code][item]?'да':'нет';
            },
            mailName(code){
               switch (code) {
                   case 'info':return 'Информационная';
                   case 'volonteer':return 'Волонтёрская';
                   case 'issue_er_decision':return 'О решении единой редакции по сообщению';
                   case 'issue_moderation':return 'Об отправке на модерацию';
                   case 'issue_response':return 'О получении ответа';
               }
               return '';
            },
            getErrors(field) {
                //выдаёт ошибку для поля, которую возвращает Yii
                if (!this.errors || !this.errors[field]) return true;
                return this.errors[field].join(', ');
            },
            validateAll(){
                //подсвечивает ошибки на всех полях
                this.$refs.form.getValidationComponents().forEach((comp)=>comp.validate());
            },
            cancelEdit() {
                this.$emit('cancel');
            },
            async loadSubscription(){
                if (!this.obj) return;
                Api.userSubscriptions.get(this.obj.ssoid).then(data=>{
                    this.subscription = data;
                    console.log(data);
                });
            },
            save() {
                this.errors = null;
                this.isNew = this.obj.id === 0;
                this.$refs.form.resetValidation();
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