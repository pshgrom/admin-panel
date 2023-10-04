<template>
    <q-dialog v-model="showDialog" @escape-key="cancelEdit">

        <q-card style="min-width: 1000px;width: 1000px">
            <q-form ref="form" @submit="save">
                <q-card-section class="dialog-header row items-center q-pb-none">
                    <div class="text-h6">Настройки системы уведомлений</div>
                    <q-space/>
                    <q-btn icon="close" flat round dense v-close-popup @click="cancelEdit"/>
                </q-card-section>
                <q-card-section class="dialog-body" style="padding-top: 0;">
<!--                    <div class="row">-->
<!--                        <div class="col-12">-->
<!--                            <q-toggle v-model="obj.is_test" label="Общий тестовый режим" :disable="!godMode"/>-->
<!--                        </div>-->
<!--                        <div class="col-12">-->
<!--                            В тестовом режиме сообщения отсылаются только тем, чьи адреса перечислены ниже-->
<!--                        </div>-->
<!--                    </div>-->
                    <div class="row form-row">
                        <div class="col-12">
                            <q-input v-model="obj.allowed_emails" type="textarea" autogrow label="Разрешённые емэйлы" dense outlined/>
                        </div>
                    </div>
                </q-card-section>

                <q-card-actions class="bg-white text-primary justify-end">
                    <custom-button title="Отмена" type="light" @click="cancelEdit" />
                    <custom-button title="Сохранить" type="purple" @click="save" />
                </q-card-actions>
            </q-form>
        </q-card>

    </q-dialog>
</template>

<script>
import {defineComponent} from 'vue';
import Api from 'src/lib/mailer/api';
import ErrorHints from 'src/components/ErrorHints';
import CustomButton from 'src/components/CustomButton';
import globalState from 'src/lib/state';

export default defineComponent({
    name: "MailerSettingsDialog",
    props: ['show'],
    emits: ['saved', 'cancel'],
    components: { ErrorHints, CustomButton },
    computed: {
        showDialog() {
            return this.show && this.obj!=null;
        },
        godMode(){
            return globalState.hiddenMenu;
        }
    },

    watch: {
        show() {
            if (this.show) {
                Api.settings.load().then(data => {
                    this.obj = data;
                });
            }
        }
    },
    data() {
        return {
            obj: null,
            errors: null
        };
    },
    methods: {
        cancelEdit() {
            this.$emit('cancel');
        },
        save() {
            Api.settings.save(this.obj).then((data) => {
                if (data._errors) {
                    this.$q.notify({
                        message: 'Сохранено',
                        caption: '',
                        color: 'green'
                    });
                } else {
                    this.$emit('saved', {obj: data});
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