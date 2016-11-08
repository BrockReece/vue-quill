<template>
  <div class="ui attached segment"
       @click.prevent="focusEditor"
       v-on:focus-editor="focusEditor"
       v-on:set-html="setHTML"
       v-on:set-content="setContents">
  </div>
</template>

<script>
    const Quill = require('quill')
    export default {
        props: {
          content: {},
          author: {},
          formats: {
            type: Array,
            default: () => [],
          },
          keyBindings: {
            type: Array,
            default: () => [],
          },
          output: { default : 'delta', },
          keyup: { default : null, },
          config: {
            type: Object,
            default: () => { return {} },
          },
        },

        data() {
          return {
            editor: {},
            editorContent: this.content,
            defaultConfig: {
              modules: {
                toolbar: [
                  [{ 'font': [] }],
                  [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
                  ['bold', 'italic', 'underline', 'strike'],
                  ['blockquote', 'code-block'],
                  [
                    { 'list': 'ordered' }, { 'list': 'bullet' }
                  ],
                  [
                    { 'script': 'sub' }, { 'script': 'super' }
                  ],
                  [
                    { 'indent': '-1' }, { 'indent': '+1' }
                  ],
                  [
                    { 'color': [] }, { 'background': [] }
                  ],
                  [{ 'align': [] }],
                  ['clean'],
                ],
              },
              theme: 'snow',
            },
          },
        },

        mounted() {
          this.editor = new Quill(this.$el, this.setOptions(this.defaultConfig, this.config))

          this.formats.map((format) => {
            this.editor.addFormat(format.name, format.options)
          })

          if (this.output !== 'delta') {
            this.editor.root.innerHTML = this.editorContent
          } else {
            this.editor.setContents(this.editorContent)
          }

          this.editor.on('text-change', (delta, source) => {
            this.$emit('text-change', this.editor, delta, source)
            this.editorContent = this.output !== 'delta' ? this.editor.root.innerHTML : this.editor.getContents()
          })

          this.editor.on('selection-change', (range) => {
            this.$emit('selection-change', this.editor, range)
          })

          if (typeof this.author !== 'undefined') {
            this.editor.addModule('authorship', {
              authorId: this.author,
            })
          }

          if (this.keyBindings.length) {
            const keyboard = this.editor.getModule('keyboard')

            this.keyBindings.map((binding) => {
              keyboard.addHotkey({ key: binding.key, metaKey: true }, binding.method.bind(this))
            })
          }
        },

        methods: {
            setOptions(defaults, properties) {
              for (let property in properties) {
                if (properties.hasOwnProperty(property)) {
                  if (typeof properties[property] === Object) {
                    defaults[property] = this.setOptions(defaults[property], properties[property]);
                  } else {
                    defaults[property] = properties[property]
                  }
                }
              }

              return defaults
            },
            focusEditor(e) {
                if (e && e.srcElement) {
                    let classList = e.srcElement.classList, isSegment = false

                    classList.forEach((className) => {
                        if (className === 'segment') {
                            isSegment = true
                            return
                        }
                    })

                    if (!isSegment) return
                }

                this.editor.focus()

                this.editor.setSelection(this.editor.getLength()-1, this.editor.getLength())
            },
            setContents(content) {
              this.editor.setContents(content)
            },
            setHTML(html) {
              this.editor.root.innerHTML = html
            }
        },
    }
</script>
