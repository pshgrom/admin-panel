<template>
    <q-dialog v-model="showDialog" @escape-key="cancelEdit">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout" style="min-width: 1100px;width: 1100px">
            <q-header bordered>
                <q-toolbar>
                    <q-toolbar-title v-html="dialogTitle"></q-toolbar-title>
                    <q-btn flat v-close-popup round dense icon="close" @click="cancelEdit"/>
                </q-toolbar>
            </q-header>

            <q-footer bordered>
                <custom-button title="Отмена" type="light" @click="cancelEdit" />
                <custom-button :title="obj.id===0?'Создать':'Сохранить'" type="purple" @click="save" />
            </q-footer>


            <q-page-container>
                <q-page padding>
                    <q-form ref="form" @submit="save" dense>
                        <div class="row form-row" v-if="tab!='test'">
                            <div class="col-3">
                                <q-input
                                    v-model="obj.name"
                                    label="Код *"
                                    :rules="[val => getErrors('name'), val => !!val || '* Необходимо заполнить']"
                                    dense
                                    outlined
                                    class="q-mr-md"/>
                            </div>
                            <div class="col-6">
                                <q-select
                                    v-model="obj.subscription_id" label="Подписка"
                                    option-value="id"
                                    option-label="title"
                                    :options="subscriptions"
                                    map-options
                                    emit-value
                                    dense
                                    outlined/>
                            </div>
                            <div class="col-3">
                                <q-checkbox v-model="obj.sends_push" label="Высылает push" outlined/>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-9">
                                <q-input v-model="obj.title" label="Название"
                                         :rules="[val => getErrors('title')]" dense outlined/>
                            </div>
                            <div class="col-3">
                                <q-checkbox v-model="obj.sends_emp" label="Отправляет в ЕЛК" outlined/>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-input v-model="obj.description" label="Описание"
                                         :rules="[val => getErrors('description')]" type="textarea" autogrow dense
                                         outlined/>
                            </div>
                        </div>
                      <div class="row form-row">
                        <div class="col-6">
                          <q-input
                          v-model="obj.title_from"
                          :rules="[val => getErrors('title_from')]"
                          label="Заголовок отправителя"
                          placeholder="Наш город 2.0"
                          type="textarea"
                          autogrow
                          dense
                          outlined/>
                        </div>

                        <div class="col-6">
                          <q-input
                          v-model="obj.email_from"
                          :rules="[val => getErrors('email_from')]"
                          label="Email отправителя"
                          placeholder="example@example.test"
                          type="textarea"
                          autogrow
                          dense
                          outlined/>
                        </div>
                      </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-tabs v-model="tab" class="bg-indigo-1" dense align="left">
                                    <q-tab name="mail" label="Шаблон ЕПС (е-mail)"/>
                                    <q-tab name="push" label="Шаблон Push" v-if="obj.sends_push"/>
<!--                                    <q-tab name="test" label="Тест"/>-->
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
                                            <div class="col-12">
                                                <q-input v-model="obj.mail_title" label="Заголовок *"
                                                         :rules="[val => getErrors('mail_title'), val => !!val || '* Необходимо заполнить']"
                                                         dense outlined/>
                                            </div>
                                            <div class="col-12">
                                                <q-editor
                                                    v-model="obj.mail_body"
                                                    :dense="$q.screen.lt.md"
                                                    :toolbar="editorToolbar"
                                                    :fonts="fonts"
                                                    label="Анонс новости"
                                                />
                                            </div>
                                        </div>

                                    </q-tab-panel>

                                    <q-tab-panel name="push">
                                        <div class="row">
                                            <div class="col-12">
                                                <q-input v-model="obj.push_body" label="Текст"
                                                         :rules="[val => getErrors('push_body')]" type="textarea"
                                                         autogrow
                                                         dense outlined/>
                                            </div>
                                        </div>
                                    </q-tab-panel>

                                    <q-tab-panel name="test">
                                        <div class="row">

                                            <div class="col-9">
                                                <q-input v-model="obj.requires"
                                                         label="Требует параметры (через запятую)"
                                                         placeholder="issue,user"
                                                         dense outlined class="q-mr-md q-pb-md"/>
                                            </div>
                                            <div class="col-3">
                                                <q-btn dense color="primary" label="Форматировать JSON"
                                                       @click="formatSample" outline/>
                                            </div>
                                            <div class="col-9">
                                                <q-input v-model="obj.sample_json" label="Пример json"
                                                         :rules="[val => getErrors('sample_json')]" type="textarea"
                                                         autogrow
                                                         dense outlined class="q-mr-md"/>
                                            </div>
                                            <div class="col-3">
                                                <q-btn size="14px" dense color="dark" label="Запустить тест"
                                                       @click="test"
                                                       outline/>
                                            </div>

                                            <div class="col-12">
                                                <q-card class="text-result">
                                                    <q-card-section v-html="test_result"></q-card-section>
                                                </q-card>
                                            </div>

                                        </div>

                                    </q-tab-panel>
                                </q-tab-panels>
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
import Api from 'src/lib/mailer/api';
import ErrorHints from 'src/components/ErrorHints';
import state from 'src/lib/state';
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
    name: "TemplateEditDialog",
    props: ['obj'],
    emits: ['saved', 'cancel'],
    components: { ErrorHints, CustomButton },
    computed: {
        showDialog() {
            return this.obj != null;
        },
        dialogTitle() {
            if (this.obj.id) return 'Шаблон ' + this.obj.name;
            return 'Новый шаблон';
        }
    },
    data() {
        return {
            tab: 'mail',
            isNew: false,
            errors: null,
            editorToolbar: state.editorToolbar(this.$q),
            fonts: state.editorFonts,
            subscriptions: [],
            test_result: 'Тест не запускался'
        };
    },
    created() {
        this.loadSubscriptions();
    },
    watch: {
        'obj.sends_push'() {
            if (this.tab == 'push' && !this.obj.sends_push) this.tab = 'mail';
        }
    },
    methods: {
        async loadSubscriptions() {
            const data = await Api.subscriptions.list({page: 1, rowsPerPage: 1000}, {});
            this.subscriptions = data.list;
        },
        test() {
        },
        formatSample() {
            try {
                let json = JSON.parse(this.obj.sample_json.trim());
                if (json) {
                    this.obj.sample_json = JSON.stringify(json, null, "\t");
                }
            } catch (e) {
                this.$q.notify({
                    message: 'Невалидный JSON',
                    caption: '',
                    color: 'red'
                });
            }

        },
        getErrors(field) {
            //выдаёт ошибку для поля, которую возвращает Yii
            if (!this.errors || !this.errors[field]) return true;
            return this.errors[field].join(', ');
        },
        validateAll() {
            //подсвечивает ошибки на всех полях
            let res = true;
            this.$refs.form.getValidationComponents().forEach((comp) => {
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
            if (!this.validateAll()) return;

            this.isNew = this.obj.id === 0;
            Api.templates.save(this.obj).then((data) => {
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

        }
    }

});
</script>