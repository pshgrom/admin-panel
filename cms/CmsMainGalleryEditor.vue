<template>
   <div class="row" v-if="obj">
      <div class="col-12">

         <q-btn dense class="bg-primary text-white" style="margin-bottom: 15px" @click="addItem">Добавить фото</q-btn>
         <draggable
            :list="obj"
            item-key="id"
            ghost-class="ghost"
            tag="div"
         >
            <template #item="{ element  }">
               <main-photo-cms-entry :item="element" @drop="dropItem"></main-photo-cms-entry>
            </template>
         </draggable>
      </div>
   </div>
</template>

<script>
    import state from "src/lib/state";
    import Helpers from 'src/lib/api/helpers';
    import MainPhotoCmsEntry from 'src/components/cms/MainPhotoCmsEntry';
    import draggable from "vuedraggable";

    export default {
        name: "CmsMainGalleryEditor",
        props: ['obj'],
        components: {MainPhotoCmsEntry, draggable},
        data() {
            return {
                dense: true,
            }
        },
        watch: {},
        created() {

        },

        methods: {
            addItem() {
                let id = 0;
                for (let i = 0; i < this.obj.length; i++) {
                    if (this.obj[i].id >= id) {
                        id = this.obj[i].id + 1;
                    }
                }
                this.obj.push({
                    id: id,
                    use_dates: false,
                    use_hours: false,
                    hours: {from: null, to: null},
                    dates: {from: null, to: null},
                    url: ''
                });
            },
            dropItem(item) {
                for (let i = 0; i < this.obj.length; i++) {
                    if (this.obj[i].id === item.id) {
                        this.obj.splice(i, 1);
                        return;
                    }
                }
            },
            ...Helpers

        }
    }
</script>

<style scoped>

</style>
