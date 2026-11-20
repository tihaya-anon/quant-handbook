<script setup lang="ts">
import DefaultTheme from 'vitepress/theme'
import { computed, onMounted, ref, watch } from 'vue'
import { useData, withBase } from 'vitepress'
import { modules, lessonCount } from '../curriculum'

const { page, frontmatter } = useData()

// 阅读布局：可收起左侧课程目录 / 右侧大纲，给公式和图表让出宽度。
const hideSidebar = ref(false)
const hideAside = ref(false)
const isDoc = computed(() => !frontmatter.value.layout || frontmatter.value.layout === 'doc')
const STORE_KEY = 'quant-handbook:layout'

function applyLayout() {
  const root = document.documentElement
  root.classList.toggle('hide-sidebar', isDoc.value && hideSidebar.value)
  root.classList.toggle('hide-aside', isDoc.value && hideAside.value)
}
function persist() {
  try { localStorage.setItem(STORE_KEY, JSON.stringify({ sidebar: hideSidebar.value, aside: hideAside.value })) } catch {}
}
function toggleFocus() {
  const next = !(hideSidebar.value && hideAside.value)
  hideSidebar.value = next
  hideAside.value = next
}
function onKey(e: KeyboardEvent) {
  const t = e.target as HTMLElement | null
  if (e.ctrlKey || e.metaKey || e.altKey || t?.closest('input, textarea, [contenteditable]')) return
  if (e.key === '[') hideSidebar.value = !hideSidebar.value
  else if (e.key === ']') hideAside.value = !hideAside.value
  else if (e.key === '\\') toggleFocus()
  else return
  e.preventDefault()
}

onMounted(() => {
  try {
    const saved = JSON.parse(localStorage.getItem(STORE_KEY) || '{}')
    hideSidebar.value = !!saved.sidebar
    hideAside.value = !!saved.aside
  } catch {}
  applyLayout()
  window.addEventListener('keydown', onKey)
  watch([hideSidebar, hideAside], () => { applyLayout(); persist() })
  watch(isDoc, applyLayout)
})
const currentModule = computed(() => modules.find(module => module.lessons.some(
  lesson => `${lesson.link.slice(1)}.md` === page.value.relativePath
)))
const currentLesson = computed(() => currentModule.value?.lessons.find(
  lesson => `${lesson.link.slice(1)}.md` === page.value.relativePath
))
</script>

<template>
  <DefaultTheme.Layout>
    <template #nav-bar-title-after><span class="brand-caption">量化研究手册</span></template>
    <template #sidebar-nav-before>
      <a class="sidebar-heading" :href="withBase('/#curriculum')">
        <span class="eyebrow">COURSE HANDBOOK</span>
        <span>课程目录 <span class="sidebar-count">{{ modules.length }} 模块 / {{ lessonCount }} 讲</span></span>
      </a>
    </template>
    <template #nav-bar-content-after>
      <div v-if="isDoc" class="layout-toggles" role="group" aria-label="阅读布局">
        <button type="button" :aria-pressed="!hideSidebar" title="显示课程目录 ( [ )" @click="hideSidebar = !hideSidebar">
          <svg viewBox="0 0 24 24" aria-hidden="true"><rect x="3" y="4" width="18" height="16" rx="2"/><path d="M9 4v16"/></svg>
          <span class="sr-only">课程目录</span>
        </button>
        <button type="button" :aria-pressed="!hideAside" title="显示本页大纲 ( ] )" @click="hideAside = !hideAside">
          <svg viewBox="0 0 24 24" aria-hidden="true"><rect x="3" y="4" width="18" height="16" rx="2"/><path d="M15 4v16"/></svg>
          <span class="sr-only">本页大纲</span>
        </button>
        <button type="button" :aria-pressed="hideSidebar && hideAside" title="专注阅读：同时收起两侧 ( \ )" @click="toggleFocus">
          <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M4 9V4h5M20 9V4h-5M4 15v5h5M20 15v5h-5"/></svg>
          <span class="sr-only">专注阅读</span>
        </button>
      </div>
    </template>
    <template #doc-before>
      <div v-if="currentModule && currentLesson" class="lesson-context">
        <nav class="lesson-breadcrumb" aria-label="当前位置">
          <a :href="withBase('/#curriculum')">课程总览</a>
          <span aria-hidden="true">/</span>
          <a :href="withBase(`/#module-${currentModule.id}`)">{{ currentModule.title }}</a>
        </nav>
        <div class="lesson-meta">
          <span class="lesson-number">LECTURE {{ String(currentLesson.number).padStart(2, '0') }}</span>
          <span>共 {{ lessonCount }} 讲</span>
          <span v-if="frontmatter.reading_time" class="reading-time">预计阅读 {{ frontmatter.reading_time }}</span>
        </div>
      </div>
      <div v-else class="guide-context"><span class="eyebrow">QUANTML / LEARNING GUIDE</span></div>
    </template>
    <template #aside-outline-before>
      <div v-if="currentModule" class="aside-module">
        <span class="eyebrow">MODULE {{ currentModule.id }}</span>
        <strong>{{ currentModule.title }}</strong>
        <span>{{ currentModule.lessons.length }} 讲 · {{ currentModule.en }}</span>
      </div>
    </template>
    <template #aside-outline-after>
      <div class="aside-resources">
        <span>配套学习资料</span>
        <a :href="withBase('/resources.html')">公开教材与工具文档 <span aria-hidden="true">↗</span></a>
        <a :href="withBase('/learning_path.html')">学习路径与掌握标准 <span aria-hidden="true">↗</span></a>
      </div>
    </template>
  </DefaultTheme.Layout>
</template>
