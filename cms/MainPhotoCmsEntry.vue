<template>
   <q-card class="my-card full-width" flat bordered>
      <q-card-section horizontal class="full-width">
         <q-card-section>
            <q-banner rounded class="bg-purple-8 text-white full-width text-center" style="height: 180px;width: 250px;min-width: 250px"
                      v-if="item.url===''">
               Добавьте фото
            </q-banner>
            <q-img :src="item.url" style="width: 250px;max-height: 150px" v-if="item.url!==''">
               <template v-slot:error>
                  <div class="absolute-full flex flex-center bg-negative text-white">
                     <div>Фото не загружается<br/><a :href="item.url"
                                                     style="color: white;text-decoration: underline;white-space: normal;word-break:break-all"
                                                     target="_blank">Ссылка на фото</a></div>
                  </div>
               </template>
            </q-img>
         </q-card-section>
         <q-card-section style="width:100%">
            <div class="row">
               <div class="col-12">
                  <q-input v-model="item.url" dense label="URL фото"></q-input>
               </div>
            </div>
            <div class="row">
               <div class="col-2">
                  <q-checkbox v-model="item.use_dates" label="Ограничивать даты"></q-checkbox>
               </div>
               <div class="col-2" v-if="item.use_dates" style="padding-left:15px">
                  <q-input label="С даты" v-model="dates.from" dense>
                     <template v-slot:append>
                        <q-btn flat round dense icon="img:icons/clear-24px.svg" @click="dates.from = null"/>
                        <q-icon name="event" class="cursor-pointer">
                           <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                              <q-date v-model="dates.from" mask="DD.MM.YYYY">
                                 <div class="row items-center justify-end">
                                    <q-btn v-close-popup label="Выбрать" color="primary" flat/>
                                 </div>
                              </q-date>
                           </q-popup-proxy>
                        </q-icon>
                     </template>
                  </q-input>
               </div>
               <div class="col-2" v-if="item.use_dates" style="padding-left:15px">
                  <q-input label="По дату" v-model="dates.to" dense>
                     <template v-slot:append>
                        <q-btn flat round dense icon="img:icons/clear-24px.svg" @click="dates.from = null"/>
                        <q-icon name="event" class="cursor-pointer">
                           <q-popup-proxy ref="qDateProxy2" transition-show="scale" transition-hide="scale">
                              <q-date v-model="dates.to" mask="DD.MM.YYYY" :options="filterMinDate">
                                 <div class="row items-center justify-end">
                                    <q-btn v-close-popup label="Выбрать" color="primary" flat/>
                                 </div>
                              </q-date>
                           </q-popup-proxy>
                        </q-icon>
                     </template>
                  </q-input>
               </div>
            </div>
            <div class="row">
               <div class="col-2">
                  <q-checkbox v-model="item.use_hours" label="Ограничивать Время"></q-checkbox>
               </div>
               <div class="col-2" v-if="item.use_hours" style="padding-left:15px">
                  <q-select label="Время с" :options="hours" v-model="item.hours.from" dense></q-select>
               </div>
               <div class="col-2" v-if="item.use_hours" style="padding-left:15px">
                  <q-select label="Время по" :options="restHours" v-model="item.hours.to" dense></q-select>
               </div>
            </div>
         </q-card-section>
         <q-card-actions>
            <delete-button  @click="delDialogOpen = true"/>
         </q-card-actions>
      </q-card-section>
      <custom-dialog  title="Удаление" :trigger="delDialogOpen" @input="delDialogOpen = $event" :buttons="dialogButtons">
         <span>Удалить фото?</span>
      </custom-dialog>
   </q-card>
</template>
<script>
    import moment from "moment";
    import "moment/locale/ru";
    import {defineComponent} from 'vue';
    import Api from 'src/lib/api/admin-api';
    import Helpers from 'src/lib/api/helpers';
    import DeleteButton from '../DeleteButton';
    import CustomDialog from '../CustomDialog';

    export default defineComponent({
        name: "MainPhotoCmsEntry",
        props: ['item'],
        emits: ['delete', 'drop'],
        components: {
           DeleteButton,
           CustomDialog,
        },
        computed: {
            restHours() {
                let startHour = this.item.hours.from ?? '00:00';
                startHour = startHour.replace(':00', '');
                startHour = parseInt(startHour, 10);
                return this.hours.slice(startHour);
            },
            dialogButtons() {
               return [
                  {
                     title: 'Отмена',
                     type: 'light',
                  },
                  {
                     title: 'Ок',
                     type: 'purple',
                     action: this.dropItem,
                  },
               ];
            },
        },
        watch: {
            dates: {
                deep: true,
                handler() {
                    this.datesChange();
                }
            },
        },
        created() {
            this.dates = {from: this.toScreenDate(this.item.dates.from), to: this.toScreenDate(this.item.dates.to)};
        },
        data() {
            return {
                hours: ['00:00', '01:00', '02:00', '03:00', '04:00', '05:00', '06:00', '07:00', '08:00', '09:00',
                    '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00',
                    '20:00', '21:00', '22:00', '23:00', '00:00'
                ],
                dates: {from: null, to: null},
                delDialogOpen: false,
            };
        },
        methods: {
            datesChange() {
                this.item.dates = {from: this.toUnixDate(this.dates.from), to: this.toUnixDate(this.dates.to)};
                let itemDates = this.item.dates;
                if (itemDates.to != null && itemDates.from != null && itemDates.from >= itemDates.to) {

                    itemDates.to = null;
                    this.dates.to = null;
                }
            },
            toScreenDate(val) {
                if (val === null) return null;
                var dt = moment(val);
                if (!dt.isValid()) return null;
                return dt.format('DD.MM.YYYY');
            },
            toUnixDate(date) {
                if (date === null) return null;
                var mmt = moment.utc(date, "DD.MM.YYYY");
                if (!mmt.isValid()) return null;
                return mmt.format('YYYY-MM-DD');
            },
            dropItem() {
                this.$emit('drop', this.item);
            },
            filterMinDate(date) {
                let dateIso = date.replaceAll('/', '-');
                if (!this.item.dates.from) return true;
                return dateIso > this.item.dates.from;
            },
        }

    });
</script>
<style>
   .myForm div.row > div {
      padding-left: 10px;
   }
</style>