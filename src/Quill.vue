    <template>
        <div>
            <div v-el:toolbar class="ui top attached menu toolbar ql-toolbar ql-snow">
                <slot name="toolbar">
                    <div class="ql-format-group">
                        <a class="ql-format-button ql-bold"></a>
                        <span class="ql-format-separator"></span>
                        <a class="ql-format-button ql-underline"></a>
                        <span class="ql-format-separator"></span>
                        <a class="ql-format-button ql-italic"></a>
                    </div>
                    <div class="ql-format-group">
                        <a class="ql-format-button ql-list"></a>
                        <span class="ql-format-separator"></span>
                        <a class="ql-format-button ql-bullet"></a>
                        <span class="ql-format-separator"></span>
                        <span title="Text Alignment" class="ql-align ql-picker">
                            <span class="ql-picker-label" data-value="left"></span>
                            <span class="ql-picker-options">
                                <span data-value="left" class="ql-picker-item ql-selected"></span>
                                <span data-value="center" class="ql-picker-item"></span>
                                <span data-value="right" class="ql-picker-item"></span>
                                <span data-value="justify" class="ql-picker-item"></span>
                            </span>
                        </span>
                    </div>
                </slot>
            </div>
            <div class="ui attached segment" v-el:quill @click.prevent="focusEditor"></div>
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
                    default() {
                        return []
                    }
                },
                keyBindings: {
                    type: Array,
                    default() {
                        return []
                    }
                },
                output : {
                    default : 'delta'
                },
                keyup : {
                    default : null
                }
            },

            data() {
                return {
                    editor: {}
                }
            },

            watch: {
                // Whenever 'content' on this vm updates, communicate that down to Quill.
                content(newValue) {
                    if (JSON.stringify(newValue) !== JSON.stringify(this.getEditorContents())) {
                        this.updateEditorContents()
                    }
                }
            },

            ready() {
                this.editor = new Quill(this.$els.quill, {
                    modules: { toolbar: this.$els.toolbar },
                    theme: 'snow'
                })

                this.formats.map((format) => {
                    this.editor.addFormat(format.name, format.options)
                })

                this.updateEditorContents()

                this.editor.on('text-change', (delta, source) => {
                    this.$dispatch('text-change', this.editor, delta, source)
                    this.content = this.getEditorContents()
                })

                this.editor.on('selection-change', (range) => {
                    this.$dispatch('selection-change', this.editor, range)
                })

                if (typeof this.author !== 'undefined') {
                    this.editor.addModule('authorship', {
                        authorId: this.author
                    })
                }

                if (this.keyBindings.length) {
                    const keyboard = this.editor.getModule('keyboard')

                    this.keyBindings.map((binding) => {
                        keyboard.addHotkey({ key: binding.key, metaKey: true }, binding.method.bind(this))
                    })
                }
            },

            events: {
                'set-content': function (content) {
                    this.setEditorOps(content)
                },

                'set-html': function (html) {
                    this.setEditorHtml(html)
                },

                'focus-editor': function () {
                    this.focusEditor()
                }
            },

            methods: {
                getEditorContents() {
                    return this.output != 'delta' ? this.editor.getHTML() : this.editor.getContents()
                },

                updateEditorContents() {
                    if (this.output != 'delta') {
                        this.setEditorHtml(this.content)
                    } else {
                        this.setEditorOps(this.content)
                    }
                },

                setEditorOps(ops) {
                    this.editor.setContents(ops)
                },

                setEditorHtml(html) {
                    this.editor.setHTML(html)
                },

                focusEditor(e) {

                    if (e && e.srcElement) {
                        let classList = e.srcElement.classList, isSegment = false;

                        classList.forEach((className) => {
                            if (className === 'segment') {
                                isSegment = true
                                return
                            }
                        })

                        if (!isSegment) return;
                    }

                    this.editor.focus()

                    this.editor.setSelection(this.editor.getLength()-1, this.editor.getLength())


                }
            }
        }
    </script>
