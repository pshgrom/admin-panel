<template>
   <q-dialog v-model="showDialog" v-if="this.obj">
      <q-layout view="Lhh lpR fff" container class="bg-white dialog-layout" style="min-width: 1000px;width: 1000px;max-height: 400px">
         <q-header bordered>
            <q-toolbar>
               <q-toolbar-title v-html="dialogTitle"></q-toolbar-title>
               <q-btn flat v-close-popup round dense icon="close" @click="cancelEdit"/>
            </q-toolbar>
         </q-header>

         <q-footer bordered>
            <custom-button title="Отмена" type="light" @click="cancelEdit" />
            <custom-button v-if="!readonly" :title="obj.facility_id===0?'Создать':'Сохранить'" type="purple" @click="save" />
         </q-footer>


         <q-page-container>
            <q-page padding>
               <q-form ref="form" @submit="save" class="user-form" dense>
                  <div class="row form-row" v-if="obj.facility_id>0">
                     <div class="col-2">
                        <q-field outlined label="ID" stack-label dense>
                           <template v-slot:control>
                              <div class="self-center full-width no-outline" tabindex="0">{{ obj.facility_id }}</div>
                           </template>
                        </q-field>
                     </div>
                     <div class="col-10">
                        <q-field outlined label="Название" stack-label dense>
                           <template v-slot:control>
                              <div class="self-center full-width no-outline" tabindex="0">{{ obj.facility_name }}</div>
                           </template>
                        </q-field>
                     </div>

                  </div>


                  <div class="row form-row">
                     <div class="col-12">
                        <pdo-organization-select
                        v-model="obj.owner_id"
                        :readonly="readonly"
                        required :showPosOrg="true"
                        outlined ></pdo-organization-select>
                     </div>
                  </div>
               </q-form>
            </q-page>
         </q-page-container>
      </q-layout>
   </q-dialog>
</template>
<style>


</style>
<script>
import {defineComponent} from 'vue';
import Helpers from 'src/lib/api/helpers';
import PdoApi from 'src/lib/pdo/api';
import PdoOrganizationSelect from "components/pdo/PdoOrganizationSelect";
import CustomButton from 'src/components/CustomButton';

export default defineComponent({
   name: "FacilityOwnerDialog",
   props: {
     obj: { type: Object, required: false },
     readonly: { type: Boolean, default: false }
   },
   emits: ['saved', 'cancel'],
   components: { PdoOrganizationSelect, CustomButton },
   computed: {
      showDialog() {
         return this.obj != null;
      },
      dialogTitle() {
         if (!this.obj) return '';
         return this.obj.facility_name;
      },
      editable() {
         return true;
      }
   },
   watch: {
      obj() {
         document.obj = this.obj;
      }
   },
   data() {
      return {
         isNew: false,
         errors: null
      };
   },
   methods: {
      validationErrors(name) {
         let res = true;
         if (this.errors && this.errors[name] && Array.isArray(this.errors[name])) res = this.errors[name].join(', ');
         return res;
      },
      unixDate: Helpers.formatUnixDate,
      validateAll() {
         //подсвечивает ошибки на всех полях
         this.$refs.form.getValidationComponents().forEach((comp) => comp.validate());
      },
      cancelEdit() {
         this.$emit('cancel');
      },
      save(stay) {
         this.errors = null;
         this.isNew = this.obj.id === 0;
         this.$refs.form.resetValidation();
         PdoApi.owner.save(this.obj).then((data) => {
            if (data._errors) {
               this.errors = data._errors;
               this.validateAll();
            } else {
               this.$q.notify({
                  message: 'Сохранено',
                  caption: '',
                  color: 'green'
               });
               this.$emit('saved', {obj: data, append: this.isNew, stay: typeof stay == 'boolean' ? stay : false});
            }
         });

      },
      ...Helpers
   }

});
</script>