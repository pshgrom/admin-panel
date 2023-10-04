<template>
    <q-dialog v-model="showDialog" @escape-key="cancelEdit" style="min-width: 1050px;width: 1050px">
        <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout" style="min-width: 1100px;width: 1100px">
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
                            <div class="col-8">
                                <q-input label="SSOID" dense v-model="obj.ssoid" :readonly="true"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input label="Система-источник" dense v-model="obj.system_code_full"
                                         :readonly="true"></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-4">
                                <q-input label="Зарегистрирован" dense v-model="userCreatedAt"
                                         :readonly="true"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input label="Последние изменения" dense v-model="userEditedAt"
                                         :readonly="true"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input label="Последняя активность" dense v-model="userActiveAt"
                                         :readonly="true"></q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-8">
                                <q-input outlined label="E-Mail" dense v-model="obj.email"
                                         :readonly="true"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input outlined label="Дата рождения" v-model="birthDate" dense
                                         mask="##.##.####"
                                         :readonly="true">
                                    <!--                                <template v-slot:append>-->
                                    <!--                                    <div>-->
                                    <!--                                        <q-icon @click="birthDate = null" name="img:icons/clear-24px.svg" dense-->
                                    <!--                                                class="cursor-pointer"/>-->
                                    <!--                                        <q-icon name="event" class="cursor-pointer">-->
                                    <!--                                            <q-popup-proxy @before-show="prevBirthDate = birthDate" ref="qDateProxy"-->
                                    <!--                                                           transition-show="scale" transition-hide="scale">-->
                                    <!--                                                <q-date minimal v-model="birthDate" mask="DD.MM.YYYY">-->
                                    <!--                                                    <div class="row items-center justify-around">-->
                                    <!--                                                        <custom-button title="Отменить" type='light'-->
                                    <!--                                                                       @click="birthDate = prevBirthDate"-->
                                    <!--                                                                       v-close-popup/>-->
                                    <!--                                                        <custom-button title="Выбрать" type='light' v-close-popup-->
                                    <!--                                                                       @click="prevBirthDate = null"/>-->
                                    <!--                                                    </div>-->
                                    <!--                                                </q-date>-->
                                    <!--                                            </q-popup-proxy>-->
                                    <!--                                        </q-icon>-->
                                    <!--                                    </div>-->
                                    <!--                                </template>-->
                                </q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-4">
                                <q-input outlined label="Фамилия" dense v-model="obj.last_name"
                                         :readonly="true"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input outlined label="Имя" dense v-model="obj.first_name"
                                         :readonly="true"></q-input>
                            </div>
                            <div class="col-4">
                                <q-input outlined label="Отчество" dense v-model="obj.middle_name"
                                         :readonly="true"></q-input>
                            </div>


                        </div>
                        <div class="row form-row">

                            <div class="col-4">
                                <q-field outlined label="Пол" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">
                                            {{ gender }}
                                        </div>
                                    </template>
                                </q-field>
                            </div>
                            <div class="col-4">
                                <q-field outlined label="Цвет аватарки" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">
                                            {{ obj.color ?? '' }}
                                            <span v-if="obj.color" :style="avatarColorStyle"></span>
                                        </div>
                                    </template>
                                </q-field>

                            </div>
                            <div class="col-4">
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-2">
                                <q-checkbox v-model="obj.is_volunteer" dense label="Волонтёр" :disable="true"/>
                            </div>
                            <div class="col-2">
                                <q-checkbox v-model="is_deputy" dense label="Депутат"
                                            :disable="true"/>
                            </div>
                            <div class="col-3">
                                <q-checkbox v-model="is_confirmed" dense label="Подтверждённая УЗ mos.ru"
                                            :disable="true"/>
                            </div>
                            <div class="col-3">
                                <q-checkbox v-model="obj.is_snils_exists" dense label="СНИЛС подтвержден"
                                            :disable="true"/>
                            </div>
                            <div class="col-2">
                                <q-checkbox v-model="is_level_full" dense label="Полная УЗ"
                                            :disable="true"/>
                            </div>
                        </div>
                        <div class="row form-row">

                            <div class="col-2" style="padding-top:10px">
                                <q-checkbox v-model="is_citizen" dense label="Житель" :disable="true"/>
                            </div>
                            <div class="col-4">
                                <q-field outlined label="Псевдоним" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">
                                            {{ obj.alias }}
                                        </div>
                                    </template>
                                </q-field>

                            </div>
                            <div class="col-3">
                                <q-field outlined label="Дата модерации" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">
                                            {{ formatUnixDate(obj.approved_at) }}
                                        </div>
                                    </template>
                                </q-field>

                            </div>
                            <div class="col-3">
                                <q-field outlined label="Модератор" stack-label dense>
                                    <template v-slot:control>
                                        <div class="self-center full-width no-outline" tabindex="0">
                                            {{ obj.approved_by_name == '' ? '' : obj.approved_by_name }}
                                        </div>
                                    </template>
                                </q-field>

                            </div>
                        </div>
                        <div class="row form-row" v-if="obj.deputy_title&&obj.deputy_title!=''">
                            <div class="col-12">
                                <q-input label="Подпись депутата" type="textarea" autogrow dense outlined readonly
                                         v-model="obj.deputy_title">

                                </q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">
                                <q-input label="Комментарий" type="textarea" autogrow dense outlined
                                         v-model="obj.comment">

                                </q-input>
                            </div>
                        </div>
                        <div class="row form-row">
                            <div class="col-12">{{ userAdress(obj) }}</div>
                        </div>
                        <div class="row" v-if="false">
                            <div class="col"><span style="font-size: 1.2em;font-weight: bold">ОМС</span></div>
                        </div>
                        <div class="row" v-if="false">
                            <div class="col-3">
                                <q-input label="SSOID" v-model="obj.ssoid" :readonly="true"></q-input>
                            </div>
                            <div class="col-3">
                                <q-input label="Зарегистрирован" v-model="userCreatedAt"
                                         :readonly="true"></q-input>
                            </div>
                            <div class="col-3">
                                <q-input label="Последние изменения" v-model="userEditedAt"
                                         :readonly="true"></q-input>
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
import Api from 'src/lib/api/admin-api';
import ErrorHints from 'src/components/ErrorHints';
import Helpers from 'src/lib/api/helpers';
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
    name: "UserEditDialog",
    props: ['obj'],
    emits: ['saved', 'cancel'],
    components: {ErrorHints, CustomButton},
    computed: {
        userCreatedAt() {
            if (!this.obj) return 'нет даты';
            return this.formatUnixDate(this.obj.created_at);
        },
        userEditedAt() {
            if (!this.obj || !this.obj.updated_at) return 'никогда';
            return this.formatUnixDate(this.obj.updated_at);
        },
        userActiveAt() {
            if (!this.obj || !this.obj.activity_at) return '';
            return this.formatUnixDate(this.obj.activity_at) + (this.obj.system_name == '' ? '' : ' [' + this.obj.system_name + ']');
        },
        showDialog() {
            return this.obj != null;
        },
        dialogTitle() {
            let str = '';
            if (this.obj.id>0){
                str += this.obj.id+': ';
                str +=  (this.obj.last_name ?? '') + ' ' + (this.obj.first_name ?? '');
                return str;
            }else {
                return 'Новый пользователь';
            }
        },
        avatarColorStyle() {
            let color = this.obj.color;
            if (color == null) color = '#ffffff';
            return 'display:inline-block;width:30px;height:10px;margin-top:10px; border:1px solid black;background-color:' + color;
        },
        is_confirmed() {
            //
            return this.obj.is_trusted;
        },
        is_citizen() {
            return this.obj.is_citizen == 1;
        },
        gender() {
            return this.obj.gender ? this.obj.gender === 'm' ? 'мужской' : 'женский' : '';
        },
        is_deputy() {
            return this.obj.is_deputy == 1;
        },
        is_level_full(){
            return this.obj.lvl=='full';
        }

    },
    watch: {
        obj() {
            document.obj = this.obj;
            if (this.obj && this.obj.birthdate) {
                this.birthDate = Helpers.formatUnixDate(this.obj.birthdate, false);
            }
            console.log(this.obj);
        }
    },
    data() {
        return {
            isNew: false,
            birthDate: null,
            errors: null,
            genderOptions: [{label: 'Мужской', value: 'male'},
                {label: 'Женский', value: 'female',}]
        };
    },
    methods: {

        invertColor(hex) {
            if (hex.indexOf('#') === 0) {
                hex = hex.slice(1);
            }
            // convert 3-digit hex to 6-digits.
            if (hex.length === 3) {
                hex = hex[0] + hex[0] + hex[1] + hex[1] + hex[2] + hex[2];
            }
            if (hex.length !== 6) {
                throw new Error('Invalid HEX color.');
            }
            // invert color components
            var r = (255 - parseInt(hex.slice(0, 2), 16)).toString(16),
                g = (255 - parseInt(hex.slice(2, 4), 16)).toString(16),
                b = (255 - parseInt(hex.slice(4, 6), 16)).toString(16);
            // pad each with zeros and return
            return '#' + this.padZero(r) + this.padZero(g) + this.padZero(b);
        },

        padZero(str, len) {
            len = len || 2;
            var zeros = new Array(len).join('0');
            return (zeros + str).slice(-len);
        },
        userAdress(User) {
            try {
                let j = JSON.parse(User.address);
                return j.name ?? '';
            } catch (e) {

            }
            return '';
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
        save() {
            this.errors = null;
            this.isNew = this.obj.id === 0;
            this.$refs.form.resetValidation();
            this.obj.birthdate = Helpers.toUnixTime(this.birthDate);
            Api.users.update(this.obj).then((data) => {
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