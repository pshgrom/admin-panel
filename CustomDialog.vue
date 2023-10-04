<template>
   <q-dialog v-model="dialogTrigger" persistent>
      <q-card class="dialogCard">
         <q-card-section class="titleWrapper" style="border-bottom: 1px solid #aaa">
            <div class="dialogTitle text-h6 text-bold">{{ title }}</div>
            <q-btn flat round dense icon="img:icons/clear-24px.svg" v-close-popup />
         </q-card-section>
         <q-card-section class="row items-center dialog-body">
            <slot>

            </slot>
         </q-card-section>

         <q-card-actions align="right" style="border-top: 1px solid #aaa">
            <custom-button v-for="button in buttons" :key="button.title + button.type" :title="button.title" :type="button.type" v-close-popup="!button.action"  @click="button.action" />
         </q-card-actions>
      </q-card>
   </q-dialog>
</template>

<script>
  import {defineComponent} from 'vue';
  import CustomButton from './CustomButton';

	export default defineComponent({
		name: "CustomDialog",
		props: ['trigger', 'title', 'buttons'],
    emits:['input'],
		data() {
			return {
				dialogTrigger: false,
        buttonList: [],
			}
		},
      mounted(){
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
    components: {
      CustomButton,
    },
	});
</script>

<style lang="scss">
  .dialogCard {
    min-width: 300px;
  }
  .titleWrapper {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .dialogTitle {
    color: #3C414D;
  }
</style>
