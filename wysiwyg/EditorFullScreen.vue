<template>
    <span>
    <q-btn dense no-caps
           ref="color"
           no-wrap
           unelevated
           text-color="black"
           size="sm" icon="fullscreen" @click="toggleFullScreen">
    </q-btn>

        </span>
</template>
<script>
import UI from 'src/lib/ui/objects';

export default {
    name: 'EditorFullScreen',
    props: {
        reference: {
            type: Object
        }
    },
    emits: ['changed'],
    computed:{

    },
    data() {
        return {

        }
    },
    methods: {
        toggleFullScreen(){
            this.reference.toggleFullscreen();
            //this.$emit('changed', true);
            UI.onEditorResize(this.reference, true);
        }

    }
}
</script>