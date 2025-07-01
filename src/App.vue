<template>
  <div id="app">
    <button @click="onClick(1)">無頭模式</button>
    <button @click="onClick(2)">富文本模式</button>
{{ mode }}
    <tiptap2 v-if="mode==1"/>
    <tiptap v-if="mode==2"/>
  </div>
</template>

<script lang="ts">
import { ref } from 'vue'
  import Tiptap2 from './components/Tiptap2.vue'
  import Tiptap from './components/Tiptap.vue'

const mode = ref(1)
const onClick = (v:number) => {
  mode.value = v
}

  export default {
    name: 'App',
    components: {
      Tiptap2,
      Tiptap,
    },
  }
</script>
