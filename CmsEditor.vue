<template>
   <div class="row">
      <!--    toolbar-->
             <q-toolbar class="shadow-2 rounded-borders col-12" dense>

         <q-toolbar-title style="font-size: 1em">{{ title }}</q-toolbar-title>
         <q-space/>
         <q-separator dark vertical/>
         <q-btn-dropdown stretch flat label="Версии">
            <q-list>
               <q-item v-for="n in revs" :key="n.id" clickable v-close-popup tabindex="0" @click="loadItem(n.rev)">
                  <q-item-section>
                     <q-item-label>{{ n.rev }} {{ n.published ? 'Опубликована' : '' }}</q-item-label>
                     <q-item-label caption>{{ formatUnixDate(n.updated_at ?? n.created_at, true) }}</q-item-label>
                  </q-item-section>
               </q-item>
            </q-list>
         </q-btn-dropdown>
         <q-separator dark vertical/>
         <q-btn
         @click="saveObj"
         :disable="!this.isValid"
         label="Сохранить"
         flat
         class="bg-primary text-white q-mr-sm"/>


         <q-btn class="bg-secondary  text-white" flat label="Опубликовать" @click="publishDialogOpen = true" v-if="!obj.published"/>
         <q-separator dark vertical/>
         <q-toggle v-model="showGallery" label="Галерея" color="primary"/>
      </q-toolbar>
      <q-btn @click="openResetDialog" v-if="isCurrentPage"  style="background: #FF9D01; color: white; margin: 10px 0;text-transform:none" :label="titlelabelPage"/>
       <q-separator dark vertical/>
       <q-separator dark vertical v-if="!obj.p"/>
      <gallery-editor :obj="obj" v-show="showGallery"></gallery-editor>
      <div class="col-12" style="margin-top: 20px">
         <cms-html-editor
         v-if="obj.editor_type===0"
         :obj="obj"/>

         <cms-faq-editor
         v-if="obj.editor_type === 1"
         :obj="obj"/>

         <cms-tellfriends-editor
         v-if="obj.editor_type === 2"
         :obj="obj"/>

         <cms-main-gallery-editor
         v-if="obj.editor_type === 5 && add_code == 'mainpic'"
         :obj="obj.json.mainpic"/>

         <cms-main-banner-editor
         v-if="obj.editor_type === 5 && add_code == 'banner'"
         :obj="obj.json.banner"/>

         <cms-main-actual-editor
         v-if="obj.editor_type === 5 && add_code == 'actual'"
         :obj="obj.json.actual"/>

        <cms-main-theme-decoration-editor
        v-if="obj.editor_type === 5 && add_code == 'decoration'"
        @validate="handleValidateResult"
        :obj="obj.json.decoration"/>

        <cms-main-themes-editor
        v-if="obj.editor_type === 5 && add_code == 'themes'"
        :obj="obj.json.themes"/>

      </div>
      <custom-dialog  title="Предупреждение" :trigger="saveDialogOpen" @input="saveDialogOpen = $event" :buttons="dialogButtons">
         <span>Эта версия опубликована на портале. При сохранении будет создана новая неопубликованная версия, если вы хотите, чтобы она появилась на портале, после сохранения вам нужно будет её опубликовать</span>
      </custom-dialog>
      <custom-dialog  title="Предупреждение" :trigger="publishDialogOpen" @input="publishDialogOpen = $event" :buttons="dialogButtons">
         <span>Просмотрите как выглядит текст на портале перед публикацией. </span>
         <span>Вы уверены, что хотите опубликовать страницу?</span>
      </custom-dialog>
      <custom-dialog  title="Внимание" :trigger="agreementDialogOpen" @input="agreementDialogOpen = $event" :buttons="dialogButtons">
         <span>Вы действительно хотите сбросить признак согласия с "{{ agreementSign }}" для всех пользователей портала?</span>
      </custom-dialog>
   </div>
</template>

<script>
import {ref} from 'vue';
import Api from 'src/lib/api/admin-api';
import Helpers from 'src/lib/api/helpers';
import CmsHtmlEditor from "./CmsHtmlEditor";
import state from 'src/lib/state';
import GalleryEditor from "./GalleryEditor";
import CmsFaqEditor from "./CmsFaqEditor";
import CmsTellfriendsEditor from "./cms/CmsTellfriendsEditor";
import CmsMainGalleryEditor from "./cms/CmsMainGalleryEditor";
import CmsMainBannerEditor from "./cms/CmsMainBannerEditor";
import CmsMainActualEditor from "./cms/CmsMainActualEditor";
import CmsMainThemeDecorationEditor from "./cms/CmsMainThemeDecorationEditor";
import CmsMainThemesEditor from "./cms/CmsMainThemesEditor";
import CustomDialog from './CustomDialog';

export default {
   name: "CmsEditor",
   components: {
      CmsMainGalleryEditor, CmsTellfriendsEditor, CmsFaqEditor, GalleryEditor, CmsHtmlEditor,
      CmsMainBannerEditor, CmsMainActualEditor, CmsMainThemeDecorationEditor, CmsMainThemesEditor, CustomDialog,
   },
   props: ['code', 'add_code','page'],
   data() {
      return {
         obj: ref({published: true, rev: 0, id: 0}),
         revs: ref([]),
         title: ref(''),
         showGallery: ref(false),
         labelPage:ref(''),
         saveDialogOpen: false,
         publishDialogOpen: false,
         agreementDialogOpen: false,
         agreementSign: '',
         isValid: true,
      }
   },
   watch: {
      code: function () {
         this.loadItem();
      }
   },
   created() {
      this.loadItem();
   },
    computed: {
      portal_link() {
         if (this.code === 'main') {
            return CONFIG.PORTAL_URL;
         }
         if (this.code === 'about.doc.moderation') {
            return CONFIG.PORTAL_URL + 'portal/documents';
         }
         if (this.code === 'about.about.tell_friends') {
            return CONFIG.PORTAL_URL + 'portal/about/share';
         }
         if (this.code === 'about.about.faq') {
            return CONFIG.PORTAL_URL + 'portal/about/faq';
         }
         if (this.code === 'about.doc.info_processing_rules') {
            return CONFIG.PORTAL_URL + 'portal/documents/reglament';
         }
         if (this.code === 'about.doc.user_agreement') {
            return CONFIG.PORTAL_URL + 'portal/documents/agreement';
         }
         return '#' + this.code + '___' + this.add_code;
      },
      isCurrentPage(){
          if(this.code==='about.doc.moderation' || this.code==='about.doc.user_agreement' || this.code==='about.doc.info_processing_rules'){
              return true
          }
      },
      titlelabelPage(){
        return {
          'about.doc.moderation': `Сбросить признак согласия с "Едиными правилами модерации"`,
          'about.doc.user_agreement': `Сбросить признак согласия с "Пользовательским соглашением"`,
          'about.doc.info_processing_rules': `Сбросить признак согласия с "Регламентом обработки информации"`
        }[this.code];
      },
      dialogButtons() {
         let action = null;
         if (this.saveDialogOpen) {
            action = this.saveObjConfirm;
         } else if (this.publishDialogOpen) {
            action = this.publishObj;
         } else if (this.agreementDialogOpen) {
            action = this.reset;
         }
         return [
            {
               title: 'Отмена',
               type: 'light',
            },
            {
               title: 'Ок',
               type: 'purple',
               action,
            },
         ];
      },
   },

   methods: {
      setRes(data) {
         var me = this;
         if (typeof data.object.json === 'string') {
            data.object.json = JSON.parse(data.object.json);
         }
         if (!data.object.json.gallery) data.object.json.gallery = [];
         if ((data.object.editor_type === 1 || data.object.editor_type === 2) && !data.object.json.items) data.object.json.items = [];
         if (data.object.editor_type === 5) {
            if (!data.object.json.mainpic) data.object.json.mainpic = [];
            if (!data.object.json.banner) data.object.json.banner = [];
            if (!data.object.json.decoration)
              data.object.json.decoration = { themes: {
                  lightTheme: {
                    from: ref('00:00'),
                    till: ref('00:00'),
                    title: 'Светлая тема',
                    radioValue: 'light',
                    isMainTheme: true,
                    isActive: true,
                  },
                  darkTheme: {
                    from: ref('23:00'),
                    till: ref('06:00'),
                    title: 'Темная тема',
                    radioValue: 'dark',
                    isMainTheme: false,
                    isActive: true,
                  }
                }};
            if (!data.object.json.actual) data.object.json.actual = {show: false, text: '', color: '#F00'}

           if (!data.object.json.themes)
             data.object.json.themes = { show: false, themes: []};

         }
         me.obj = data.object;
         me.obj.last_edit_by = state.User.id;
         me.revs = data.revs;
         me.title = 'Версия ' + me.obj.rev + ' от ' + Helpers.formatUnixDate(me.obj.updated_at ?? me.obj.created_at, true);
         if (me.obj.published) {
            me.title += ' (опубликована)';
         }
      },
      loadItem(rev) {
         rev = rev ?? 0;
         var me = this;
         Api.cms.get(me.code, rev).then((data) => {
            me.setRes(data);
         });
      },
      openResetDialog() {
        this.agreementSign = {
          'about.doc.moderation': 'Едиными правилами модерации',
          'about.doc.user_agreement': 'Пользовательским соглашением',
          'about.doc.info_processing_rules': 'Регламентом обработки информации'
        }[this.code];
         this.agreementDialogOpen = true;
      },
       reset() {
         let func = {
           'about.doc.moderation': 'resetRules',
           'about.doc.user_agreement': 'resetAgreement',
           'about.doc.info_processing_rules': 'resetReglament'
         }[this.code];

         Api.users[func]().then(date => {
           if (date){
             this.$q.notify({
               message: 'Успешно',
               color: 'green'
             });
           } else {
             this.$q.notify({
               message: data,
               color: 'red'
             });
           }
         });

         this.agreementDialogOpen = false;
       },
      publishObj() {
         Api.cms.publish(this.obj).then((data) => {
            if (data.object) {
               this.setRes(data);
               this.$q.notify({
                  message: 'Опубликовано',
                  color: 'primary'
               });
            } else {
               this.$q.notify({
                  message: data,
                  color: 'red'
               });
            }
         });
         this.publishDialogOpen = false;
      },
      saveObj() {
         if (this.obj.published) {
            this.saveDialogOpen = true;
         } else {
            this.saveObjConfirm();
         }

      },
      saveObjConfirm() {
         Api.cms.update(this.obj).then((data) => {
            if (data.object) {
               this.setRes(data);
               this.$q.notify({
                  message: 'Сохранено',
                  color: 'primary'
               });
            } else {
               this.$q.notify({
                  message: data,
                  color: 'red'
               });
            }

         });
         this.saveDialogOpen = false;
      },

      handleValidateResult(isValid){
        this.isValid = isValid;
      },

      ...Helpers

   }
}
</script>

<style >

</style>