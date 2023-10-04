<template>
   <div>
      <q-separator v-if="item.separator"/>
      <q-item v-if="!item.separator && item.list==null && !item.isSelected($route.path)" :to="item.to"
              active-class="q-item-no-link-highlighting" dense>
         <q-item-section avatar v-if="item.icon!=''">
            <q-icon :name="item.icon"/>
         </q-item-section>
         <q-item-section>
            <q-item-label>{{item.name}}</q-item-label>
         </q-item-section>
      </q-item>
      <q-item v-if="!item.separator && item.list==null && item.isSelected($route.path)" class="bg-primary text-white"
              style="background-color: #8C7ACE; color:#FFF" dense>
         <q-item-section avatar v-if="item.icon!=''">
            <q-icon :name="item.icon"/>
         </q-item-section>
         <q-item-section>
            <q-item-label>{{item.name}}</q-item-label>
         </q-item-section>
      </q-item>
      <hr v-if="item.separator" role="separator" aria-orientation="horizontal"
          class="q-separator q-separator q-separator--horizontal"/>
      <q-expansion-item v-if="item.list!=null"
                        expand-separator
                        :icon="item.icon"
                        :label="item.name"
                        v-model="expanded"
                        dense>
         <menu-item v-for="subitem in item.list" :key="subitem.id" :item="subitem" style="margin-left:30px">

         </menu-item>
      </q-expansion-item>
   </div>
</template>

<script>
    import api from 'src/lib/api/http_api';
    import md5 from 'md5';

    export default {
        name: "MenuItem",
        props: ['item'],
        data() {

            return {
                expanded: false
            }
        },
        created(){
            this.itemSet();
        },
        watch: {
            item() {
                this.itemSet();
            },
            expanded(){
                if (this.item){
                    const myStorage = window.localStorage;
                    myStorage.setItem('_menu_' + md5(this.item.name), this.expanded?1:0);
                }
            }
        },

        methods: {
            itemSet(){
                if (this.item && this.item.list!=null) {
                    const myStorage = window.localStorage;
                    const hash = md5(this.item.name);
                    const fromCookies = myStorage.getItem('_menu_' + md5(this.item.name));
                    this.item.expanded = (this.item.expanded||fromCookies==1);

                    this.expanded = this.item.expanded;
                }
            }
        }
    }
</script>

<style scoped>

</style>
