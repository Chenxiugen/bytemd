<template>
  <div class="editor-container">
    <div class="editor-header">
      <div class="plugin-box">
        Plugins:
        <label v-for="p in Object.keys(enabled)" :key="p">
          <input
            type="checkbox"
            :checked="enabled[p]"
            @change="handlePluginChange($event, p)"
          />
          {{ p }}
        </label>
      </div>
      <div class="function-btns">
        <span class="tips" :class="tips.show && 'show'">{{ tips.text }}</span>
        <!-- <span class="btn" @click="submitGit">提交</span>
        <span class="btn" @click="saveLocal">保存草稿</span> -->
      </div>
    </div>

    <div class="editor-box">
      <Editor
        class="editor-content"
        :value="value"
        @change="autoSave"
        :plugins="enabledPlugins"
        :toolbarItems="[]"
      />
    </div>
  </div>
</template>

<script>
import { Editor, Viewer } from '@bytemd/vue'
import highlight from '@bytemd/plugin-highlight'
import math from '@bytemd/plugin-math'
import mermaid from '@bytemd/plugin-mermaid'
import gfm from '@bytemd/plugin-gfm'
import gemoji from '@bytemd/plugin-gemoji'
import { markdownText } from '../test'

const DraftKey = '_MD_DRAFT_'

export default {
  components: {
    Editor,
    Viewer,
  },
  data() {
    return {
      value: '',
      enabled: {
        highlight: true,
        math: true,
        mermaid: true,
        gemoji: true,
        GFM: true,
      },
      saveTimer: null,
      tips: {
        show: false,
        text: '自动保存',
        timer: null,
        limit: 1000,
      },
    }
  },
  mounted() {
    // 获取草稿信息
    this.value = this.getLocalDraft()
  },
  computed: {
    /** 启用的插件 */
    enabledPlugins() {
      return [
        this.enabled.highlight && highlight(),
        this.enabled.math && math(),
        this.enabled.mermaid && mermaid(),
        this.enabled.GFM && gfm(),
        this.enabled.gemoji && gemoji(),
      ]
    },
  },
  methods: {
    /** 自动保存本地 */
    autoSave(v) {
      this.value = v
      this.saveTimer && clearTimeout(this.saveTimer)
      this.saveTimer = setTimeout(this.saveLocal, 1000)
    },
    handlePluginChange($event, p) {
      this.enabled[p] = $event.target.checked
    },
    /** 保存草稿 */
    saveLocal() {
      sessionStorage &&
        sessionStorage.setItem(
          DraftKey,
          JSON.stringify({
            data: this.value,
          })
        )
      this.showTips()
    },
    /** 获取草稿 */
    getLocalDraft() {
      let output = markdownText
      const str = (sessionStorage && sessionStorage.getItem(DraftKey)) || ''
      try {
        output = JSON.parse(str).data
      } catch (error) {}
      return output
    },
    /** 提交git TODO */
    submitGit() {
      alert('待实现')
    },
    /** 限时提示信息 */
    showTips(text) {
      this.tips.text = text || '保存草稿...'
      this.tips.show = true
      this.tips.timer && clearTimeout(this.tips.timer)
      this.tips.timer = setTimeout(() => {
        this.tips.show = false
      }, this.tips.limit)
    },
  },
}
</script>

<style>
html {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
body {
  margin: 0 10px;
}
.editor-container {
  display: flex;
  flex-direction: column;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  padding: 10px;
}
.editor-box {
  position: relative;
  height: 100%;
  overflow: hidden;
}
.editor-content {
  height: 100%;
}
.bytemd {
  height: 100% !important;
}

.editor-header {
  display: flex;
  justify-content: space-between;
  height: 32px;
  margin-bottom: 10px;
}
.function-btns,
.plugin-box {
  align-self: center;
}
.tips {
  opacity: 0;
  font-size: 14px;
  color: #333333;
}
.tips.show {
  animation: tips-show 1s alternate-reverse ease;
}
.function-btns .btn {
  display: inline-block;
  height: 24px;
  padding: 0 10px;
  border-radius: 4px;
  cursor: pointer;
}

@keyframes tips-show {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
