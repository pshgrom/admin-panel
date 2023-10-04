<template>
   <q-card class="my-card full-width" flat bordered>
      <q-card-section class="full-width">


         <q-card-section>
            <div class="col-12 text-bold">
               {{item.name}}
            </div>
            <div class="col-12" v-html="item.html">

            </div>
         </q-card-section>
         <q-card-actions align="right" v-if="!readonly">
            <q-btn dense flat icon="delete_forever" @click="dropItem" label="Удалить"></q-btn>
            <q-btn dense flat icon="edit" @click="editItem" label="Редактировать"></q-btn>
         </q-card-actions>


      </q-card-section>
      <q-dialog v-model="showEdit">
         <q-card style="min-width: 1000px;width: 1000px">
            <q-card-section class="dialog-header row items-center q-pb-none">
               <div class="text-h6">Редактирование раздела</div>
               <q-space/>
               <q-btn icon="close" flat round dense v-close-popup/>
            </q-card-section>

            <q-card-section class="q-pt-none">
               <div class="row">
                  <div class="col-12">
                     <q-input outlined label="Название раздела" v-model="name"></q-input>
                  </div>
                  <div class="col-12">
                     <q-editor
                        ref="editor2"
                        v-model="html"
                        :dense="$q.screen.lt.md"
                        :toolbar="editorToolbar"
                        :fonts="fonts"
                     >
                        <template v-slot:token>
                           <editor-color-button :reference="$refs.editor2"/>
                        </template>
                        <template v-slot:sublist>
                           <sublist-button :reference="$refs.editor2"></sublist-button>
                        </template>
                     </q-editor>
                  </div>
               </div>
            </q-card-section>

            <q-card-actions class="bg-white text-primary justify-end">
               <q-btn label="Отмена" type="submit" @click="showEdit=false"/>
               <q-btn label="Сохранить" type="submit" color="primary" @click="saveItem"/>
            </q-card-actions>
         </q-card>
      </q-dialog>
   </q-card>

</template>
<script>
    import moment from "moment";
    import "moment/locale/ru";
    import {defineComponent} from 'vue';
    import Api from 'src/lib/api/admin-api';
    import Helpers from 'src/lib/api/helpers';
    import state from "src/lib/state";
    import EditorColorButton from 'components/wysiwyg/EditorColorButton';
    import SublistButton from "components/wysiwyg/SublistButton";

    export default defineComponent({
        name: "RuleItemEditor",
        props: {
          item: { type: Object, required: true },
          readonly: { type: Boolean, default: false }
        },
        emits: ['delete'],
        components: {SublistButton, EditorColorButton},
        computed: {},
        watch: {},
        created() {
        },
        data() {
            return {
                showEdit: false,
                editorToolbar: state.editorToolbar(this.$q),
                fonts: state.editorFonts,
                name:'',
                html:''
            };
        },
        methods: {
            editItem() {
                this.name = this.item.name;
                this.html = this.item.html;
                this.showEdit=true;
            },
            saveItem(){
                this.item.name = this.name;
                this.item.html = this.html;
                this.showEdit=false;
            },
            dropItem() {
                this.$emit('delete', this.item);
            },
        }

    });
</script>
<style>
   .myForm div.row > div {
      padding-left: 10px;
   }
</style>