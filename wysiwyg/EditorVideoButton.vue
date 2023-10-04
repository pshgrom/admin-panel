<template>
    <span>
    <q-btn dense no-caps
           ref="color"
           no-wrap
           unelevated
           text-color="black"
           label="Видео"
           size="sm" icon="movie" @click="startDialog">

    </q-btn>
        <q-dialog v-model="showDialog" @escape-key="showDialog=false">
            <q-card style="min-width: 600px;width: 600px">
            <q-form ref="form">
                <q-card-section class="dialog-header row items-center q-pb-none">
                    <div class="text-h6">Добавление видео</div>
                    <q-space/>
                    <q-btn icon="close" flat round dense v-close-popup @click="showDialog=false"/>
                </q-card-section>
                <q-card-section class="q-pt-none">
                    <div class="row">
                        <div class="col-12">
                            <q-input dense outlined v-model="videoUrl" label="Ссылка на видео"></q-input>
                        </div>
                    </div>
                </q-card-section>
                <q-card-actions class="bg-white text-primary justify-end">
                    <q-btn label="Отмена" @click="showDialog=false" outline/>
                    <q-btn label="Сохранить" @click="addVideo" color="primary"/>
                </q-card-actions>
            </q-form>
            </q-card>
        </q-dialog>
        </span>
</template>
<script>
import UI from 'src/lib/ui/objects';

export default {
    name: 'EditorVideoButton',
    props: {
        reference: {
            type: Object
        }
    },
    emits: ['changed'],
    data() {
        return {
            showDialog: false,
            videoUrl: ''
        }
    },
    methods: {
        startDialog() {
            var selection = window.getSelection();
            var container = selection.anchorNode;
            if (container == null){
                this.$q.notify({
                    message: 'Установите курсор на место вставки видео',
                    color: 'red'
                });
                return;
            }
            this.showDialog = true;
        },
        addVideo() {
            const edit = this.reference;
            edit.focus();
            let url = this.videoUrl.replace('watch?v=', 'embed/');
            //https://www.youtube.com/embed/417zx9XYujc
            let iframe = '<div class="embeded_video"><iframe class="news-video" src="'+url+'"' +
                ' width="560" height="315" frameborder="0" \n' +
                ' allowfullscreen="allowfullscreen"></iframe></div>';
            //iframe = '<div>HELLO</div>';
            //const html = '<div class="videoWrapper">'+iframe+'</div>';
            //this.pasteHtmlAtCaret(iframe);
            edit.runCmd('insertHtml', iframe);
            edit.focus();
            //чтобы снять выделение и поставить курсор в конец выделения
            setTimeout( ()=> {
                window.getSelection().collapseToEnd();
                this.$emit('changed', true);
            }, 150);
            this.showDialog = false;
            this.videoUrl = '';
        },
        insertNodeOverSelection(node, containerNode) {
            var sel, range, html;
            if (window.getSelection) {
                sel = window.getSelection();
                if (sel.getRangeAt && sel.rangeCount) {
                    range = sel.getRangeAt(0);
                    if (isOrContainsNode(containerNode, range.commonAncestorContainer)) {
                        range.deleteContents();
                        range.insertNode(node);
                    } else {
                        containerNode.appendChild(node);
                    }
                }
            } else if (document.selection && document.selection.createRange) {
                range = document.selection.createRange();
                if (isOrContainsNode(containerNode, range.parentElement())) {
                    html = (node.nodeType == 3) ? node.data : node.outerHTML;
                    range.pasteHTML(html);
                } else {
                    containerNode.appendChild(node);
                }
            }
        },
        pasteHtmlAtCaret: UI.pasteHtmlAtCaret
    }
}
</script>