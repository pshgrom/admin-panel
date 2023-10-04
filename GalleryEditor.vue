<template>
   <q-card class="my-card col-12" style="margin-top: 20px" v-if="obj.json&& obj.json.gallery">
      <q-card-section class="flex-block">
         <q-avatar v-for="item in obj.json.gallery" :key="item.id" :item="item" rounded size="150px"
                   style="margin:5px 5px;cursor:pointer"
                   @click="showInfo(item)">
            <img :src="itemUrl(item)"/>
            <q-badge floating color="blue" style="border:1px solid black;cursor: pointer;font-size:20px">
               <q-icon name="delete_forever" color="white" @click="openDialog(item)"/>
            </q-badge>
         </q-avatar>
         <q-avatar rounded size="150px" style="margin:5px 5px">
            <q-uploader ref="uploader" style="width:150px;margin:5px 5px"
                        label="Загрузить файл"
                        auto-upload
                        :factory="factoryFn"
                        field-name="photo"
            />
         </q-avatar>
      </q-card-section>
      <q-card-section v-if="selItem">
         <span style="font-size: 1.2em;font-weight: bold">Файл {{selItem.id}}</span>
         <q-markup-table dense>
            <tbody>
            <tr v-for="file in selItem.files" :key="file.id">
               <td>{{file.file_type}}</td>
               <td>{{CONFIG.SRV_MEDIA_URL + '/file/link?id='+selItem.id+'&type='+file.file_type}}</td>
               <td>
                  <q-btn dense icon="content_copy" @click="copyPath(selItem.id, file.file_type)"></q-btn>
               </td>
               <td><a :href="fullPath(file.path)" target="_blank">Открыть</a></td>
            </tr>
            </tbody>
         </q-markup-table>
      </q-card-section>
			<custom-dialog  title="Удаление" :trigger="delDialogOpen" @input="delDialogOpen = $event" :buttons="dialogButtons">
         <span>Удалить файл?</span>
      </custom-dialog>
   </q-card>
</template>

<script>
	import Helpers from 'src/lib/api/helpers';
	import Api from 'src/lib/api/admin-api';
	import {copyToClipboard} from 'quasar';
	import CustomDialog from './CustomDialog';

	export default {
		name: "GalleryEditor",
		props: ['obj'],
		data() {
			return {
				loading: true,
				selItem: null,
        CONFIG:CONFIG,
				delDialogOpen: false,
				dialogItem: null,
			}
		},
		watch: {},
		components: {
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
		created() {

		},
		methods: {
			copyPath(id, type) {
				copyToClipboard(CONFIG.SRV_MEDIA_URL + '/file/link?id='+id+'&type='+type).then(() => {
					this.$q.notify({
						message: 'Скопировано',
						color: 'primary'
					});
				});
			},
			fullPath(path) {
				return CONFIG.SRV_MEDIA_URL + path;
			},
			showInfo(item) {
				if (this.selItem && this.selItem.id == item.id) {
					this.selItem = null;
					return;
				}
				this.selItem = item;
			},
			dropItem(item) {
				this.dialogItem = null;
				this.delDialogOpen = false;

				for (let i = 0; i < this.obj.json.gallery.length; i++) {
					if (this.obj.json.gallery[i].id === item.id) {
						this.obj.json.gallery.splice(i, 1);
						this.selItem = null;
						return;
					}
				}
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
			itemUrl(item) {
				let url = this.searchFileType(item, 'thumb_sm');
				if (!url) url = this.searchFileType(item, 'thumb_lg');
				return url ?? 'img/no-photo.svg';
			},
			factoryFn(file) {
				let me = this;
				Api.cms.addFile(me.obj.id, file[0]).then((data) => {
					if (data.id) {
						this.obj.json.gallery.push(data);

					} else {
						me.$q.notify({
							message: data,
							color: 'red'
						});
					}
					me.$refs.uploader.reset();
				});
			},
			openDialog(item) {
				this.delDialogOpen = true;
				this.dialogItem = item;
			},
			...Helpers

		}
	}
</script>

<style scoped>

</style>