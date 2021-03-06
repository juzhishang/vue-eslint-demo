<template>
    <div class="app__root">
        <div class="app__header">
            <div class="app__header-title">
                Playground for <a href="https://github.com/vuejs/eslint-plugin-vue#readme" target="_blank" rel="noopener">eslint-plugin-vue</a>.
            </div>
            <label class="app__header-option-item">
                <select v-model.number="indentSize">
                    <option value="2">TabSize: 2</option>
                    <option value="4">TabSize: 4</option>
                    <option value="8">TabSize: 8</option>
                </select>
            </label>
            <label class="app__header-option-item">
                <select v-model="indentType">
                    <option value="space">Indent: spaces</option>
                    <option value="tab">Indent: tabs</option>
                </select>
            </label>
            <label class="app__header-option-item">
                <select v-model="editorType">
                    <option value="codeAndFixedCode">FixedCode: show</option>
                    <option value="codeOnly">FixedCode: hide</option>
                </select>
            </label>
        </div>
        <div class="app__body">
            <configuration
                class="app__sidebar"
                :config="config"
                @change="onConfigChange"
            />
            <div class="app__main">
                <code-editor
                    class="app__editor"
                    :code="code"
                    :messages="messages"
                    :fixed-code="fixedCode"
                    :fixed-messages="fixedMessages"
                    :format-options="formatOptions"
                    :show-fixed-code="showFixedCode"
                    @edit="onCodeChange"
                    @initialize.once="onEditorInitialize"
                />
                <message-list
                    class="app__errors"
                    :messages="messages"
                />
            </div>
        </div>
        <div class="app__footer">
            <div
                class="app__version-item"
                v-for="(v, name) in versions"
                :key="name"
            >
                <a :href="'https://github.com/' + v.repo" target="_blank" rel="noopener">{{ name }}</a>
                v{{ v.version }}
            </div>
        </div>
        <div class="app__update-ready-toast" v-if="showUpdateReadyToast">
            <div>Please reload because a new version of this site is available.</div>
            <button @click="reload">Reload</button>
        </div>
    </div>
</template>

<script>
import AppState from "./app-state.js"
import CodeEditor from "./code-editor.vue"
import Configuration from "./configuration.vue"
import MessageList from "./message-list.vue"
import versions from "./versions.js"

export default {
    name: "App",

    components: {
        CodeEditor,
        MessageList,
        Configuration,
    },

    data() {
        return new AppState(location.hash.slice(1))
    },

    computed: {
        formatOptions() {
            return {
                insertSpaces: this.indentType === "space",
                tabSize: this.indentSize,
            }
        },

        showFixedCode() {
            return this.editorType === "codeAndFixedCode"
        },

        versions() {
            return versions
        },
    },

    watch: {
        code() {
            this.$data.lint()
            this.applyUrlHash()
        },
        indentSize() {
            this.$data.lint()
            this.applyUrlHash()
        },
        indentType() {
            this.$data.lint()
            this.applyUrlHash()
        },
        editorType() {
            this.applyUrlHash()
        },
    },

    mounted() {
        window.addEventListener("hashchange", this.onUrlHashChange)
    },
    beforeDestroey() {
        window.removeEventListener("hashchange", this.onUrlHashChange)
    },

    methods: {
        onEditorInitialize() {
            window.MainContent.show()
        },

        onCodeChange(code) {
            this.code = code
        },

        onConfigChange() {
            // The inside of `this.config` was changed directly.
            this.$data.lint()
            this.applyUrlHash()
        },

        onUrlHashChange() {
            this.$data.deserialize(location.hash.slice(1))
        },

        applyUrlHash() {
            location.replace(`#${this.$data.serialize()}`)
        },

        reload() {
            location.reload(false)
        },
    },
}
</script>

<style>
a {
    color: inherit;
}
a:hover {
    color: #2196F3;
}

select, button {
    font-family: inherit;
}

.app__root {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAQAAAAECAYAAACp8Z5+AAAAHElEQVQYV2O8e/fuf2VlZUYGKIAzMARgKjFUAABhrQgFvaGkawAAAABJRU5ErkJggg==) repeat;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
}

.app__header {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: flex-end;
    flex-shrink: 0;
    background-color: #A5D6A7;
    border-bottom: 1px solid #4CAF50;
}
.app__header-title {
    flex-grow: 1;
    padding: 8px;
    font-weight: bold;
}
.app__header-option-item {
    flex-shrink: 0;
    padding: 2px;
}
.app__header-option-item > select {
    padding: 4px;
    border: 1px solid #4CAF50;
    border-radius: 3px;
    background-color: #E8F5E9;
}

.app__body {
    display: flex;
    flex-grow: 1;
    min-height: 0;
}

.app__sidebar {
    width: calc(25% - 1px);
    border-right: 1px solid #CCC;
}

.app__main {
    display: flex;
    flex-direction: column;
    flex-grow: 1;
}

.app__editor {
    height: calc(75% - 1px);
    flex-grow: 1;
    border-bottom: 1px solid #CCC;
}

.app__errors {
    height: 25%;
}

.app__show-fixed-code-button {
    display: block;
    float: right;
    font-weight: normal;
}
.app__show-fixed-code-button > input[type=checkbox] {
    vertical-align: middle;
}

.app__footer {
    display: flex;
    justify-content: flex-end;
    flex-wrap: wrap;
    flex-shrink: 0;
    border-top: 1px solid #CCC;
    background-color: white;
}
.app__version-item {
    flex-shrink: 0;
    margin-right: 8px;
}
.app__version-item:not(:last-child)::after {
    content: ","
}

.app__update-ready-toast {
    display: block;
    position: absolute;
    right: 24px;
    bottom: 32px;
    width: 320px;
    padding: 8px;
    border: 1px solid #4CAF50;
    border-radius: 3px;
    background: white;
    box-shadow: 0 4px 16px rgba(0,0,0, 0.5);
    text-align: center;
    animation: AppToastIn 0.333s;
}
.app__update-ready-toast > div {
    text-align: left;
}
.app__update-ready-toast > button {
    margin-top: 8px;
    padding: 4px 32px;
}

@keyframes AppToastIn {
    0% {
        opacity: 0;
        transform: translate(0, 50%) scale(0.5);
    }
    100% {
        opacity: 1;
        transform: none;
    }
}
</style>
