<template>
    <section>
        <q-table :columns="Table.columns" :rows="Table.list" :loading="Table.loading" no-data-label="Нет данных"
                 v-model:pagination="Table.pagination" hide-pagination
                 @request="Table.load" class="no-shadow no-border sticky-header-table" wrap-cells>

            <template v-slot:header="props">
                <q-tr :props="props">
                    <q-th auto-width v-if="Table.deletable">
                        <q-checkbox toggle-indeterminate v-model="Table.allSelected"
                                    @click.native="Table.massSelect(Table)" dense/>
                    </q-th>
                    <q-th
                        v-for="col in props.cols"
                        :key="col.name"
                        :props="props"
                    >
                        {{ col.label }}
                    </q-th>
                </q-tr>
            </template>

            <template v-slot:body="props">
                <q-tr :props="props" :style="Table.rowStyle(props.row)" @click="Table.rowEdit(props.row)">
                    <q-td v-if="Table.deletable">
                        <q-checkbox v-model="props.row._sel" @click.native="Table.selClick(Table)" dense/>
                    </q-td>
                    <slot :row="props.row">
                        <q-td
                            v-for="col in props.cols"
                            :key="col.name"
                            :props="props"
                        >
                            {{ props.row[col.name] }}
                        </q-td>
                    </slot>
                </q-tr>
            </template>
            <template v-slot:bottom="scope">
                <custom-pagination :scope="scope" :pagination="Table.pagination" @loadData="Table.load" />
            </template>
        </q-table>
    </section>
</template>

<script>
import { defineComponent } from 'vue';
import CustomPagination from 'src/components/CustomPagination';

export default defineComponent({
    name: "CommonTable",
    props: {
        Table: {type: Object, default: null},
    },
    components: {
        CustomPagination,
    },
    computed: {
        pagesCount() {
            return Math.ceil(this.Table.pagination.rowsNumber / this.Table.pagination.rowsPerPage);
        },
    },
    data() {
        return {};
    },
    methods: {}

});
</script>