<template>
    <div>
        <q-select input-debounce="500" :outlined="outlined" :label="label" type="search" v-model="organization"
                  dense
                  use-input :options="options" map-options
                  @filter-abort="abortFilterFn" @filter="search" clearable
                  :rules="[val => validate(val)]" :readonly="readonly"
        >
            <template v-slot:option="scope">
                <q-item v-bind="scope.itemProps" dense>
                    <q-item-section avatar>
                        {{ scope.opt.value }}
                    </q-item-section>
                    <q-item-section>
                        <q-item-label v-html="scope.opt.label"/>
                    </q-item-section>
                </q-item>
            </template>
        </q-select>
        <div class="row" v-if="showPosOrg">
            <div class="col-12" style="padding-left: 10px;padding-top: 10px">
                <label><b>Ответственный в ПОС:</b>&nbsp;{{posName}}</label>
            </div>
        </div>
    </div>
</template>
<script>
import {defineComponent} from 'vue';
import Api from 'src/lib/pdo/api';

export default defineComponent({
    name: "PdoOrganizationSelect",
    props: {
        showPosOrg: {
            type: Boolean,
            default: false
        },
        modelValue: {
            type: Number,
            default: 0
        },
        outlined: {
            type: Boolean,
            default: false
        },
        required: {
            type: Boolean,
            default: false
        },
        readonly: {
            type: Boolean,
            default: false
        },
        label: {
            type: String,
            default: 'Название балансодержателя'
        }
    },
    emits: ['update:modelValue'],
    watch: {
        modelValue() {
            this.modelChanged();
        },
        organization() {
            const value = this.organization ? this.organization.value : 0;
            this.$emit('update:modelValue', value);
        }
    },
    data() {
        return {
            options: [],
            organization: null
        };
    },
    mounted() {
        this.modelChanged();
    },
    computed: {
        posName() {
            let posName = '';
            for (let i = 0; i < this.options.length; i++) {
                if (this.options[i].value == this.modelValue) {
                    posName = (this.options[i].pos_name ?? '');
                }
            }
            return posName;
        }
    },
    methods: {
        validate(val) {
            if (!this.required) return true;
            return (val !== 0 && val != null) || 'Заполните организацию';
        },
        abortFilterFn() {

        },
        addFilterVal() {

        },
        modelChanged() {
            if (this.organization && this.organization.id === this.modelValue) return;
            Api.owner.organizationSearch(this.modelValue).then((list) => {
                this.organization = list.find(item => item.value == this.modelValue);
                this.options = list;
            });
        },
        search(val, update, abort) {
            // if (val.trim() === '') {
            //    update(() => {
            //       this.options = [];
            //    });
            //    return;
            // }
            Api.owner.organizationSearch(val).then((list) => {
                update(() => {
                    this.options = list;
                });
            });

        },
    }

});
</script>