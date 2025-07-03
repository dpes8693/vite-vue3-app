<template>
  <div id="app">
    <div class="floating-buttons">
      <button :class="{ active: mode === 1 }" @click="onClick(1)">富文本模式</button>
      <button :class="{ active: mode === 2 }" @click="onClick(2)">無頭模式</button>
      <button :class="{ active: mode === 3 }" @click="onClick(3)">Notion模式</button>
    </div>
    <div>
      <h2>[tip-tap editor demo]</h2>
    </div>
    <br />
    <tiptap2 v-show="mode === 1" />
    <tiptap v-show="mode === 2" />
    <div v-show="mode === 3" class="notion-mode">
      <a href="https://github.com/CafeinoDev/tiptap-notion" target="_blank">source react+tip-tap</a>
      <iframe src="https://cafeinodev.github.io/tiptap-notion/" frameborder="0"></iframe>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Tiptap2 from './components/Tiptap2.vue'
import Tiptap from './components/Tiptap.vue'

const mode = ref(1)
const onClick = (v: number) => {
  mode.value = v
}
</script>

<style lang="scss">
.notion-mode {
  display: flex;
  flex-direction: column;

  iframe {
    height: 70dvh;
    width: 90dvh;
  }
}

.floating-buttons {
  position: fixed;
  top: 16px;
  left: 16px;
  z-index: 1000;
  display: flex;
  gap: 8px;
}

.floating-buttons button.active {
  background: #1976d2;
  color: #fff;
  border: 1.5px solid #1976d2;
  box-shadow: 0 2px 8px rgba(25, 118, 210, 0.15);
}
</style>