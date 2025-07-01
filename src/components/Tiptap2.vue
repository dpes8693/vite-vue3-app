<template>
  <div>
    <!-- Toolbar -->
    <div ref="toolbarRef" class="toolbar" :style="toolbarStyle">
      <!-- Main Toolbar Content -->
      <template v-if="mobileView === 'main'">
        <div class="spacer"></div>

        <!-- Undo/Redo Group -->
        <div class="toolbar-group">
          <button class="toolbar-button" @click="editor?.chain().focus().undo().run()" :disabled="!editor?.can().undo()"
            aria-label="Undo">
            ↶
          </button>
          <button class="toolbar-button" @click="editor?.chain().focus().redo().run()" :disabled="!editor?.can().redo()"
            aria-label="Redo">
            ↷
          </button>
        </div>

        <div class="toolbar-separator"></div>

        <!-- Format Group -->
        <div class="toolbar-group">
          <!-- Heading Dropdown -->
          <select class="toolbar-select" :value="currentHeading" @change="setHeading(($event.target as HTMLSelectElement).value)">
            <option value="">Paragraph</option>
            <option value="1">Heading 1</option>
            <option value="2">Heading 2</option>
            <option value="3">Heading 3</option>
            <option value="4">Heading 4</option>
          </select>

          <!-- List Dropdown -->
          <select class="toolbar-select" :value="currentList" @change="setList(($event.target as HTMLSelectElement).value)">
            <option value="">No List</option>
            <option value="bulletList">Bullet List</option>
            <option value="orderedList">Ordered List</option>
            <!-- <option value="taskList">Task List</option>-->
          </select>

          <button class="toolbar-button" :class="{ active: editor?.isActive('blockquote') }"
            @click="editor?.chain().focus().toggleBlockquote().run()" title="Blockquote">
            "
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive('codeBlock') }"
            @click="editor?.chain().focus().toggleCodeBlock().run()" title="Code Block">
            &lt;/&gt;
          </button>
        </div>

        <div class="toolbar-separator"></div>

        <!-- Text Style Group -->
        <div class="toolbar-group">
          <button class="toolbar-button" :class="{ active: editor?.isActive('bold') }"
            @click="editor?.chain().focus().toggleBold().run()" title="Bold">
            <strong>B</strong>
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive('italic') }"
            @click="editor?.chain().focus().toggleItalic().run()" title="Italic">
            <em>I</em>
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive('strike') }"
            @click="editor?.chain().focus().toggleStrike().run()" title="Strike">
            <s>S</s>
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive('code') }"
            @click="editor?.chain().focus().toggleCode().run()" title="Inline Code">
            `code`
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive('underline') }"
            @click="editor?.chain().focus().toggleUnderline().run()" title="Underline">
            <u>U</u>
          </button>

          <!-- Highlight Button -->
          <button v-if="!isMobile" class="toolbar-button" :class="{ active: editor?.isActive('highlight') }"
            @click="toggleHighlight" title="Highlight">
            🖍
          </button>
          <button v-else class="toolbar-button" @click="mobileView = 'highlighter'" title="Highlight">
            🖍
          </button>

          <!-- Link Button -->
          <button v-if="!isMobile" class="toolbar-button" :class="{ active: editor?.isActive('link') }"
            @click="toggleLink" title="Link">
            🔗
          </button>
          <button v-else class="toolbar-button" @click="mobileView = 'link'" title="Link">
            🔗
          </button>
        </div>

        <div class="toolbar-separator"></div>

        <!-- Superscript/Subscript Group -->
        <div class="toolbar-group">
          <button class="toolbar-button" :class="{ active: editor?.isActive('superscript') }"
            @click="editor?.chain().focus().toggleSuperscript().run()" title="Superscript">
            x²
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive('subscript') }"
            @click="editor?.chain().focus().toggleSubscript().run()" title="Subscript">
            x₂
          </button>
        </div>

        <div class="toolbar-separator"></div>

        <!-- Text Align Group -->
        <div class="toolbar-group">
          <button class="toolbar-button" :class="{ active: editor?.isActive({ textAlign: 'left' }) }"
            @click="editor?.chain().focus().setTextAlign('left').run()" title="Align Left">
            ⬅
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive({ textAlign: 'center' }) }"
            @click="editor?.chain().focus().setTextAlign('center').run()" title="Align Center">
            ↔
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive({ textAlign: 'right' }) }"
            @click="editor?.chain().focus().setTextAlign('right').run()" title="Align Right">
            ➡
          </button>

          <button class="toolbar-button" :class="{ active: editor?.isActive({ textAlign: 'justify' }) }"
            @click="editor?.chain().focus().setTextAlign('justify').run()" title="Justify">
            ≡
          </button>
        </div>

        <div class="toolbar-separator"></div>

        <!-- Image Upload Group -->
        <div class="toolbar-group">
          <input ref="imageInput" type="file" accept="image/*" style="display: none" @change="handleImageUpload">
          <button class="toolbar-button" @click="($refs.imageInput as HTMLInputElement | undefined)?.click()" title="Add Image">
            📷 Add
          </button>
        </div>

        <div class="spacer"></div>

        <div v-if="isMobile" class="toolbar-separator"></div>

        <!-- Theme Toggle Group -->
        <div class="toolbar-group">
          <button class="toolbar-button" @click="toggleTheme"
            :aria-label="`Switch to ${isDarkMode ? 'light' : 'dark'} mode`" title="Toggle Theme">
            {{ isDarkMode ? '🌙' : '☀️' }}
          </button>
        </div>
      </template>

      <!-- Mobile Toolbar Content -->
      <template v-else>
        <div class="toolbar-group">
          <button class="toolbar-button" @click="mobileView = 'main'" title="Back">
            ← {{ mobileView === 'highlighter' ? '🖍' : '🔗' }}
          </button>
        </div>

        <div class="toolbar-separator"></div>

        <!-- Highlighter Options -->
        <div v-if="mobileView === 'highlighter'" class="toolbar-group">
          <button class="toolbar-button" @click="editor?.chain().focus().toggleHighlight().run()"
            title="Yellow Highlight">
            🟡
          </button>
          <button class="toolbar-button" @click="editor?.chain().focus().toggleHighlight({ color: '#ff6b6b' }).run()"
            title="Red Highlight">
            🔴
          </button>
          <button class="toolbar-button" @click="editor?.chain().focus().toggleHighlight({ color: '#4ecdc4' }).run()"
            title="Green Highlight">
            🟢
          </button>
          <button class="toolbar-button" @click="editor?.chain().focus().unsetHighlight().run()"
            title="Remove Highlight">
            ❌
          </button>
        </div>

        <!-- Link Options -->
        <div v-else-if="mobileView === 'link'" class="toolbar-group">
          <input v-model="linkUrl" type="url" placeholder="Enter URL" class="toolbar-input" @keyup.enter="setLink">
          <button class="toolbar-button" @click="setLink" title="Set Link">
            ✓
          </button>
          <button class="toolbar-button" @click="editor?.chain().focus().unsetLink().run()" title="Remove Link">
            ❌
          </button>
        </div>
      </template>
    </div>

    <!-- Editor Content -->
    <div class="content-wrapper">
      <EditorContent :editor="editor" class="simple-editor-content" role="presentation" />
    </div>
  </div>
</template>

<script setup lang="ts">
import type { CSSProperties } from 'vue'
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import { useEditor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Image from '@tiptap/extension-image'
import TaskItem from '@tiptap/extension-task-item'
import TextAlign from '@tiptap/extension-text-align'
import Typography from '@tiptap/extension-typography'
import Highlight from '@tiptap/extension-highlight'
import Subscript from '@tiptap/extension-subscript'
import Superscript from '@tiptap/extension-superscript'
import Underline from '@tiptap/extension-underline'
import Link from '@tiptap/extension-link'
// import "./simple-editor.scss"

// Reactive data
const toolbarRef = ref<HTMLElement>()
const imageInput = ref<HTMLInputElement | null>(null)
const linkUrl = ref('')
const isDarkMode = ref(false)
const mobileView = ref<'main' | 'highlighter' | 'link'>('main')
const windowSize = ref({ height: window.innerHeight, width: window.innerWidth })

// Computed properties
const isMobile = computed(() => windowSize.value.width < 768)

const toolbarStyle = computed<CSSProperties>(() => {
  if (isMobile.value) {
    return {
      position: 'fixed',
      bottom: 0,
      left: 0,
      right: 0,
      zIndex: 1000
    }
  }
  return {}
})

const currentHeading = computed(() => {
  if (!editor.value) return ''
  for (let level = 1; level <= 4; level++) {
    if (editor.value.isActive('heading', { level })) {
      return level.toString()
    }
  }
  return ''
})

const currentList = computed(() => {
  if (!editor.value) return ''
  if (editor.value.isActive('bulletList')) return 'bulletList'
  if (editor.value.isActive('orderedList')) return 'orderedList'
  if (editor.value.isActive('taskList')) return 'taskList'
  return ''
})

// Editor instance
const editor = useEditor({
  content: `
    <h1>Welcome to the Simple Editor</h1>
    <p>This is a basic example of a Tiptap editor converted to Vue 3. You can:</p>
    <ul>
      <li>Format text with <strong>bold</strong>, <em>italic</em>, and other styles</li>
      <li>Create lists and headers</li>
      <li>Add links and images</li>
      <li>Use keyboard shortcuts</li>
    </ul>
    <p>Try typing something!</p>
  `,
  extensions: [
    StarterKit,
    TextAlign.configure({
      types: ['heading', 'paragraph'],
    }),
    Underline,
    //TaskList,
    TaskItem.configure({
      nested: true,
    }),
    Highlight.configure({
      multicolor: true,
    }),
    Image,
    Typography,
    Superscript,
    Subscript,
    Link.configure({
      openOnClick: false,
    }),
  ],
  editorProps: {
    attributes: {
      autocomplete: 'off',
      autocorrect: 'off',
      autocapitalize: 'off',
      'aria-label': 'Main content area, start typing to enter text.',
    },
  },
})

// Methods
const setHeading = (level: string) => {
  if (!editor.value) return
  if (level === '') {
    editor.value.chain().focus().setParagraph().run()
  } else {
    // 轉型為 Level 型別 (1|2|3|4)
    editor.value.chain().focus().toggleHeading({ level: parseInt(level) as 1 | 2 | 3 | 4 }).run()
  }
}

const setList = (type: string) => {
  if (!editor.value) return
  switch (type) {
    case 'bulletList':
      editor.value.chain().focus().toggleBulletList().run()
      break
    case 'orderedList':
      editor.value.chain().focus().toggleOrderedList().run()
      break
    default:
      if (editor.value.isActive('bulletList')) {
        editor.value.chain().focus().toggleBulletList().run()
      } else if (editor.value.isActive('orderedList')) {
        editor.value.chain().focus().toggleOrderedList().run()
      }
  }
}

const toggleHighlight = () => {
  if (!editor.value) return
  editor.value.chain().focus().toggleHighlight().run()
}

const toggleLink = () => {
  if (!editor.value) return

  if (editor.value.isActive('link')) {
    editor.value.chain().focus().unsetLink().run()
  } else {
    const url = window.prompt('Enter URL:')
    if (url) {
      editor.value.chain().focus().setLink({ href: url }).run()
    }
  }
}

const setLink = () => {
  if (!editor.value || !linkUrl.value) return

  editor.value.chain().focus().setLink({ href: linkUrl.value }).run()
  linkUrl.value = ''
  mobileView.value = 'main'
}

const handleImageUpload = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]

  if (!file || !editor.value) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const src = e.target?.result as string
    editor.value?.chain().focus().setImage({ src }).run()
  }
  reader.readAsDataURL(file)

  // Reset input
  target.value = ''
}

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  document.documentElement.classList.toggle('dark', isDarkMode.value)
}

const updateWindowSize = () => {
  windowSize.value = {
    height: window.innerHeight,
    width: window.innerWidth
  }
}

// Theme detection
const detectTheme = () => {
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
  const hasDarkMeta = !!document.querySelector('meta[name="color-scheme"][content="dark"]')
  isDarkMode.value = hasDarkMeta || prefersDark
  document.documentElement.classList.toggle('dark', isDarkMode.value)
}

// Lifecycle
onMounted(() => {
  detectTheme()
  window.addEventListener('resize', updateWindowSize)

  // Listen for system theme changes
  const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
  const handleThemeChange = () => {
    if (!document.documentElement.classList.contains('dark') &&
      !document.documentElement.classList.contains('light')) {
      isDarkMode.value = mediaQuery.matches
      document.documentElement.classList.toggle('dark', isDarkMode.value)
    }
  }
  mediaQuery.addEventListener('change', handleThemeChange)

  // Cleanup function will be called in onUnmounted
  return () => {
    mediaQuery.removeEventListener('change', handleThemeChange)
  }
})

onUnmounted(() => {
  window.removeEventListener('resize', updateWindowSize)
  editor.value?.destroy()
})

// Watch for mobile view changes
watch(isMobile, (newIsMobile) => {
  if (!newIsMobile && mobileView.value !== 'main') {
    mobileView.value = 'main'
  }
})
</script>

<style scoped>
/* CSS Variables */
:global(:root) {
  --tt-toolbar-height: 44px;
  --tt-theme-text: #1a1a1a;
  --tt-bg-color: #ffffff;
  --tt-border-color: #e5e7eb;
  --tt-scrollbar-color: #d1d5db;
}

:global(.dark) {
  --tt-theme-text: #ffffff;
  --tt-bg-color: #1a1a1a;
  --tt-border-color: #374151;
  --tt-scrollbar-color: #4b5563;
}

/* Toolbar Styles */
.toolbar {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 8px 16px;
  border-bottom: 1px solid var(--tt-border-color);
  background-color: var(--tt-bg-color);
  min-height: var(--tt-toolbar-height);
  overflow-x: auto;
}

.toolbar-group {
  display: flex;
  align-items: center;
  gap: 2px;
}

.toolbar-separator {
  width: 1px;
  height: 24px;
  background-color: var(--tt-border-color);
  margin: 0 4px;
}

.spacer {
  flex: 1;
}

.toolbar-button {
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 32px;
  height: 32px;
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  background: transparent;
  color: var(--tt-theme-text);
  cursor: pointer;
  transition: background-color 0.2s;
  font-size: 14px;
  white-space: nowrap;
}

.toolbar-button:hover {
  background-color: var(--tt-border-color);
}

.toolbar-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.toolbar-button.active {
  background-color: var(--tt-border-color);
  font-weight: 600;
}

.toolbar-select {
  padding: 4px 8px;
  border: 1px solid var(--tt-border-color);
  border-radius: 4px;
  background-color: var(--tt-bg-color);
  color: var(--tt-theme-text);
  font-size: 14px;
  cursor: pointer;
}

.toolbar-input {
  padding: 4px 8px;
  border: 1px solid var(--tt-border-color);
  border-radius: 4px;
  background-color: var(--tt-bg-color);
  color: var(--tt-theme-text);
  font-size: 14px;
  min-width: 200px;
}

/* Content Styles */
.content-wrapper {
  height: calc(100vh - var(--tt-toolbar-height));
  overflow-y: auto;
  background-color: var(--tt-bg-color);
}

.content-wrapper::-webkit-scrollbar {
  width: 8px;
}

.content-wrapper::-webkit-scrollbar-track {
  background: transparent;
}

.content-wrapper::-webkit-scrollbar-thumb {
  background-color: var(--tt-scrollbar-color);
  border-radius: 4px;
}

.simple-editor-content {
  max-width: 640px;
  width: 100%;
  margin: 0 auto;
}

:deep(.ProseMirror) {
  padding: 3rem;
  outline: none;
  font-family: 'Inter', sans-serif;
  color: var(--tt-theme-text);
  line-height: 1.6;
}

@media screen and (max-width: 480px) {
  :deep(.ProseMirror) {
    padding: 1rem 1.5rem;
  }

  .toolbar {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 1000;
    border-top: 1px solid var(--tt-border-color);
    border-bottom: none;
  }

  .content-wrapper {
    height: calc(100vh - var(--tt-toolbar-height));
    padding-bottom: var(--tt-toolbar-height);
  }
}

/* Editor Content Styles */
:deep(.ProseMirror h1) {
  font-size: 2rem;
  font-weight: 700;
  margin-top: 2rem;
  margin-bottom: 1rem;
  line-height: 1.2;
}

:deep(.ProseMirror h2) {
  font-size: 1.5rem;
  font-weight: 600;
  margin-top: 1.5rem;
  margin-bottom: 0.75rem;
  line-height: 1.3;
}

:deep(.ProseMirror h3) {
  font-size: 1.25rem;
  font-weight: 600;
  margin-top: 1.25rem;
  margin-bottom: 0.5rem;
  line-height: 1.4;
}

:deep(.ProseMirror h4) {
  font-size: 1.125rem;
  font-weight: 600;
  margin-top: 1rem;
  margin-bottom: 0.5rem;
  line-height: 1.4;
}

:deep(.ProseMirror p) {
  margin-bottom: 1rem;
}

:deep(.ProseMirror ul, .ProseMirror ol) {
  margin-bottom: 1rem;
  padding-left: 1.5rem;
}

:deep(.ProseMirror li) {
  margin-bottom: 0.25rem;
}

:deep(.ProseMirror blockquote) {
  border-left: 4px solid var(--tt-border-color);
  padding-left: 1rem;
  margin: 1rem 0;
  font-style: italic;
}

:deep(.ProseMirror pre) {
  background-color: var(--tt-border-color);
  border-radius: 4px;
  padding: 1rem;
  margin: 1rem 0;
  overflow-x: auto;
}

:deep(.ProseMirror code) {
  background-color: var(--tt-border-color);
  padding: 0.125rem 0.25rem;
  border-radius: 2px;
  font-size: 0.875em;
}

:deep(.ProseMirror pre code) {
  background: none;
  padding: 0;
}

:deep(.ProseMirror img) {
  max-width: 100%;
  height: auto;
  border-radius: 4px;
  margin: 1rem 0;
}

:deep(.ProseMirror mark) {
  background-color: #fef08a;
  padding: 0.125rem 0.25rem;
  border-radius: 2px;
}

:deep(.ProseMirror a) {
  color: #3b82f6;
  text-decoration: underline;
}

:deep(.ProseMirror a:hover) {
  color: #1d4ed8;
}

:deep(.dark .ProseMirror a) {
  color: #60a5fa;
}

:deep(.dark .ProseMirror a:hover) {
  color: #93c5fd;
}
</style>