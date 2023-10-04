<template>
  <div class="col-auto" v-if="obj">
    <div class="flex">
        <q-checkbox
        v-model="obj.show"
        label='Показывать блок "Новое на портале"'
        class="text-bold text-h6"/>
        <q-icon
        name="help"
        color="primary"
        dense
        class="q-ma-sm cursor-pointer text-h5">
          <q-tooltip class="text-body2">
            Отображение блока на главной странице портала
          </q-tooltip>
        </q-icon>
      </div>

      <div class="q-ma-sm flex">
        <q-input
        v-model="obj.text"
        type="textarea"
        label="Текст"
        maxlength="150"
        dense
        autogrow
        class="cms-main-actual-editor__input"/>
      </div>

      <div class="flex">
        <q-checkbox
        v-model="obj.isColorActive"
        label="Изменить цвет текста"/>

        <q-icon
        name="help"
        color="primary"
        dense
        class="q-pa-sm cursor-pointer text-h5">
          <q-tooltip class="text-body2">
            Данный цвет текста будет отображаться независимо от выбранной темы
          </q-tooltip>
        </q-icon>

        <q-input
        :disable="!obj.isColorActive"
        label="Цвет текста"
        v-model="obj.color"
        class="my-input"
        dense>
          <template v-slot:append>
            <q-icon name="colorize" class="cursor-pointer">
              <q-popup-proxy
              transition-show="scale"
              transition-hide="scale">
                <q-color v-model="obj.color"/>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>

        <div
        v-text="'Пример текста'"
        :style="`color: ${obj.color}`"
        class="q-pa-sm cms-main-actual-editor__example"/>
      </div>

    <div class="q-ma-sm flex flex-lg">
      <q-input
      v-model="obj.link"
      label='Ссылка "Подробнее"'
      dense
      class="cms-main-actual-editor__input"/>
    </div>

    </div>
</template>

<script>

export default {
  name: "CmsMainActualEditor",
  props: ['obj'],
  components: {},
  data() {
    return {

    }
  },
  watch: {},
  created() {

  },

  methods: {

  }
}
</script>

<style scoped>
.cms-main-actual-editor__input {
  min-width: 1200px;
}

.cms-main-actual-editor__example {
  font-size: 1rem;
  font-weight: 400;
}
</style>
