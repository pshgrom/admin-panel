<template>
  <q-card class="my-card full-width" flat bordered>
    <q-card-section horizontal class="full-width">
      <q-card-section style="width:100%">

        <div class="flex">
          <div class="col-auto">
            <q-form
            ref="form"
            no-error-focus>

              <div class="col-auto">

                <div class="flex justify-center">
                  <div
                  v-text="'Выбор основной темы'"
                  class="q-pa-sm text-bold"/>

                  <q-icon
                  name="help"
                  color="primary"
                  dense
                  class="q-pa-sm cursor-pointer text-h5">
                    <q-tooltip class="text-body2">
                      Основная тема применяется всегда, кроме времени, установленного для другой темы (если эта тема активна)
                    </q-tooltip>
                  </q-icon>
                </div>

                <div class="flex justify-between">
                  <div
                  v-text="'Светлая тема'"
                  class="q-ma-md text-bold"/>

                  <q-btn-toggle
                  v-model="mainTheme"
                  :options="[{label: '', value: 'light'}, {label: '', value: 'dark'}]"
                  toggle-color="primary"
                  color="white"
                  text-color="primary"
                  spread
                  no-caps
                  rounded
                  unelevated
                  class="q-ma-sm cms-main-theme-decoration-editor__toggle"/>

                  <div
                  v-text="'Темная тема'"
                  class="q-ma-md text-bold"/>
                </div>

              </div>

              <div class="row">

                <div
                v-for="(theme, themeKey) in obj.themes"
                :key="theme.title"
                class="col-auto">

                  <div class="q-pa-sm col-auto">
                    <q-input
                    @update:model-value="val => handleTimeInput(themeKey, val, theme.till)"
                    v-model="theme.from"
                    :disable="theme.isMainTheme || !theme.isActive"
                    :error-message="'Некорректная временная граница'"
                    :rules="['time']"
                    mask="time"
                    label="С"
                    filled
                    class="q-mb-sm">
                      <template v-slot:append>
                        <q-icon name="access_time" class="cursor-pointer">
                          <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                            <q-time v-model="theme.from">
                              <div class="items-center justify-end">
                                <q-btn v-close-popup label="Закрыть" color="primary" flat />
                              </div>
                            </q-time>
                          </q-popup-proxy>
                        </q-icon>
                      </template>
                    </q-input>

                    <q-input
                    @update:model-value="val => handleTimeInput(themeKey, theme.from, val)"
                    v-model="theme.till"
                    :disable="theme.isMainTheme || !theme.isActive"
                    :error-message="'Некорректная временная граница'"
                    :rules="['time']"
                    mask="time"
                    label="По"
                    filled>
                      <template v-slot:append>
                        <q-icon name="access_time" class="cursor-pointer">
                          <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                            <q-time v-model="theme.till">
                              <div class="items-center justify-end">
                                <q-btn v-close-popup label="Закрыть" color="primary" flat />
                              </div>
                            </q-time>
                          </q-popup-proxy>
                        </q-icon>
                      </template>
                    </q-input>
                  </div>

                  <div class="col-2">
                    <q-checkbox
                    v-model="theme.isActive"
                    :disable="theme.isMainTheme"
                    label="Тема активна"/>

                    <q-icon
                    name="help"
                    color="primary"
                    dense
                    class="q-pa-sm cursor-pointer text-h5">
                      <q-tooltip class="text-body2">
                        Если тема неактивна, то основная тема применяется на постоянной основе
                      </q-tooltip>
                    </q-icon>
                  </div>

                </div>

              </div>
            </q-form>
          </div>

          <div class="col-auto q-pa-sm">
            <q-btn
            @click="resetTime"
            flat
            label="Сбросить изменения"
            class="bg-primary text-white"/>
          </div>

        </div>

      </q-card-section>
    </q-card-section>
  </q-card>

</template>
<script>

import "moment/locale/ru";
import {defineComponent, ref} from 'vue';
import { cloneDeep } from "lodash";

export default defineComponent({
  name: "CmsMainThemeDecorationEditor",
  props: {
    obj: { type: Object, default: {} },
  },

  data() {
    return {
      decoration: cloneDeep(this.obj.themes),
      decorationData: {
        lightTheme: {
          label: 'Светлая тема',
          value: 'light',
          from: ref('00:00'),
          till: ref('00:00'),
          tillNextDay: false,
          title: 'Светлая тема',
          radioValue: 'light',
          isMainTheme: true,
          isActive: true,
        },
        darkTheme: {
          label: 'Темная тема',
          value: 'dark',
          from: ref('23:00'),
          till: ref('06:00'),
          tillNextDay: true,
          title: 'Темная тема',
          radioValue: 'dark',
          isMainTheme: false,
          isActive: true,
        }
      },
      hasError: false,
      hasFormError: false,
      mainTheme: 'light',
    };
  },

  watch: {
    'obj.themes': {
      deep: true,
      handler() {
        setTimeout(this.validateForm, 100);
      }
    },
    mainTheme: {
      handler() {
        this.handleMainThemeChange();
      }
    }
  },
  created() {
    if (!this.obj.themes) {
      this.obj.themes = cloneDeep(this.decorationData);
      this.decoration = cloneDeep(this.obj.themes);
    }

    this.setMainTheme();
  },

  methods: {
    handleMainThemeChange() {
      if (this.mainTheme == 'light') {
        this.obj.themes.lightTheme.isMainTheme = true;
        this.obj.themes.lightTheme.isActive = true;

        this.obj.themes.lightTheme.from = '00:00';
        this.obj.themes.lightTheme.till = '00:00';

        this.obj.themes.darkTheme.isMainTheme = false;
      } else {
        this.obj.themes.darkTheme.isMainTheme = true;
        this.obj.themes.darkTheme.isActive = true;


        this.obj.themes.darkTheme.from = '00:00';
        this.obj.themes.darkTheme.till = '00:00';

        this.obj.themes.lightTheme.isMainTheme = false;
      }
    },

    validateForm() {
      if (this.$refs.form) {
        this.$refs.form.validate().then(valid => {
          this.hasFormError = !valid;
          this.$emit('validate', valid);
        });
      }
    },

    fromGreaterThanTill(from, till) {
      const fromDate = new Date();
      fromDate.setHours(from.split(':')[0]);
      fromDate.setMinutes(from.split(':')[1]);

      const tillDate = new Date();
      tillDate.setHours(till.split(':')[0]);
      tillDate.setMinutes(till.split(':')[1]);

      return fromDate.getTime() > tillDate.getTime();
    },

    resetTime() {
      this.obj.themes = cloneDeep(this.decoration);
      this.setMainTheme();
      this.$q.notify({
        message: 'Изменения успешно сброшены',
        color: 'primary'
      });
    },

    setMainTheme() {
      this.mainTheme = this.obj.themes.lightTheme.isMainTheme
      ? this.obj.themes.lightTheme.radioValue
      : this.obj.themes.darkTheme.radioValue;
    },

    handleTimeInput(theme, from, till) {
       this.obj.themes[theme].tillNextDay = this.fromGreaterThanTill(from, till);
    }
  }

});
</script>

<style scoped>
.cms-main-theme-decoration-editor__toggle {
  border: 1px solid #8c7ace;
  flex: 1 1 auto;
}
</style>
