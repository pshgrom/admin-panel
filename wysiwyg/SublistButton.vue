<template>
    <q-btn dense no-caps
           ref="color"
           no-wrap
           unelevated
           text-color="black"
           label="Подсписок"
           size="sm" icon="segment" @click="insertSublist">

    </q-btn>
</template>

<script>
import UI from 'src/lib/ui/objects';
export default {
    name: 'SublistButton',
    props: {
        reference: {
            type: Object
        }
    },
    data() {
        return {}
    },
    methods: {
        insertSublist(cmd, name) {
            const edit = this.reference;
            edit.focus();
            const html = '<ul><li>Пункт 1</li><li>Пункт 2</li></ul>';
            this.pasteHtmlAtCaret(html);
            //const template = document.createElement('div');
            // const ul = document.createElement('ul');
            // let item = document.createElement('li');
            // item.innerText = 'Пункт 1';
            // ul.appendChild(item);
            // item = document.createElement('li');
            // item.innerText = 'Пункт 2';
            // ul.appendChild(item);
            // let node = window.getSelection().focusNode;
            // if (node.nodeName == '#text') {
            //     node = node.nextSibling;
            // }
            // if (node.className.indexOf('q-editor__content') >= -1) {
            //     node.appendChild(ul);
            // } else {
            //     node.parentElement.appendChild(ul);
            // }
            edit.focus();
            //чтобы снять выделение и поставить курсор в конец выделения
            setTimeout(function () {
                window.getSelection().collapseToEnd();
            }, 150);

        },
        pasteHtmlAtCaret: UI.pasteHtmlAtCaret
    }
}
</script>