<template>
   <tr class="highlighted-row">
         <td class="no-wrap text-right"><q-icon v-if="!readOnly" name="swap_vert"></q-icon>{{obj.id}}</td>
         <td><q-input bg-color="white" outlined v-model="obj.code" :readonly="obj.is_custom===0 || readOnly" dense></q-input></td>
         <td><q-input outlined v-model="obj.title" dense :readonly="readOnly"></q-input></td>
         <td><q-select outlined v-model="obj.data_type" :readonly="obj.is_custom===0 || readOnly"
                       :options="dataTypes"  map-options emit-value dense/></td>
         <td style="text-align: center"><template v-if="readonly">
             {{ booleanToString(obj.visible) }}
         </template><q-checkbox outlined v-model="obj.visible" dense :disable="readOnly"/></td>
         <td style="text-align: center"><q-checkbox outlined v-model="obj.editable" dense :disable="readOnly"/></td>
         <td v-if="!readOnly">
             <delete-button type="button"  @click="delDialogOpen = true"/>
         </td>
         <custom-dialog title="Удаление аттрибута" :trigger="delDialogOpen" @input="delDialogOpen = $event" :buttons="dialogButtons">
             <span>{{ dialogMessage }}</span>
         </custom-dialog>
   </tr>
</template>
<script>
    import {defineComponent} from 'vue';
    import globalState from 'src/lib/state';
    import ErrorHints from 'src/components/ErrorHints';
    import { booleanToString } from 'src/lib/api/helpers';
    import { find } from 'lodash';
    import DeleteButton from '../DeleteButton';
    import CustomDialog from '../CustomDialog';

    export default defineComponent({
        name: "AttributeEditRow",
        props: {
          obj: { type: Object, required: true },
          readonly: { type: Boolean, default: false },
          draggable: { type: Boolean, default: true }
        },
        components: { ErrorHints, DeleteButton, CustomDialog },
        emits: ['delete'],
        computed: {
            showDialog() {
                return this.obj != null;
            },
            readOnly() {
                return globalState.User.isReadOnly('portal-admin-pdo-categories');
            },
            dialogMessage() {
                let notice = this.obj.is_custom===1?'':"\n После очередной синхронизации атрибут будет загружен снова.";
                return 'Уверены, что хотите удалить аттрибут "'+this.obj.title+'"?'+notice;
            },
            dialogButtons() {
                return [
                    {
                     title: 'Отмена',
                     type: 'light',
                  },
                  {
                     title: 'Ок',
                     type: 'purple',
                     action: this.delEntry,
                  },
                ]
            }
        },
        watch: {

        },
        data() {
            return {
                isNew: false,
                errors: null,
                dataTypes:[{label: 'Строка', value: 'string'},
                    {label: 'Да/нет', value: 'boolean'},
                    {label: 'Словарь', value: 'dictionary'},
                    {label: 'Часы работы', value: 'workinghours'},
                    {label: 'Целое число', value: 'integer'},
                    {label: 'Дробное число', value: 'float'},
                ],
                delDialogOpen: false,
            };
        },
        methods: {
            delEntry() {
                this.$emit('delete', this.obj.id);
            },

          getDataTypeByValue(value) {
              const type = find(this.dataTypes, item => item.value === value);
              return type || { label: 'Нет типа', value: null };
          },

          booleanToString
        }

    });
</script>