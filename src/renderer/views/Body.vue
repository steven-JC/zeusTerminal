<template>
  <section class="term-body">
    <div class="terminals-wrapper"
         flex="dir:top box:first">
      <TerminalsHeader></TerminalsHeader>
      <div class="terminals"
           :style="{
                    overflow: terminalHeight > minHeight ? 'hidden' : 'auto'
                }"
           :class="`column-${columns}`">
        <Tabs v-if="columns === 1" />
        <section class="terminal-box">
          <CTerminal v-for="item in terminals"
                     v-if="!item.type"
                     :height="
                            columns === 1 && focused.includes(item.id)
                                ? maxHeight
                                : terminalHeight
                        "
                     :columns="columns"
                     :term="item"
                     :key="item.id" />
        </section>
      </div>
    </div>
    <CEditer v-if="scriptShow"
             :class="scriptShow === 2 ? `show` : ''" />
  </section>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'
import CEditer from './components/Editor.vue'
import CTerminal from './components/Terminal/Terminal.vue'
import { State } from 'motx/dist/motx-vue'
import motx from '@/motx'
import TerminalsHeader from './components/TerminalsHeader.vue'
import Tabs from './components/Tabs.vue'
import Draggable from 'vuedraggable'

@Component({
    components: {
        CEditer,
        CTerminal,
        TerminalsHeader,
        Tabs,
        Draggable
    }
})
export default class Body extends Vue {
    @State('terminals') terminals: PlainObject[] = []
    @State('columns') columns: number = 1
    @State('focused') focused: number[] = []

    protected winHeight: number = window.innerHeight
    protected handlers: PlainObject = {}
    protected scriptShow: number = 0
    protected minHeight: number = 240
    protected maxHeight: number = 240

    protected get terminalHeight() {
        const len = this.terminals.length
        const winHeight = this.winHeight
        const columns = this.columns
        const rows = Math.ceil(len / columns)
        this.maxHeight = winHeight - 51
        const height =
            (winHeight - 51) / rows > this.minHeight
                ? (winHeight - 51) / rows
                : this.minHeight
        return height
    }

    mounted() {
        let scriptShowTimo
        this.handlers.handleScriptShow = (show) => {
            if (scriptShowTimo) {
                clearTimeout(scriptShowTimo)
                scriptShowTimo = null
            }
            if (show) {
                this.scriptShow = 1
                scriptShowTimo = setTimeout(() => {
                    this.scriptShow = 2
                    scriptShowTimo = null
                }, 10)
            } else {
                this.scriptShow = 3
                scriptShowTimo = setTimeout(() => {
                    this.scriptShow = 0
                }, 500)
            }
        }

        this.handlers.terminalFit = () => {
            this.winHeight = window.innerHeight
        }

        this.terminals = motx.getState('terminals')
        this.columns = motx.getState('columns')
        motx.subscribe('terminal-fit', this.handlers.terminalFit)
        motx.subscribe('toggle-script', this.handlers.handleScriptShow)
        setTimeout(() => {
            motx.setState('focused', [this.terminals[0].id])
        }, 500)
    }

    protected beforeDestroy() {
        motx.unsubscribe('terminal-fit', this.handlers.terminalFit)
        motx.unsubscribe('toggle-script', this.handlers.handleScriptShow)
    }

    protected onDragEnd(e) {
        console.log(JSON.parse(JSON.stringify(this.terminals)))
        motx.publish('save-terminals', this.terminals)
    }
}
</script>

<style lang="stylus">
.ghost-terminal
  opacity 0.7
.term-body
  .terminals-wrapper
    height 100%
  .terminals
    width 100%
    height 100%
    overflow auto
    &.column-1
      .terminal-box
        height calc(100% - 25px)
        width 100%
        position relative
        .terminal-wrapper
          width 100%
          height 100%
          min-height 100%
          float none
          overflow auto
          z-index 0
          position absolute
          left 0
          right 0
          top 0
          bottom 0
          .xterm-header
            position absolute
            z-index 10000
            .left > div
              display none
          &.focus
            width 100%
            height 100%
            min-height 100%
            z-index 1
    &.column-2
      .terminal-wrapper
        width 50%
    &.column-3
      .terminal-wrapper
        width 33.333%
    &.column-4
      .terminal-wrapper
        width 25%
    &>div
      min-height 100%
      display inline-block
      width 100%
</style>
