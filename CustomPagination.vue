<template>
   <div class="row justify-center items-center q-pa-lg customPagination" style="width: 100%;">
     <q-btn
     @click="scope.prevPage"
     :disable="scope.isFirstPage"
     icon="img:icons/chevron_left-24px.svg"
     flat
     dense
     class="no-padding"/>

     <q-pagination
     v-model="pagination.page"
     :max="scope.pagesNumber"
     :max-pages="6"
     :boundary-numbers="true"
     flat
     active-color="primary"
     active-text-color="gorodPrimary"
     text-color="gorodPrimary"/>

     <q-btn
     @click="scope.nextPage"
     :disable="scope.isLastPage"
     icon="img:icons/chevron_right-24px.svg"
     flat
     dense
     class="no-padding q-mr-md"/>

     <div class="paginationControls">
       <q-select
       :options="perPageOptions"
       v-model="pagination.rowsPerPage"
       outlined
       dense
       emit-value
       map-options
       class="paginationSelect"/>

       <span
       v-text="'Перейти'"
       class="q-mr-sm"/>

       <q-input
       :max="scope.pagesNumber"
       v-model="pageToGo"
       min="1"
       type="number"
       debounce="500"
       outlined
       dense
       class="paginationPageInput"/>

       <span
       v-text="`Всего записей: ${scope.pagination.rowsNumber}`"/>
     </div>

    </div>
</template>

<script>
  import { defineComponent } from 'vue';
  import { debounce } from 'lodash';

  export default defineComponent({
    name: "CustomPagination",
    props: ['scope', 'pagination'],
    emits:['loadData'],
    data() {
      return {
        perPageOptions: [{ label: "10 / страницу", value: 10 }, { label: "20 / страницу", value: 20 }, { label: "50 / страницу", value: 50 }],
        pageToGo: null,
      }
    },
    watch: {
      pageToGo() {
        if (!this.pageToGo)
          return;

        if (this.pageToGo > this.scope.pagesNumber) {
          this.pagination.page = Number(this.scope.pagesNumber);
          // this.pagination.page = Number(this.pageToGo);
          this.$emit('loadData');
        } else  if (this.pageToGo==0) {
          this.pagination.page = Number(this.pageToGo = 1);
          this.$emit('loadData');
        } else {
          this.pagination.page = Number(this.pageToGo);
          this.$emit('loadData');
        }
      },

      'pagination.page'() {
        this.pageToGo = null;
        this.load();
      },
    },
    methods: {
      load() {
        if (this.$_debounced) this.$_debounced.cancel();

        this.$_debounced = debounce(() => {
          this.$emit('loadData');
        }, 300);

        return this.$_debounced();
      },

    },
  });
</script>

<style lang="scss">
    .customPagination {
      position: relative;

      .q-pagination .q-btn {
        width: 36px;
        height: 36px;
        background: transparent !important;
        border: 1px solid $borders-gray;
        border-radius: 4px;
        margin-right: 8px;

        &.text-gorodPrimary {
          border-color: $primary;
        }

        &::before {
          box-shadow: none;
        }
      }

      .q-pagination > .q-btn {
        &:last-child {
          margin-right: 14px;
        }
        &:first-child {
          margin-left: 14px;
        }

        &:nth-child(2),
        &:nth-last-child(2) {
          border: none;
          background: no-repeat center center url(../../public/icons/more_horiz-24px.svg) !important;
          color: #fff !important;

          &:hover {
            background-color: $background-gray !important;
          }
        }
      }

      .paginationSelect {
        height: 36px;
        margin:0 16px;
        font-size: 16px;


      }
      .q-field__append,
      .q-field__control,
      .q-field__control::before {
        height: 36px;
      }

      & span {
        font-size: 16px;
      }

      .paginationPageInput {
        width: 80px;
        margin-right: 16px;
      }

      .paginationControls {
        display: flex;
        align-items: center;
        //position: absolute;
        right: 0;

        @media(max-width: 1800px) {
          position: static;
        }
      }
    }
</style>
