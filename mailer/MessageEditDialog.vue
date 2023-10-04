<template>
    <q-dialog v-model="showDialog" @escape-key="cancelEdit">

        <q-card style="min-width: 1200px;width: 1200px">
            <q-form ref="form" @submit="cancelEdit">
                <q-card-section class="dialog-header row items-center q-pb-none">
                    <div class="text-h6">{{ dialogTitle }}</div>
                    <q-space/>
                    <q-btn icon="close" flat round dense v-close-popup @click="cancelEdit"/>
                </q-card-section>
                <q-card-section class="dialog-body" style="padding-top: 0;">
                    <q-tabs v-model="tab" class="bg-indigo-1" dense align="left">
                        <q-tab name="mail" label="Сообщение"/>
                        <q-tab name="log" label="Лог"/>
                    </q-tabs>
                    <q-tab-panels
                        v-model="tab"
                        animated
                        vertical
                        transition-prev="slide-right"
                        transition-next="slide-left"
                        style="min-height: 300px"
                        class="np-panels"
                    >
                        <q-tab-panel name="mail">
                            <div class="row">
                                <div class="col-4" style="padding-right: 10px">
                                    <div class="row">
                                        <div class="col-12">
                                            <q-input v-model="obj.source_id" readonly label="Источник" dense></q-input>
                                        </div>
                                        <div class="col-12">
                                            <q-input v-model="obj.sender_id" readonly label="Компонент" dense></q-input>
                                        </div>
                                        <div class="col-12">
                                            <q-input v-model="obj.email" readonly label="E-Mail" dense></q-input>
                                        </div>
                                        <div class="col-12" style="margin:10px 0 0 0" v-if="obj.status==3">
                                            <span
                                                style="font-weight: bold">Время отправки E-mail:</span>&nbsp{{ unixTime(obj.sent_at, true) }}
                                        </div>
                                        <div class="col-12" style="margin:10px 0px">
                                            <q-btn
                                                :label="statusName(obj.status, 'E-Mail')"
                                                :style="btnStateColorStyle(obj.status)"
                                                dense
                                                class="message-status"/>
                                        </div>

                                        <div class="col-12" style="margin:10px 20px 0 0" v-if="obj.push_status==3">
                                            <span
                                                style="font-weight: bold">Время отправки push:</span>&nbsp{{ unixTime(obj.push_sent_at, true) }}
                                        </div>
                                        <div class="col-12" style="margin:10px 0px">
                                            <q-btn
                                                :label="statusName(obj.push_status, 'push')"
                                                :style="btnStateColorStyle(obj.push_status)"
                                                dense
                                                class="message-status"/>
                                        </div>

                                        <div class="col-12" style="margin:10px 0 0 0" v-if="obj.emp_status==3">
                                            <span
                                                style="font-weight: bold">Время отправки в ЕЛК:</span>&nbsp{{ unixTime(obj.emp_sent_at, true) }}
                                        </div>
                                        <div class="col-12" style="margin:10px 0px">
                                            <q-btn
                                                :label="statusName(obj.emp_status, 'ЕЛК')"
                                                :style="btnStateColorStyle(obj.emp_status)"
                                                dense
                                                class="message-status"/>
                                        </div>
                                    </div>

                                </div>
                                <div class="col-8">
                                    <div v-html="msgBody"></div>
                                </div>
                            </div>
                        </q-tab-panel>

                        <q-tab-panel name="log">
                            <message-log-table :message_id="message_id"></message-log-table>
                        </q-tab-panel>
                    </q-tab-panels>

                </q-card-section>

                <q-card-actions class="bg-white text-primary justify-end">
                    <custom-button title="Закрыть" type="light" @click="cancelEdit" />
                </q-card-actions>
            </q-form>
        </q-card>

    </q-dialog>
</template>

<script>
import {defineComponent} from 'vue';
import Api from 'src/lib/mailer/api';
import Helpers from 'src/lib/api/helpers';
import MessageLogTable from 'src/components/mailer/MessageLogTable';
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
    name: "SubscriptionEditDialog",
    props: ['obj'],
    emits: ['saved', 'cancel'],
    components: { MessageLogTable, CustomButton },
    computed: {
        message_id() {
            let mi = this.obj ? this.obj.id : 0;
            return mi;
        },
        showDialog() {
            return this.obj != null;
        },
        dialogTitle() {
            if (this.obj.id) return 'Уведомление №' + this.obj.id;
            return 'Новый тип уведомления';
        },
        msgBody() {
            let body = this.obj.body.replace(/<img name=\\"pixel_hash\\" id=\\"pixel_hash\\" src[\d\w\?\&\=\:\/\.].*height=\\"1\\" width=\\"1\\">>/gm, '');
            body = body.replace(/<img name=\"pixel_hash\".*?>/gm, '');
            return body;
        }
    },
    watch: {
        obj() {
            if (this.obj) {
                this.loadTemplate();
                this.loadLog();
            }
        }
    },
    data() {
        return {
            isNew: false,
            errors: null,
            tab: "mail",
            options: {
                mail: true,
                push: true,
                elk: true
            },

        };
    },
    methods: {
        async loadTemplate() {
            let templateId = 0;
            if (this.obj.template_id > 0) {
                let template = await Api.templates.get(this.obj.template_id);
                if (template) {
                    this.options.push = template.sends_push;
                    this.options.elk = template.sends_emp;
                }
            } else {
                this.options.mail = true;
                this.options.push = true;
                this.options.elk = true;
            }
        },
        async loadLog() {
            let logs = await Api.logs.byMessage(this.obj.id);
        },
        btnStateColorStyle(state) {

            const STATUSES = {
                1: '#FF9D01',
                2: '#4A4F5E',
                3: '#486824',
                4: '#F55449',
                5: '#4A4F5E',
                6: '#FF9D01',
                7: '#4A4F5E'
            };
            return `background-color: ${STATUSES[state] ?? 5};`;
        },
        statusName(state, sys) {
            const STATUSES = {
                1: 'В ожидании',
                2: 'Черновик',
                3: 'Отправлено',
                4: 'Ошибка',
                5: 'Отменено',
                6: 'Повторная попытка',
                7: 'Не подписан'
            };
            const ret = STATUSES[state] ?? 'Неизвестно';
            return sys + ' ' + ret;

        },
        unixTime: Helpers.friendlyUnixDateTime,
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
            // this.errors = null;
            // this.isNew = this.obj.id === 0;
            // this.$refs.form.resetValidation();
            // Api.subscriptions.save(this.obj).then((data) => {
            //     if (data._errors) {
            //         this.errors = data._errors;
            //         this.validateAll();
            //         this.$q.notify({
            //             message: 'Сохранено',
            //             caption: '',
            //             color: 'green'
            //         });
            //     } else {
            //         this.$emit('saved', {obj: data, append: this.isNew});
            //     }
            // });

        }
    }

});
</script>
<style>
.myForm div.row > div {
    padding-left: 10px;
}
</style>