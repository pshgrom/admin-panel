<template>

  <div v-if="obj">
    <div class="col-auto">
      <q-checkbox
      :disable="ticked.length !== 3"
      v-model="obj.show"
      label='Показывать блок "Актуальные проблемные темы"'
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

      <div class="row cms-main-themes-editor__notice">
        <q-icon
          color="yellow"
          name="info"
          dense
          class="cursor-pointer text-h5"/>

        <div
        v-text="'Должно быть выбрано 3 темы'"
        class="q-ml-sm text-bold"/>
      </div>
    </div>
    <div class="row">

      <theme-selector
      @theme-selector:update="handleTickedUpdate"
      :ticked="ticked"/>

      <div class="col-5 q-ml-md">
        <div class="text-bold">Выбранные темы:</div>
        <div>
          <div
          v-for="(tick, index) in ticked"
          :key="`ticked-${tick.id}`"
          v-text="`${index+1}. ${tick.title}`"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import ThemeSelector from 'components/cms/ThemeSelector';
export default {
  name: "CmsMainThemesEditor",
  props: {
    obj: { type: Object, default: null },
  },
  components: { ThemeSelector },

  data() {
    const selectedThemes = (this.obj && this.obj.themes && this.obj.themes.ticked) ? this.obj.themes.ticked : [];

    return {
      selected: [],
      ticked: selectedThemes,
    }
  },

  methods: {
    handleTickedUpdate(ticked){
      if(ticked.length !== 3){
        this.$q.notify({
          message: 'Должно быть выбрано 3 темы',
          caption: '',
          color: 'yellow',
          duplicates: 'prevent'
        });
        this.obj.show = false;
      }
      this.ticked = ticked;

      this.obj.themes = {};
      this.obj.themes.data = ticked;
      this.obj.themes.ticked = ticked.map(item => item.id);
    },
  },

  computed: { },

  created() {},
}
</script>

<style scoped>
.cms-main-themes-editor__notice {
 margin: 10px;
}
</style>