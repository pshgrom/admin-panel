<template>
   <div>

      <q-btn v-if="photo.media_id===0 && !readonly" color="primary" :label="title + ' - выбрать'"
             @click="dialogTrigger=true"></q-btn>

      <q-card v-if="photo.media_id!==0">
         <q-card-section>
            {{ title }}<br/>
            <q-img :src="photo.url" style="cursor:pointer; height: 140px; max-width: 150px"
                   @click="dialogTrigger=true"/>
         </q-card-section>
      </q-card>


      <q-dialog v-model="dialogTrigger" persistent>
         <q-card style="min-width: 900px">
            <q-card-section class="dialog-header row items-center q-pb-none">
               <div class="text-h6">Выберите картинку</div>
               <q-space/>
               <q-btn icon="close" flat round dense v-close-popup/>
            </q-card-section>
            <q-card-section class="row items-center dialog-body">
               <q-avatar v-for="item in gallery" :key="item.id" :item="item" rounded size="150px"
                         style="margin:5px 5px;cursor:pointer"
                         @click="selectItem(item)">
                  <img :src="itemUrl(item)"/>
               </q-avatar>
            </q-card-section>
            <q-card-section v-if="selItem && selItem.files && false">
               <span style="font-size: 1.2em;font-weight: bold">Файл {{ selItem.id }}</span>
               <q-markup-table dense>
                  <tbody>
                  <tr v-for="file in selItem.files" :key="file.id">
                     <q-td>{{ file.file_type }}</q-td>
                     <q-td align="right">
                        <q-btn dense label="Выбрать" color="primary" v-close-popup @click="useFile(file)"/>
                     </q-td>
                  </tr>
                  </tbody>
               </q-markup-table>
            </q-card-section>

            <q-card-actions align="right" style="border-top: 1px solid #aaa">
               <q-btn v-if="!readonly" flat color="red" label="Убрать картинку" @click="undoPhoto" v-close-popup/>
               <q-btn flat label="Отмена" @click="dialogTrigger=false" v-close-popup/>
            </q-card-actions>
         </q-card>
      </q-dialog>
   </div>
</template>

<script>
export default {
   name: "GalleryPhotoSelect",
   props: ['gallery', 'photo', 'title', 'readonly'],
   emits: ['input', 'commit'],
   data() {
      return {
         dialogTrigger: false,
         selItem: null
      }
   },
   mounted() {

   },
   watch: {},

   methods: {
      selectItem(item) {
         if (this.selItem && this.selItem.id === item.id) {
            this.selItem = null;
            return;
         }
         this.selItem = item;
         if (item.files) {
            for (let i in item.files) {
               if (item.files.hasOwnProperty(i)) {
                  let file = item.files[i];
                  if (file.file_type === 'path') {
                     this.useFile(file);
                     this.dialogTrigger = false;
                  }
               }
            }
         }
      },
      itemUrl(item) {
         let url = this.searchFileType(item, 'thumb_sm');
         if (!url) url = this.searchFileType(item, 'thumb_lg');
         return url ?? 'img/no-photo.svg';
      },
      useFile(file) {
         debugger;
         this.photo.url = CONFIG.SRV_MEDIA_URL + file.path;
         this.photo.media_id = file.id;
         this.photo.media_size = file.file_type;
      },
      searchFileType(item, type) {
         let url = null;
         if (item.files) {
            for (let i in item.files) {
               if (item.files.hasOwnProperty(i)) {
                  let file = item.files[i];
                  if (file.file_type === type) {
                     url = CONFIG.SRV_MEDIA_URL + file.path;
                     break;
                  }
               }
            }
         }
         return url;
      },
      undoPhoto(){
         this.photo.url = '';
         this.photo.media_id = 0;
         this.photo.media_size = null;
      }
   }
}
</script>