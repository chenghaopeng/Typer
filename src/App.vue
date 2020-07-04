<template>
  <div id="app">
    <codemirror ref="myCm" v-model="code" :options="cmOptions"></codemirror>
    <div class="footer">
      <div>由 PP 用 Vue 、 vue-codermirror 构建 <a href="https://github.com/chenghaopeng/Typer">GitHub</a></div>
      <div v-if="view === 0" class="frame">
        <button @click="handleClear">清空</button>
        <input v-model="speed" type="text" style="width: 30px"/>
        <input v-model="step" type="text"/>
        <button @click="handleStart">开始</button>
      </div>
      <div v-if="view !== 0" class="frame">
        <button @click="handleBack">返回</button>
        <button @click="handleNext" :disabled="running || !(now + 1 < steps.length)">继续</button>
      </div>
    </div>
  </div>
</template>

<script>
import { codemirror } from 'vue-codemirror'
require("codemirror/mode/clike/clike.js")
require('codemirror/addon/fold/foldcode.js')
require('codemirror/addon/fold/foldgutter.js')
require('codemirror/addon/fold/brace-fold.js')
require('codemirror/addon/fold/xml-fold.js')
require('codemirror/addon/fold/indent-fold.js')
require('codemirror/addon/fold/markdown-fold.js')
require('codemirror/addon/fold/comment-fold.js')
import 'codemirror/theme/mdn-like.css'
export default {
  name: 'App',
  components:{
    codemirror
  },
  data () {
    return {
      cmOptions: {
        tabSize: 4,
        mode: 'text/x-c++src',
        theme: 'mdn-like',
        lineNumbers: true,
        line: true,
      },
      view: 0,
      step: '',
      code: '',
      steps: '',
      codes: [],
      lines: [],
      now: -1,
      running: null,
      speed: 50
    }
  },
  methods: {
    handleClear () {
      this.step = this.code = ''
    },
    handleStart () {
      this.steps = []
      this.step.split(',').forEach(v => {
        this.steps.push([])
        v.split(' ').forEach(v => this.steps[this.steps.length - 1].push(parseInt(v)))
      })

      this.codes = this.code.split('\n')
      this.lines = []
      for (let i = 0; i < this.codes.length; ++i) {
        this.lines.push('')
      }

      this.now = -1
      this.code = ''
      this.view = 1
    },
    handleNext () {
      if (this.now + 1 >= this.steps.length) return
      this.now++
      this.running = setInterval(() => {
        for (let i = 0; i < this.steps[this.now].length; ++i) {
          let ln = this.steps[this.now][i] - 1
          if (this.lines[ln].length < this.codes[ln].length) {
            while (this.lines[ln].length < this.codes[ln].length && this.codes[ln][this.lines[ln].length].match(/\s/)) this.lines[ln] += this.codes[ln][this.lines[ln].length]
            if (this.lines[ln].length < this.codes[ln].length) {
              this.lines[ln] += this.codes[ln][this.lines[ln].length]
              this.code = this.lines.filter((line, ln) => { return line.length > 0 || this.codes[ln].length === 0 }).join('\n')
              return;
            }
          }
        }
        clearInterval(this.running)
        this.running = null
      }, this.speed)
    },
    handleBack () {
      this.view = 0
    }
  }
}
</script>

<style>
:root {
  font-size: 14px;
  font-family: Consolas, Menlo, Monaco, Lucida Console, Liberation Mono,
    DejaVu Sans Mono, Bitstream Vera Sans Mono, Courier New, monospace, serif;
}
html, body, #app {
  margin: 0;
  padding: 0;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height: 100vh;
  display: flex;
  flex-flow: column nowrap;
}
.vue-codemirror {
  flex: 1 0;
}
.vue-codemirror .CodeMirror {
  height: 100%;
}
.footer {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  align-items: center;
}
.frame {
  flex: none;
  display: flex;
  justify-content: flex-end;
}
</style>
