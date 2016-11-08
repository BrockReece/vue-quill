# vue-quill
A vue component wrapping the quill editor

## Installation
```
npm install --save vue-quill
```

You will also need to include the css file matching your chosen theme in your project
```html
<link href="https://cdnjs.cloudflare.com/ajax/libs/quill/1.1.5/quill.snow.min.css" rel="stylesheet">
```
or
```html
<link href="https://cdnjs.cloudflare.com/ajax/libs/quill/1.1.5/quill.bubble.min.css" rel="stylesheet">
```

## Usage
Install the vue plugin
```js
Vue.use(require('vue-quill'))
```
### Component
```html
<quill :content="content"></quill>
```
You may want to initialize the variable as a valid delta object too

```js
data() {
    return {
        content: {
            ops: [],
        },
    }
}
```

### Configuration
```html
<quill :content="content" :config="config"></quill>
```
You can also provide a config object as described in http://quilljs.com/docs/configuration/

```js
data() {
    return {
        content: {
            ops: [],
        },
        config: {
            readOnly: true,
            placeholder: 'Compose an epic...',
        },
    }
}
```

### Custom Filter
The plugin also installs a custom filter for converting a delta object to raw html

```html
<span v-html="content | quill"></span>
```

## Options
By default, the component outputs the content as a delta object, you can pass in a prop to return raw html
```html
<quill :content="content" output="html"></quill>
```

## Custom Formats
To add custom formats to the editor, you can pass an array of formats as a prop. The array should be in the following format
```js
formats: [
    {
        name: 'custom',
        options: {
            attribute: 'custom',
        },
    },
],
```

## Custom Keybindings
You can add custom keybindings by passing through an array in the props, the array should be in the following format
```js
keyBindings: [
    {
        key: 's',
        method: function(range) {
            this.$emit('save', this.editor, range)
            return false
        },
    },
]
```

## Events
This quill component emit events when the text or selection changes on the quill editor, you can listen for these on the parent component by declaring an event similar to this
```html
<quill :content="content" v-on:selection-change="selectionChange"></quill>
```

```js
methods: {
    selectionChange(editor, range) {
        if (range) {
            if (range.start !== range.end) {
                this.selectedText = editor.getText(range.start, range.end)
                editor.formatText(range, 'custom', 'hello world')
            }
        }
    }
},
```
