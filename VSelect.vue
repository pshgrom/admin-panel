<template>

  <q-select v-model="selected" :options="opts" option-value="id"  option-label="label"
            :label="label"  @filter="filterFn" use-input :dense="dense" emit-value>
    <template v-slot:append>
      <q-icon name="close" @click.stop="clearSelection" class="cursor-pointer"></q-icon>
    </template>
  </q-select>

</template>

<script>
    export default {
        name: "VSelect",
        props: ['label','options', 'selected', 'dense'],
        data() {
            return {
                selection:null,
                opts:[]
            }
        },
        mounted(){
        },
        watch:{
            selection(){
              this.updateValue();
            },
            options(){
                this.opts = this.options;
            }
        },
        methods:{
            filterFn (val, update) {
                var me = this;
                console.log(val);
                if (val === '') {
                    update(() => {
                        this.opts = me.options;

                        // with Quasar v1.7.4+
                        // here you have access to "ref" which
                        // is the Vue reference of the QSelect
                    });
                    return
                }

                update(() => {
                    const needle = val.toLowerCase();
                    this.opts =  me.options.filter(v => v.label.toLowerCase().indexOf(needle) > -1);
                })
            },
            clearSelection(){
                this.selection = null;
                this.updateValue();
            },
            updateValue() {
                //this.$emit('input', this.selection);
            }
        }
    }
</script>

<style scoped>

</style>
