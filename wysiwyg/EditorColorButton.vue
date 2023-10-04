<template>
   <q-btn-dropdown dense
                   no-caps
                   ref="color"
                   no-wrap
                   unelevated
                   text-color="black"
                   label="Цвет"
                   size="sm">
      <q-list dense>
         <q-item tag="label"
                 clickable
                 @click="color('backColor', highlight)">
            <q-item-section side>
               <q-icon name="format_color_fill" />
            </q-item-section>
            <q-item-section>
               <q-color v-model="highlight"
                        default-view="palette"
                        no-header
                        no-footer
                        flat
                        class="my-picker"
                        :palette="paletteHighlight" />
            </q-item-section>
         </q-item>
         <q-item tag="label"
                 clickable
                 @click="color('foreColor', foreColor)">
            <q-item-section side>
               <q-icon name="format_color_text" />
            </q-item-section>
            <q-item-section>
               <q-color v-model="foreColor"
                        no-header
                        no-footer
                        default-view="palette"
                        flat
                        class="my-picker"
                        :palette="palletteFore" />
            </q-item-section>
         </q-item>
      </q-list>
   </q-btn-dropdown>
</template>
<style>
   .my-picker {
      max-width: 100px;
      background-color: transparent;
      border-radius: 0;
   }
   .q-color-picker__cube{
      border:1px solid #999;
      height: 22px !important;;
      width: 22px !important;;
   }
</style>
<script>
    export default {
        name: 'EditorColorButton',
        props: {
            reference: {
                type: Object
            }
        },
        data () {
            return {
                paletteHighlight: [
                    '#FFFFFF', '#C3FAF8', '#FEEAC6', '#EBECF0',
                    '#3D3D3D','#777B8C','#BCC1CC','#5566BB','#3AEDE7','#4C648A','#C14466','#FFBB44','#00CC88'
                ],
                palletteFore: ['#FFFFFF', '#3D3D3D','#C3FAF8', '#FEEAC6', '#EBECF0',
                    '#3D3D3D','#777B8C','#BCC1CC','#5566BB','#3AEDE7','#4C648A','#C14466','#FFBB44','#00CC88'],
                foreColor: '#000000',
                highlight: '#FFBB44'
            }
        },
        methods: {
            color (cmd, name) {

                const edit = this.reference;
                this.$refs.color.hide();
                console.log(edit);
                //debugger;

                edit.runCmd(cmd, name);
                edit.focus();
                //чтобы снять выделение и поставить курсор в конец выделения
                setTimeout(function () {
                    window.getSelection().collapseToEnd();
                }, 150);

            }
        }
    }
</script>