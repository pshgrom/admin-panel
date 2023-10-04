<template>
    <span>
    <q-btn dense no-caps
           ref="color"
           no-wrap
           unelevated
           text-color="black"
           label="Фото"
           size="sm" icon="photo" @click="imageDialog">

    </q-btn>
        <q-dialog v-model="showDialog" @escape-key="showDialog=false">
            <q-card style="min-width: 600px;width: 600px">
            <q-form ref="form">
                <q-card-section class="dialog-header row items-center q-pb-none">
                    <div class="text-h6">Добавление фотографии</div>
                    <q-space/>
                    <q-btn icon="close" flat round dense v-close-popup @click="showDialog=false"/>
                </q-card-section>
                <q-card-section class="q-pt-none">
                    <div class="row">
                        <div class="col-12">
                             <q-uploader
                                 extensions=".jpg, .png"
                                 :url="uploadUrl"
                                 field-name="file"
                                 @uploaded="uploaded"
                                 ref="uploader"
                                 label="Загрузить фото"
                                 auto-expand
                                 style="width: 100%"
                                  stack-label="Загрузить фото"
                                 auto-upload
                             />
                        </div>
                    </div>
                </q-card-section>
            </q-form>
            </q-card>
        </q-dialog>
        </span>
</template>
<script>
import Api from 'src/lib/special/api';
import UI from 'src/lib/ui/objects';
import Services from "src/lib/const/services";

export default {
    name: 'EditorImgButton',
    props: {
        reference: {
            type: Object
        },
        obj_tp: {
            type: String,
            default: ''
        },
        obj_id: {
            type: Number,
            default: 0
        }
    },
    emits: ['changed'],
    computed:{
        uploadUrl(){
            return CONFIG.API_URLS.gorod_spec+'media/addphoto';
        }
    },
    data() {
        return {
            showDialog: false
        }
    },
    methods: {
        imageDialog() {
            var selection = window.getSelection();
            var container = selection.anchorNode;
            if (container == null){
                this.$q.notify({
                    message: 'Установите курсор на место вставки фото',
                    color: 'red'
                });
                return;
            }
            this.showDialog = true;
        },
        uploadFactory(file) {
            Api.media.addFile(file[0], this.obj_tp, this.obj_id).then(data => {
                console.log(data);
                if (data.id) {
                    var addUrl = '';
                    if (document.location.href.indexOf('localhost')>0) addUrl = CONFIG.SRV_BASE_URL.substr(0, CONFIG.SRV_BASE_URL.length-1);
                    this.addImage(addUrl+'/api/media/file/link?id=' + data.id + '&type=path_jpg');
                } else {
                    this.$q.notify({
                        message: data,
                        color: 'red'
                    });
                }
            });
        },
        uploaded(info){
            try {
                const response = JSON.parse(info.xhr.response);
                if (response.status && response.status==='error'){
                    this.$q.notify({
                        message: response.message,
                        caption: '',
                        color: 'red'
                    });
                    return;
                }
                let ddd = JSON.parse(info.files[0].xhr.response)
                let id = ddd.data.id;
                var addUrl = '';
                if (document.location.href.indexOf('localhost')>0) addUrl = CONFIG.SRV_BASE_URL.substr(0, CONFIG.SRV_BASE_URL.length-1);
                this.addImage(addUrl+'/api/media/file/link?id=' + id + '&type=path_jpg');
            }catch (e){
                this.$q.notify({
                    message: 'Ошибка '+JSON.stringify(e),
                    caption: '',
                    color: 'red'
                });
            }
        },
        addImage(url) {
            this.$refs.uploader.reset();
            this.showDialog = false;
            const edit = this.reference;
            edit.focus();
            const html = '<img src="' + url + '" class="news-image">';
            edit.runCmd('insertHtml', html);
            //this.pasteHtmlAtCaret(html);
            // const img = document.createElement('img');
            // img.src = url;
            // img.width = '100%';
            // let node = window.getSelection().focusNode;
            // if (node.nodeName == '#text') {
            //     node = node.nextSibling;
            // }
            // if (node.className.indexOf('q-editor__content') >= -1) {
            //     node.appendChild(img);
            // } else {
            //     node.parentElement.appendChild(img);
            // }

            edit.focus();
            //чтобы снять выделение и поставить курсор в конец выделения
            setTimeout( ()=> {
                window.getSelection().collapseToEnd();
                this.$emit('changed', true);
            }, 150);

        },
        pasteHtmlAtCaret: UI.pasteHtmlAtCaret

    }
}
</script>