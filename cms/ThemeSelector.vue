<template>
    <div class="theme-selector">
        <div class="col-5">

            <q-tree
                v-if="!isLoading"
                :nodes="themesTree"
                node-key="tree_path"
                v-model:ticked="tickedData"
                v-model:expanded="expanded"
                tick-strategy="leaf"
                class="col-12 col-sm-6"/>

            <q-spinner
                v-else
                color="primary"
                size="60px"/>

        </div>
    </div>
</template>

<script>
import Api from 'src/lib/special/api';

export default {
    name: "ThemeSelector",
    props: {
        ticked: {type: Array, default: []}
    },
    data() {
        return {
            themesTree: [],
            tickedData: this.ticked,
            expanded: [],
            themes: [],
            isLoading: false
        }
    },

    methods: {
        prepareTicked() {
            let res = [];
            const themes = this.themes;

            this.tickedData.forEach(function (tick) {
              const parentId = tick.substr(0, tick.indexOf('.'));
              const theme = typeof tick === 'string'          //theme.tree_path or theme.id
              ? themes.find(themeItem => themeItem.tree_path == tick)
              : themes.find(themeItem => themeItem.id == tick);

              res.push({id: theme.id, title: theme.title, parent_id: parentId, tree_path: theme.tree_path});
            });

            this.$emit('theme-selector:update', res);
        }
    },

    async created() {

        function buildTree(items, parentId = null) {
            let branch = [];

            items.forEach(function (item) {
                item.label = item.title;

                if (item.parent_id === parentId) {
                    let children = buildTree(items, item.id);
                    if (children) {
                        item.children = children;
                    }
                    branch.push(item);
                }
            })
            return branch;
        }

        this.isLoading = true;
        await Api.themes.tree().then((data) => {
            if (data) {
                this.themes = data;
                this.themesTree = buildTree(data);
            }

            const themes = this.themes;
            if (this.tickedData.length) {
                let na = [];
                for (let i = 0; i < this.tickedData.length; i++) {
                    const tick = this.tickedData[i];
                    const theme = themes.find(themeItem => themeItem.id == tick);
                    if (theme) {
                      na.push(theme.tree_path);
                      const treePath = theme.tree_path.split('.');
                      for (let j = 0; j < treePath.length; j++) {
                        const sliced = treePath.slice(0, j+1);
                        this.expanded = this.expanded.concat(sliced.join('.'));
                      }
                    }

                }

                this.tickedData = na;
            }

            this.isLoading = false;
        });
    },

    watch: {
        tickedData: {
            handler() {
                this.prepareTicked();
            }
        }
    }
}
</script>

<style scoped>

</style>