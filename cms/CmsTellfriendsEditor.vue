<template>
   <div class="row">
      <div class="col-12">

         <q-item-label class="q-pb-xs">Листовки</q-item-label>
         <q-btn dense class="bg-primary text-white" style="margin-bottom: 15px" @click="addItem">Добавить листовку</q-btn>
         <q-markup-table dense>
            <thead>
            <tr>
               <th rowspan="2">Название</th>
               <th rowspan="2">Фото</th>
               <th colspan="2">Линк на файл</th>
               <th></th>
            </tr>
            <tr>
               <th>цветная</th>
               <th>чёрно-белая</th>
            </tr>
            </thead>
            <tbody>
            <tr class="highlighted-row" v-for="item in obj.json.items" :key="item.id">
               <td>
                  <q-input outlined v-model="item.title" :dense="dense" label="Название"/>
               </td>
               <td>
                  <q-input outlined v-model="item.photo" :dense="dense" label="Фото"/>
               </td>
               <td>
                  <q-input outlined v-model="item.colorfull" :dense="dense" label="Цветная"/>
               </td>
               <td>
                  <q-input outlined v-model="item.blackandwhite" :dense="dense" label="Чёрнобелая"/>
               </td>
               <td>
                  <delete-button  @click="openDialog(item)"/>
               </td>
            </tr>
            </tbody>
         </q-markup-table>
      </div>
      <custom-dialog  title="Удаление" :trigger="delDialogOpen" @input="delDialogOpen = $event" :buttons="dialogButtons">
         <span>Удалить листовку?</span>
      </custom-dialog>
   </div>
</template>

<script>
	import state from "src/lib/state";
	import Helpers from 'src/lib/api/helpers';
	import DeleteButton from '../DeleteButton';
   import CustomDialog from '../CustomDialog';

	export default {
		name: "CmsTellfriendsEditor",
		props: ['obj'],
		components: {
				DeleteButton,
            CustomDialog,
		},
		data() {
			return {
				editorToolbar: state.editorToolbar(this.$q),
				newsDate: null,
				dense: true,
				dpLocale: state.datePickerLocale,
				loading: true,
				fonts: state.editorFonts,
            delDialogOpen: false,
            dialogItem: null,
			}
		},
		watch: {},
		created() {

		},
      computed: {
           dialogButtons() {
              if (!this.dialogItem) {
                 return [];
              }

               return [
                  {
                     title: 'Отмена',
                     type: 'light',
                  },
                  {
                     title: 'Ок',
                     type: 'purple',
                     action: () => this.dropItem(this.dialogItem),
                  },
               ];
            },
        },
		methods: {
			addItem() {
				let id = 0;
				for (let i = 0; i < this.obj.json.items.length; i++) {
					if (this.obj.json.items[i].id >= id) {
						id = this.obj.json.items[i].id + 1;
					}
				}
				this.obj.json.items.push({id: id, title: '', photo: '', colorfull:'', blackandwhite:''});
			},
			dropItem(item) {
            this.dialogItem = null;
            this.delDialogOpen = false;

            for (let i = 0; i < this.obj.json.items.length; i++) {
               if (this.obj.json.items[i].id === item.id) {
                  this.obj.json.items.splice(i, 1);
                  return;
               }
            }
			},
         openDialog(item) {
            this.delDialogOpen = true;
            this.dialogItem = item;
         },
			...Helpers,
		}
	}
</script>
