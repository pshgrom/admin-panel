<template>
   <div class="row">
      <div class="col-12">
<!--         <q-banner class="bg-warning" style="margin-bottom: 20px">-->
<!--            Прежде чем публиковать текст, исправленный в редакторе, проверьте как он будет выглядеть на сайте.-->
<!--            Если ломаются шрифты или дизайн, необходимо исправить HTML код.-->
<!--         </q-banner>-->
          <q-item-label class="q-pb-xs">Текст страницы</q-item-label>
         <q-editor class="editor_doc"
            ref="editor"
            v-model="obj.html"
            :dense="$q.screen.lt.md"
            :toolbar="editorToolbar"
            :fonts="fonts"
            label="Текст страницы"
         >
            <template v-slot:token>
               <editor-color-button :reference="$refs.editor"/>
            </template>
             <template v-slot:imgadd>
                 <editor-img-button obj_tp="news" :obj_id="obj.id"
                                    :reference="$refs.editor"
                                    @changed="onEditorChanged($refs.editor)"/>
             </template>
             <template v-slot:videoadd>
                 <editor-video-button :reference="$refs.editor"
                                      @changed="onEditorChanged($refs.editor)"/>
             </template>
             <template v-slot:fscreen>
                 <editor-full-screen :reference="$refs.editor"/>
             </template>
         </q-editor>
      </div>
   </div>
</template>

<script>
    import {ref} from 'vue';
    import Api from 'src/lib/api/admin-api';
    import state from "src/lib/state";
    import Helpers from 'src/lib/api/helpers';
    import EditorColorButton from 'components/wysiwyg/EditorColorButton';
    import UI from "src/lib/ui/objects";
    import EditorImgButton from "components/wysiwyg/EditorImgButton";
    import EditorVideoButton from "components/wysiwyg/EditorVideoButton";
    import EditorFullScreen from "components/wysiwyg/EditorFullScreen";
    export default {
        name: "CmsHtmlEditor",
        props: ['obj'],
        components: {EditorColorButton,EditorImgButton,EditorVideoButton,EditorFullScreen},
        data() {
            return {
                editorToolbar: state.editorToolbar(this.$q),
                newsDate: null,
                dense: true,
                dpLocale: state.datePickerLocale,
                loading: true,
                fonts: state.editorFonts,
            }
        },
        watch: {

        },
        created() {

        },


        methods: {
            ...Helpers,
            onEditorChanged(editor) {
                UI.onEditorResize(editor, true);
            },
        onEditorResize() {
            UI.onEditorResize(this.$refs.editor);
        }
        }
    }
</script>

<style>
.videoWrapper {
    position: relative;
    padding-bottom: 56.25%; /* 16:9 */
    height: 0;
}
.videoWrapper iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}


</style>