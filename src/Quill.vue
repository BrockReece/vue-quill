<template>
    <div>
        <div class="ui attached segment" ref="quill" @click.prevent="focusEditor"></div>
    </div>
</template>

<script>
    import defaultsDeep from 'lodash.defaultsdeep'
    import Quill from 'quill'
    import GrammarlyInline from './formats/GrammarlyInline'

    export default {
        model: {
            prop: 'content',
        },

        props: {
            content: {},

            formats: {
                type: Array,
                default() {
                    return []
                },
            },

            keyBindings: {
                type: Array,
                default() {
                    return []
                },
            },

            output: {
                default : 'delta'
            },

            bus: {
                default: false,
            },

            config: {
                default() {
                    return {}
                },
            },
        },

        data() {
            return {
                editor: {},
                defaultConfig: {
                    modules: {
                        toolbar: [
                            ['bold', 'italic', 'underline'],
                            [
                               { 'list': 'ordered' }, { 'list': 'bullet' }
                            ],
                            [{ 'align': [] }],
                        ],
                    },
                    theme: 'snow',
                },
            }
        },

        mounted() {
            if (this.keyBindings.length) {
                this.defaultConfig.modules.keyboard = {
                    bindings: this.keyBindings.map((binding) => {
                        if (binding.remove) return false
                        return {
                            key: binding.key,
                            metaKey: true,
                            handler: binding.method.bind(this),
                        }
                    })
                }
            }

            if (this.config.modules && this.config.modules.toolbar) {
                this.defaultConfig.modules.toolbar = []
            }

            Quill.register(GrammarlyInline)

            this.editor = new Quill(this.$refs.quill, defaultsDeep(this.config, this.defaultConfig))

            if (this.content && this.content !== '') {
	            if (this.output != 'delta') {
	                this.editor.pasteHTML(this.content)
	            } else {
	                this.editor.setContents(this.content)
	            }
            }

            this.editor.on('text-change', (delta, source) => {
                this.$emit('text-change', this.editor, delta, source)
                if (this.editor.getText().length <= 1) {
                  this.$emit('input', '')
                } else {
                  this.$emit('input', this.output != 'delta' ? this.editor.root.innerHTML : this.editor.getContents())
                }
            })

            this.editor.on('selection-change', (range) => {
                this.$emit('selection-change', this.editor, range)
            })

            if (this.bus) {
                this.bus.$on('focus-editor', () => this.focusEditor())
                this.bus.$on('set-content', (content) => this.editor.setContents(content))
                this.bus.$on('set-html', (html) => {
                    if (!html || html === '') return

                    this.editor.root.innerHTML = html
                })
            }

            this.$on('focus-editor', () => this.focusEditor())
            this.$on('set-content', (content) => this.editor.setContents(content))
            this.$on('set-html', (html) => {
                if (!html || html === '') return

                this.editor.root.innerHTML = html
            })

            this.$nextTick(() => {
                const selectors = ['button', '.ql-picker-label', '.ql-picker-item']
                const toolbar = this.$el.querySelector('.ql-toolbar')
                selectors.forEach((selector) => {
                    toolbar.querySelectorAll(selector).forEach((element) => {
                        element.tabIndex = -1
                    })
                })
            })
        },

        methods: {
            focusEditor(e) {
                if (e && e.srcElement) {
                    let classList = e.srcElement.classList,
                        isSegment = false

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
            }
        },

        beforeDestroy() {
            if (this.bus) {
                this.bus.$off('focus-editor')
                this.bus.$off('set-content')
                this.bus.$off('set-html')
            }
        },
    }
</script>
