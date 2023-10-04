<template>
   <SearchBar
   v-model="Filters.search"
   title="Поиск">
      <div class="row">
         <div class="col-8">
            <q-select
            v-model="Filters.log_level"
            label="Статус"
            :options="logLevels"
            option-value="id"
            option-label="title"
            map-options
            emit-value
            dense
            clearable/>
         </div>
         <div class="col-4">
            <q-btn
            @click="loadData(null)"
            icon="sync"
            dense/>
         </div>
      </div>


   </SearchBar>
   <q-table :columns="Table.columns" :rows="Table.list" :loading="Table.loading" no-data-label="Нет данных"
            v-model:pagination="Table.pagination"
            @request="loadData" class="no-shadow no-border sticky-header-table" wrap-cells>

      <template v-slot:header="props">
         <q-tr :props="props">
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
         <q-tr :props="props" style="cursor: pointer">

            <q-td style="width: 100px">{{ props.row.id }}</q-td>
            <q-td style="width: 100px">{{ unixTime(props.row.created_at) }}</q-td>
            <q-td style="width: 200px">{{ props.row.job_code}}</q-td>
            <q-td style="width: 200px">{{ logLevel(props.row.log_level)}}</q-td>
            <q-td>
               <pre>{{ props.row.message}}</pre>
            </q-td>

         </q-tr>
      </template>
   </q-table>
</template>
<script>
    import {defineComponent} from 'vue'
    import UI from 'src/lib/ui/objects';
    import Api from 'src/lib/mailer/api';
    import SearchBar from 'src/components/SearchBar';

    import Helpers from 'src/lib/api/helpers';

    export default defineComponent({
        name: "MessageLogTable",
        props: ['message_id'],
        components: {SearchBar},
        data() {
            const Table = new UI.TableContainer('id', true, 20, [
                new UI.TableColumn('id', 'ID', true, 'left'),
                new UI.TableColumn('created_at', 'Дата', true, 'left'),
                new UI.TableColumn('job_code', 'Sender Job', true, 'left'),
                new UI.TableColumn('log_level', 'Уровень', true, 'left'),
                new UI.TableColumn('message', 'Сообщение', true, 'left')
            ]);


            return {
                Filters: {search: '', message_id: 0, log_level: null},
                loading: false,
                Table: Table,
                logLevels: [{id: 0, title: 'Ошибка'}, {id: 1, title: 'Админ'}, {id: 2, title: 'Информация'},
                    {id: 10, title: 'debug'}]
            };
        },
        computed: {},
        mounted() {
            this.loadData();
        },
        watch: {
            Filters: {
                // This will let Vue know to look inside the array
                deep: true,
                handler() {
                    this.loadData();
                }
            },
            message_id() {
                this.loadData();
            }
        },
        methods: {
            logLevel(level) {
                //0 - error, 1 - admin actions, 2 - info, 10 - debug
                switch (level) {
                    case 0:
                        return 'Ошибка';
                    case 1:
                        return 'Админ';
                    case 2:
                        return 'Информация';
                    case 10:
                        return 'debug';
                }
                return level;
            },
            unixTime: Helpers.friendlyUnixDateTime,
            rowStyle(row) {
                return 'cursor:pointer';
            },
            loadData(pagination) {
                pagination = (pagination && pagination.pagination) ? pagination.pagination : this.Table.pagination;
                if (!this.message_id) return;
                this.Filters.message_id = this.message_id;
                this.Table.loading = true;
                Api.logs.list(pagination, this.Filters).then((data) => {
                    this.Table.loading = false;
                    if (data) {
                      this.Table.assign(data);
                      if(!data.list.length){
                        this.$q.notify({
                          message: 'Нет результатов',
                          caption: '',
                          color: 'red'
                        });
                      }
                    }

                });

            }

        },
        setup() {

        },

    });
</script>