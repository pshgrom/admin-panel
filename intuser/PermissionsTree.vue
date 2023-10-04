<template>
    <q-card flat v-if="permissionsTree&&permissionsTree.tree&&obj">
        <!--                              <q-card-section>-->
        <!--                                 <span class="text-h6 no-margin">Стаканы</span>-->
        <!--                              </q-card-section>-->
        <q-card-section>
            <div class="row">
                <div class="col-6">
                    <q-input class="col-12"
                             ref="filterRef"

                             v-model="filter"
                             label="Поиск разрешений"
                             dense
                    >
                        <template v-slot:append>
                            <q-icon v-if="filter !== ''" name="clear" class="cursor-pointer" @click="resetFilter"/>
                        </template>
                    </q-input>
                    <div class="treeRow" style="width: 100%;display: block">
                        <div class="ptree_menuCol"
                             style="width: 100%;display: block;position: relative;height: 28px;border-bottom: 1px solid #aaa;padding-right: 10px">
                            Только просмотр
                        </div>
                    </div>
                    <q-tree style="width: 100%" v-if="editable&&obj"
                            :nodes="permissionsTree.tree"
                            ref="permTree"
                            node-key="id"
                            tick-strategy="leaf"
                            v-model:ticked="obj.permissions"
                            :filter="filter"
                            :filter-method="treeFilter"
                    >
                        <template v-slot:default-header="prop">
                            <div class="treeRow row items-stretch" style="width: 100%;display: block">
                                <q-icon name="o_folder"
                                        v-if="prop.node.is_menu && prop.node.parent_id>0"
                                        color="primary" class="text-weight-light"/>
                                <span :class="prop.node.tickable?'':'text-italic'">{{ prop.node.name }}</span>
                                <div class="ptree_menuCol"
                                     v-if="obj.permissions&&obj.permissions.indexOf(prop.node.id)>-1">
                                    <q-toggle v-model="cbModels['p'+prop.node.id].value"
                                              @click.stop="viewOnlyChanged(prop.node)"
                                              :disable="!prop.node.tickable"
                                    ></q-toggle>
                                </div>
                            </div>
                        </template>
                    </q-tree>
                </div>
                <div class="col-6" style="padding-left: 10px">

                    <div>
                        <div class="text-bold"
                             style="width: 100%;display: block;height: 28px;margin-top:8px;border-bottom: 1px solid #aaa;">
                            Выбрано
                        </div>
                        <div v-for="tick in obj.permissions"
                             :key="`ticked-${tick}`" :class="permissionName(tick).class">
                            {{ permissionName(tick).text }}
                        </div>
                    </div>
                </div>
            </div>
        </q-card-section>
    </q-card>
</template>
<style scoped>
.ptree_menuCol {
    position: absolute;
    right: 0;
    text-align: right;
    top: -6px;
    width: 160px;

}

.treeRow {
    margin: 9px 0 0;
    padding-top: 5px;
    padding-bottom: 15px;
    height: 35px;
    position: relative;
    border-bottom: 1px solid #eee;
}
</style>
<script>
import {defineComponent} from 'vue';
import Meta from 'src/lib/meta';

export default defineComponent({
    name: "PermissionsTree",
    props: {
        modelValue: {
            type: Object,
            default: null
        },
        obj: {
            type: Object,
            default: null
        },
        preset: {
            type: Object,
            default: null
        },
        editable: {
            type: Boolean,
            default: true
        }
    },
    emits: ['update:modelValue'],
    watch: {
        modelValue() {
            this.value = this.modelValue;
        },
        value() {
            this.$emit('update:modelValue', this.value);
        },
        preset: {
            // This will let Vue know to look inside the array
            deep: true,
            handler() {
                this.updIndex++;
            }
        }
    },
    data() {
        return {
            updIndex: 1,
            cbModels: {},
            filter: ''
        };
    },
    computed: {
        permissionsTree() {

            let nt = JSON.parse(JSON.stringify(Meta.auth.permissionsTree));
            if (this.obj && this.updIndex > 0) {
                this.adoptNodes(nt.tree);
            }
            return nt;
        },
    },
    async created() {
        await Meta.auth.load();
    },
    methods: {
        treeFilter(node, filter) {
            const filt = filter.toLowerCase()
            return node.name && node.name.toLowerCase().indexOf(filt) > -1;
        },

        resetFilter() {
            this.filter = ''
            this.$refs.filterRef.focus();
        },
        adoptNodes(nodes) {
            for (let i = 0; i < nodes.length; i++) {
                const node = nodes[i];
                if (!this.cbModels.hasOwnProperty('p' + node.id)) {
                    this.cbModels['p' + node.id] = {value: false};
                }
                if (this.obj.view_only.indexOf(node.id) >= 0) this.cbModels['p' + node.id].value = true;
                //в пресетах IDшники строки
                if (this.preset && this.preset.permissions.indexOf('' + node.id) >= 0) {
                    if (this.obj.permissions.indexOf(node.id) < 0) this.obj.permissions.push(node.id);
                    node.tickable = false;
                    if (this.preset.view_only.indexOf('' + node.id) >= 0) {
                        this.cbModels['p' + node.id].value = true;
                    }
                }
                if (node.children) this.adoptNodes(node.children);
            }

        },
        viewOnlyChanged(node) {
            const indx = this.obj.view_only.indexOf(node.id);
            if (this.cbModels['p' + node.id].value) {
                if (indx < 0) this.obj.view_only.push(node.id);
            } else {
                if (indx >= 0) this.obj.view_only.splice(indx, 1);
            }
            console.log(this.obj.view_only);
        },
        nothing() {

        },
        roleName(code) {
            return code + ': ' + this.rolesTree.typeIds[code] ?? 'Нет названия';
        },
        permissionName(id) {
            const map = this.permissionsTree.nodemap;
            const node = map[id];
            if (!node) return '???';
            let cls = '';
            //if (node.code == 'portal-admin-pdo-categories') debugger;
            let name = node.id + (node.code && node.code !== '' ? ': [' + node.code + '] ' : ' ') + this.getPath(node);
            if (this.obj.view_only.indexOf(id) >= 0 || (this.preset && this.preset.view_only.indexOf('' + id) >= 0)) {
                name += ' (только просмотр)';
            }
            if (this.preset && this.preset.permissions.indexOf('' + id) >= 0) cls = 'text-italic text-blue';
            return {text: name, class: cls};
        },
        getPath(node) {
            if (node == null) return '';
            if (node.parent_id === 0) return '';
            const map = this.permissionsTree.nodemap;
            const parent = map[node.parent_id];
            return this.getPath(parent) + '/' + node.name;
        },
    }

});
</script>