<template>
    <q-dialog v-model="dialogTrigger" persistent>
        <q-card>
            <q-card-section class="row justify-between" style="border-bottom: 1px solid #aaa">
                <div class="dialogTitle text-h6 text-bold">Подтвердите удаление</div>
                <q-btn flat round dense icon="img:icons/clear-24px.svg" v-close-popup />
            </q-card-section>
            <q-card-section class="row items-center dialog-body">
                <span class="q-ml-sm">{{ message }}</span>
            </q-card-section>

            <q-card-actions align="right" style="border-top: 1px solid #aaa">
                <custom-button title="Отмена" type="light" v-close-popup />
                <custom-button title="Ок" type="purple" @click="delSelected()" />
            </q-card-actions>
        </q-card>
    </q-dialog>
</template>

<script>
import CustomButton from './CustomButton';

export default {
    name: "DelItemsDialog",
    props: ['message', 'trigger'],
    emits: ['input', 'commit'],
    components: {
        CustomButton,
    },
    data() {
        return {
            dialogTrigger: false
        }
    },
    mounted() {
        this.dialogTrigger = this.trigger;
    },
    watch: {
        trigger(oldVal, newVal) {
            this.dialogTrigger = this.trigger;
        },
        dialogTrigger() {
            this.$emit('input', this.dialogTrigger);
        }
    },

    methods: {
        delSelected() {
            this.$emit('commit', true);
            this.dialogTrigger = false;
        }
    }
}
</script>

<style lang="scss">
    .dialogTitle {
        color: #3C414D;
    }
</style>