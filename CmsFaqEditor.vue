<template>
   <div class="row">
      <div class="col-12">

         <q-item-label class="q-pb-xs">Часто задаваемые вопросы</q-item-label>
         <q-btn dense class="bg-primary text-white" style="margin-bottom: 15px" @click="addItem">Добавить вопрос</q-btn>
         <q-markup-table dense class="vertical-top">
            <thead>
            <tr>
               <th>Вопрос</th>
               <th>Ответ</th>
               <th></th>
            </tr>
            </thead>
            <tbody>
            <tr class="highlighted-row" v-for="item in obj.json.items" :key="item.id">
               <td style="min-width: 300px" class="vertical-top">
                  <q-input outlined type="textarea"
                           autogrow
                     v-model="item.question"
                     :dense="dense"
                     label="Вопрос"
                  />
               </td>
               <td style="max-width: 60%" class="vertical-top">
                  <q-editor
                     v-model="item.answer"
                     :dense="dense"
                     :toolbar="editorToolbar"
                     :fonts="fonts"
                     label="Ответ"
                     min-height="2rem"
                  />
               </td>
               <td>
                  <delete-button @click="openDialog(item)"></delete-button>
               </td>
            </tr>
            </tbody>
            <q-btn class="bg-primary text-white" style="margin-bottom: 15px" @click="addItem">Добавить вопрос</q-btn>
         </q-markup-table>

      </div>
      <custom-dialog  title="Удаление" :trigger="delDialogOpen" @input="delDialogOpen = $event" :buttons="dialogButtons">
         <span>Удалить вопрос?</span>
      </custom-dialog>
   </div>
</template>

<script>
	import state from "src/lib/state";
	import Helpers from 'src/lib/api/helpers';
   import DeleteButton from './DeleteButton';
   import CustomDialog from './CustomDialog';

	export default {
		name: "CmsFaqEditor",
		props: ['obj'],
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
      components: {
         DeleteButton,
         CustomDialog,
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
				this.obj.json.items.push({id: id, question: '', answer: ''});
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
			...Helpers

		}
	}
</script>
