<script setup>
import { computed, nextTick, onMounted, ref, watch } from 'vue'
import WhiteboardView from './components/WhiteboardView.vue'

const APP_STATE_KEY = 'draft-paper-app-state'
const APP_LOCALE_KEY = 'draft-paper-locale'
const APP_SCHEMA_VERSION = 1
const SUPPORTED_LOCALES = ['zh-CN', 'en-US']

const appCopy = {
  'zh-CN': {
    untitledDraft: '未命名草稿',
    justNow: '刚刚',
    editorIdle: '未开始编辑',
    localMode: '本地模式',
    lastSavedAt: '最近保存 {time}',
    draftCreated: '已创建新草稿，等待书写',
    restoredDraft: '已恢复本地草稿，{time}',
    blankDraftOpened: '空白草稿已打开',
    draftPrefix: '草稿',
    importDraft: '导入草稿',
    importingFile: '正在导入 {name}...',
    workbenchReady: '工作台已就绪',
    deleteConfirm: '确认删除“{title}”吗？该草稿的本地内容也会一起清除。',
    draftDeleted: '草稿已删除',
    draftCopySuffix: '副本',
    autosavedAt: '已自动保存，{time}',
    emptyHint: '新建一个草稿，或者导入已有草稿文件',
    appTitleSuffix: 'Draft Paper',
    eyebrow: '单人版线上草稿纸',
    heroTitle: '打开就写，离开也能回来。',
    heroBody: '这是一版纯前端、本地保存的草稿纸工作台。它保留了现有白板能力，并补上最近草稿、继续编辑、导入导出和使用说明，让它更接近真正可用的个人工具。',
    newDraft: '新建草稿',
    importDraftButton: '导入草稿',
    shortcuts: '查看快捷键',
    localeLabel: '语言',
    localeChinese: '中文',
    localeEnglish: 'English',
    draftCountUnit: '份本地草稿',
    contentCountUnit: '份已有内容',
    localStorageNote: '所有数据都保存在当前浏览器本地。导出文件后可离线备份，再导入恢复。',
    recentDrafts: '最近草稿',
    recentDraftsHint: '从最近编辑记录继续，不需要重新找入口。',
    import: '导入',
    performanceHigh: '高性能',
    performanceBalanced: '平衡',
    strokeCount: '共 {count} 条笔迹',
    noContentYet: '还没有写入内容',
    duplicate: '复制',
    delete: '删除',
    noDrafts: '还没有草稿',
    noDraftsHint: '先创建一张空白草稿纸，或者导入已有草稿文件。',
    featuresTitle: '当前版本能做什么',
    featuresHint: '这一版聚焦单人本地使用，不引入后端和账号。',
    featureAutosave: '自动保存当前草稿，刷新后继续写。',
    featureHistory: '支持撤销、重做、截图、导入导出。',
    featureViewport: '支持平移、缩放、旋转和形状优化。',
    featureMultiDraft: '工作台可管理多份本地草稿，并保留最近访问记录。',
    usageTitle: '建议使用方式',
    usageHint: '把它当作个人草稿纸、演算板或讲解板使用。需要长期保存时，记得定期导出文件。',
    backToWorkbench: '返回工作台',
    boardName: '草稿名',
    performanceMode: '性能模式',
    localBadge: '本地版',
    help: '帮助',
    helpTitle: '快捷键与使用说明',
    helpHint: '这一版主要服务于单人本地书写场景。',
    close: '关闭',
    helpItemSpace: '空格：临时橡皮擦，滚轮可调节橡皮大小。',
    helpItemCtrlWheel: 'Ctrl + 滚轮：调整画笔大小或缩放视图。',
    helpItemRotate: '右键 + 滚轮：围绕指针进行慢速旋转。',
    helpItemUndo: 'Ctrl + Z / Ctrl + Shift + Z：撤销与重做。',
    helpItemSave: '退出工作台前无需手动保存，白板会自动写入本地。'
  },
  'en-US': {
    untitledDraft: 'Untitled Draft',
    justNow: 'Just now',
    editorIdle: 'Not editing yet',
    localMode: 'Local mode',
    lastSavedAt: 'Last saved {time}',
    draftCreated: 'New draft created and ready to write',
    restoredDraft: 'Local draft restored, {time}',
    blankDraftOpened: 'Blank draft opened',
    draftPrefix: 'Draft',
    importDraft: 'Imported Draft',
    importingFile: 'Importing {name}...',
    workbenchReady: 'Workbench ready',
    deleteConfirm: 'Delete "{title}"? Its local content will be removed as well.',
    draftDeleted: 'Draft deleted',
    draftCopySuffix: 'Copy',
    autosavedAt: 'Autosaved at {time}',
    emptyHint: 'Create a new draft or import an existing draft file',
    appTitleSuffix: 'Draft Paper',
    eyebrow: 'Single-user online draft paper',
    heroTitle: 'Start writing instantly, come back anytime.',
    heroBody: 'This is a pure front-end, locally saved draft-paper workbench. It keeps the whiteboard core and adds recent drafts, resume flow, import/export, and usage guidance so it behaves more like a real personal tool.',
    newDraft: 'New Draft',
    importDraftButton: 'Import Draft',
    shortcuts: 'Shortcuts',
    localeLabel: 'Language',
    localeChinese: '中文',
    localeEnglish: 'English',
    draftCountUnit: 'local drafts',
    contentCountUnit: 'with content',
    localStorageNote: 'All data stays in this browser. Export files for offline backup, then import them again when needed.',
    recentDrafts: 'Recent Drafts',
    recentDraftsHint: 'Resume from recent work without digging for the entry point again.',
    import: 'Import',
    performanceHigh: 'High Performance',
    performanceBalanced: 'Balanced',
    strokeCount: '{count} strokes',
    noContentYet: 'No content yet',
    duplicate: 'Duplicate',
    delete: 'Delete',
    noDrafts: 'No drafts yet',
    noDraftsHint: 'Create a blank draft first, or import an existing draft file.',
    featuresTitle: 'What this version can do',
    featuresHint: 'This version focuses on local single-user usage, without backend services or accounts.',
    featureAutosave: 'Autosave the current draft and restore it after a refresh.',
    featureHistory: 'Supports undo, redo, screenshot, import, and export.',
    featureViewport: 'Supports pan, zoom, rotate, and shape beautification.',
    featureMultiDraft: 'Manage multiple local drafts and keep recent activity on the workbench.',
    usageTitle: 'Suggested usage',
    usageHint: 'Use it as a personal draft paper, scratchpad, or explanation board. Export files regularly if you want long-term backups.',
    backToWorkbench: 'Back to Workbench',
    boardName: 'Draft Name',
    performanceMode: 'Performance Mode',
    localBadge: 'Local',
    help: 'Help',
    helpTitle: 'Shortcuts and usage',
    helpHint: 'This version is mainly designed for local single-user writing.',
    close: 'Close',
    helpItemSpace: 'Space: temporary eraser; use the wheel to adjust eraser size.',
    helpItemCtrlWheel: 'Ctrl + wheel: adjust brush size or zoom the view.',
    helpItemRotate: 'Right click + wheel: rotate slowly around the pointer.',
    helpItemUndo: 'Ctrl + Z / Ctrl + Shift + Z: undo and redo.',
    helpItemSave: 'You do not need to save manually before leaving the workbench; the board saves itself locally.'
  }
}

function interpolate(template, params = {}) {
  return String(template).replace(/\{(\w+)\}/g, (_, key) => params[key] ?? '')
}

function createDraftRecord(overrides = {}) {
  const now = Date.now()
  const id = overrides.id || `draft-${Math.random().toString(36).slice(2, 10)}`
  const fallbackLocale = overrides.locale && SUPPORTED_LOCALES.includes(overrides.locale) ? overrides.locale : 'zh-CN'

  return {
    id,
    title: overrides.title || appCopy[fallbackLocale].untitledDraft,
    performanceMode: overrides.performanceMode || 'balanced',
    autosaveKey: overrides.autosaveKey || `draft-paper-board-${id}`,
    createdAt: overrides.createdAt || now,
    updatedAt: overrides.updatedAt || now,
    lastOpenedAt: overrides.lastOpenedAt || now,
    summary: {
      strokeCount: overrides.summary?.strokeCount || 0,
      bgColor: overrides.summary?.bgColor || '#ffffff',
      texture: overrides.summary?.texture || 'none',
      hasContent: overrides.summary?.hasContent || false
    }
  }
}

function resolveInitialLocale() {
  try {
    const stored = localStorage.getItem(APP_LOCALE_KEY)
    if (stored && SUPPORTED_LOCALES.includes(stored)) return stored
  } catch (_) {}

  if (typeof navigator !== 'undefined') {
    const navLocale = String(navigator.language || '').toLowerCase()
    if (navLocale.startsWith('en')) return 'en-US'
  }

  return 'zh-CN'
}

const locale = ref(resolveInitialLocale())

function t(key, params = {}) {
  const bundle = appCopy[locale.value] || appCopy['zh-CN']
  return interpolate(bundle[key] ?? key, params)
}

function formatTime(ts) {
  if (!ts) return t('justNow')
  const date = new Date(ts)
  return new Intl.DateTimeFormat(locale.value === 'en-US' ? 'en-US' : 'zh-CN', {
    month: 'numeric',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  }).format(date)
}

const drafts = ref([])
const activeDraftId = ref('')
const isEditorMode = ref(false)
const helpOpen = ref(false)
const saveStatus = ref(t('editorIdle'))
const whiteboardRef = ref(null)
const workbenchImportInput = ref(null)

const activeDraft = computed(() => drafts.value.find((draft) => draft.id === activeDraftId.value) || null)
const sortedDrafts = computed(() => {
  return [...drafts.value].sort((left, right) => (right.lastOpenedAt || right.updatedAt || 0) - (left.lastOpenedAt || left.updatedAt || 0))
})
const hasDrafts = computed(() => drafts.value.length > 0)
const lastSavedText = computed(() => {
  if (!activeDraft.value) return t('localMode')
  return t('lastSavedAt', { time: formatTime(activeDraft.value.updatedAt) })
})
const editorKey = computed(() => activeDraft.value ? `${activeDraft.value.id}:${activeDraft.value.autosaveKey}` : 'empty')

function persistAppState() {
  const payload = {
    schemaVersion: APP_SCHEMA_VERSION,
    activeDraftId: activeDraftId.value,
    drafts: drafts.value
  }
  localStorage.setItem(APP_STATE_KEY, JSON.stringify(payload))
}

function syncDraftSummaryFromStorage(draft) {
  if (!draft) return

  try {
    const raw = localStorage.getItem(draft.autosaveKey)
    if (!raw) {
      draft.summary = {
        ...draft.summary,
        strokeCount: 0,
        hasContent: false
      }
      return
    }

    const payload = JSON.parse(raw)
    const strokes = Array.isArray(payload?.strokes) ? payload.strokes : []
    draft.summary = {
      strokeCount: strokes.length,
      bgColor: payload?.meta?.bgColor || draft.summary?.bgColor || '#ffffff',
      texture: payload?.meta?.texture || draft.summary?.texture || 'none',
      hasContent: strokes.length > 0
    }
  } catch (_) {
    draft.summary = {
      ...draft.summary,
      strokeCount: draft.summary?.strokeCount || 0,
      hasContent: !!draft.summary?.hasContent
    }
  }
}

function loadAppState() {
  try {
    const raw = localStorage.getItem(APP_STATE_KEY)
    if (raw) {
      const payload = JSON.parse(raw)
      drafts.value = Array.isArray(payload?.drafts)
        ? payload.drafts.map((draft) => createDraftRecord(draft))
        : []
      activeDraftId.value = payload?.activeDraftId || drafts.value[0]?.id || ''
    }
  } catch (_) {
    drafts.value = []
    activeDraftId.value = ''
  }

  drafts.value.forEach(syncDraftSummaryFromStorage)

  if (activeDraftId.value && drafts.value.some((draft) => draft.id === activeDraftId.value)) {
    isEditorMode.value = true
  }
}

function createDraft(options = {}) {
  const draft = createDraftRecord(options)
  drafts.value.unshift(draft)
  activeDraftId.value = draft.id
  saveStatus.value = t('draftCreated')
  persistAppState()
  return draft
}

function openDraft(draftId) {
  const draft = drafts.value.find((item) => item.id === draftId)
  if (!draft) return

  draft.lastOpenedAt = Date.now()
  activeDraftId.value = draft.id
  isEditorMode.value = true
  saveStatus.value = draft.summary?.hasContent ? t('restoredDraft', { time: lastSavedText.value }) : t('blankDraftOpened')
  persistAppState()
}

function startNewDraft() {
  const draft = createDraft({ title: `${t('draftPrefix')} ${drafts.value.length + 1}`, locale: locale.value })
  isEditorMode.value = true
  openDraft(draft.id)
}

function triggerWorkbenchImport() {
  workbenchImportInput.value?.click()
}

async function openImportedDraft(file) {
  if (!file) return

  const baseName = String(file.name || t('importDraft')).replace(/\.[^.]+$/, '') || t('importDraft')
  const draft = createDraft({ title: baseName })
  isEditorMode.value = true
  openDraft(draft.id)
  saveStatus.value = t('importingFile', { name: file.name })

  await nextTick()
  await whiteboardRef.value?.importFile?.(file)
}

async function onWorkbenchImportChange(event) {
  const file = event.target.files && event.target.files[0]
  event.target.value = null
  if (!file) return

  await openImportedDraft(file)
}

function returnToWorkbench() {
  isEditorMode.value = false
  saveStatus.value = t('workbenchReady')
  persistAppState()
}

function deleteDraft(draftId) {
  const draft = drafts.value.find((item) => item.id === draftId)
  if (!draft) return
  const confirmed = window.confirm(t('deleteConfirm', { title: draft.title }))
  if (!confirmed) return

  localStorage.removeItem(draft.autosaveKey)
  drafts.value = drafts.value.filter((item) => item.id !== draftId)

  if (activeDraftId.value === draftId) {
    activeDraftId.value = drafts.value[0]?.id || ''
    isEditorMode.value = false
  }

  saveStatus.value = t('draftDeleted')
  persistAppState()
}

function duplicateDraft(draftId) {
  const source = drafts.value.find((item) => item.id === draftId)
  if (!source) return

  const draft = createDraft({
    title: `${source.title} ${t('draftCopySuffix')}`,
    performanceMode: source.performanceMode,
    summary: source.summary
  })

  const raw = localStorage.getItem(source.autosaveKey)
  if (raw) {
    localStorage.setItem(draft.autosaveKey, raw)
    syncDraftSummaryFromStorage(draft)
  }

  openDraft(draft.id)
}

function handleAutosaveUpdated(event) {
  const draft = activeDraft.value
  if (!draft || event?.autosaveKey !== draft.autosaveKey) return

  draft.updatedAt = event.updatedAt || Date.now()
  draft.lastOpenedAt = draft.updatedAt
  syncDraftSummaryFromStorage(draft)
  saveStatus.value = t('autosavedAt', { time: formatTime(draft.updatedAt) })
  persistAppState()
}

function setLocale(nextLocale) {
  if (!SUPPORTED_LOCALES.includes(nextLocale)) return
  locale.value = nextLocale
  try { localStorage.setItem(APP_LOCALE_KEY, nextLocale) } catch (_) {}
  if (!isEditorMode.value) {
    saveStatus.value = hasDrafts.value ? t('workbenchReady') : t('emptyHint')
  }
}

watch(
  () => activeDraft.value ? [activeDraft.value.title, activeDraft.value.performanceMode] : null,
  () => {
    if (!activeDraft.value) return
    activeDraft.value.updatedAt = Date.now()
    activeDraft.value.lastOpenedAt = activeDraft.value.updatedAt
    persistAppState()
  },
  { deep: true }
)

watch(
  () => activeDraft.value?.title,
  (title) => {
    document.title = title ? `${title} | ${t('appTitleSuffix')}` : t('appTitleSuffix')
  },
  { immediate: true }
)

watch(locale, () => {
  if (activeDraft.value?.title) {
    document.title = `${activeDraft.value.title} | ${t('appTitleSuffix')}`
  } else {
    document.title = t('appTitleSuffix')
  }
})

onMounted(() => {
  loadAppState()
  if (!hasDrafts.value) {
    saveStatus.value = t('emptyHint')
  }
})
</script>

<template>
  <div class="app-shell">
    <template v-if="!isEditorMode">
      <header class="hero-shell">
        <div class="hero-copy">
          <span class="eyebrow">{{ t('eyebrow') }}</span>
          <h1>{{ t('heroTitle') }}</h1>
          <p>{{ t('heroBody') }}</p>

          <div class="hero-actions">
            <button class="primary-btn" @click="startNewDraft">{{ t('newDraft') }}</button>
            <button class="ghost-btn" @click="triggerWorkbenchImport">{{ t('importDraftButton') }}</button>
            <button class="ghost-btn" @click="helpOpen = true">{{ t('shortcuts') }}</button>
            <label class="mode-select locale-select">
              <span>{{ t('localeLabel') }}</span>
              <select :value="locale" @change="setLocale($event.target.value)">
                <option value="zh-CN">{{ t('localeChinese') }}</option>
                <option value="en-US">{{ t('localeEnglish') }}</option>
              </select>
            </label>
          </div>
          <input ref="workbenchImportInput" class="hidden-file" type="file" accept=".zxkjbb,.json" @change="onWorkbenchImportChange" />
        </div>

        <div class="hero-panel">
          <div class="hero-stat">
            <strong>{{ drafts.length }}</strong>
            <span>{{ t('draftCountUnit') }}</span>
          </div>
          <div class="hero-stat">
            <strong>{{ drafts.filter((draft) => draft.summary?.hasContent).length }}</strong>
            <span>{{ t('contentCountUnit') }}</span>
          </div>
          <div class="hero-note">{{ t('localStorageNote') }}</div>
        </div>
      </header>

      <main class="workbench">
        <section class="panel recent-panel">
          <div class="section-head">
            <div>
              <h2>{{ t('recentDrafts') }}</h2>
              <p>{{ t('recentDraftsHint') }}</p>
            </div>
            <div class="section-actions">
              <button class="ghost-btn" @click="triggerWorkbenchImport">{{ t('import') }}</button>
              <button class="ghost-btn" @click="startNewDraft">{{ t('newDraft') }}</button>
            </div>
          </div>

          <div v-if="sortedDrafts.length" class="draft-grid">
            <article
              v-for="draft in sortedDrafts"
              :key="draft.id"
              class="draft-card"
              :style="{ '--card-tint': draft.summary?.bgColor || '#ffffff' }"
            >
              <button class="draft-open" @click="openDraft(draft.id)">
                <span class="draft-chip">{{ draft.performanceMode === 'high' ? t('performanceHigh') : t('performanceBalanced') }}</span>
                <h3>{{ draft.title }}</h3>
                <p>
                  {{ draft.summary?.hasContent ? t('strokeCount', { count: draft.summary?.strokeCount || 0 }) : t('noContentYet') }}
                </p>
                <span class="draft-time">{{ formatTime(draft.lastOpenedAt || draft.updatedAt) }}</span>
              </button>

              <div class="draft-actions">
                <button class="link-btn" @click="duplicateDraft(draft.id)">{{ t('duplicate') }}</button>
                <button class="link-btn danger" @click="deleteDraft(draft.id)">{{ t('delete') }}</button>
              </div>
            </article>
          </div>

          <div v-else class="empty-state">
            <h3>{{ t('noDrafts') }}</h3>
            <p>{{ t('noDraftsHint') }}</p>
          </div>
        </section>

        <section class="panel info-panel">
          <div class="section-head">
            <div>
              <h2>{{ t('featuresTitle') }}</h2>
              <p>{{ t('featuresHint') }}</p>
            </div>
          </div>

          <ul class="capability-list">
            <li>{{ t('featureAutosave') }}</li>
            <li>{{ t('featureHistory') }}</li>
            <li>{{ t('featureViewport') }}</li>
            <li>{{ t('featureMultiDraft') }}</li>
          </ul>

          <div class="tips-box">
            <h3>{{ t('usageTitle') }}</h3>
            <p>{{ t('usageHint') }}</p>
          </div>
        </section>
      </main>
    </template>

    <template v-else-if="activeDraft">
      <header class="editor-topbar">
        <div class="editor-main">
          <button class="ghost-btn" @click="returnToWorkbench">{{ t('backToWorkbench') }}</button>

          <div class="editor-title-wrap">
            <label for="board-name">{{ t('boardName') }}</label>
            <input id="board-name" v-model="activeDraft.title" class="board-name-input" type="text" maxlength="40" />
          </div>

          <label class="mode-select">
            <span>{{ t('performanceMode') }}</span>
            <select v-model="activeDraft.performanceMode">
              <option value="balanced">{{ t('performanceBalanced') }}</option>
              <option value="high">{{ t('performanceHigh') }}</option>
            </select>
          </label>

          <label class="mode-select locale-select compact-locale-select">
            <span>{{ t('localeLabel') }}</span>
            <select :value="locale" @change="setLocale($event.target.value)">
              <option value="zh-CN">{{ t('localeChinese') }}</option>
              <option value="en-US">{{ t('localeEnglish') }}</option>
            </select>
          </label>
        </div>

        <div class="editor-status">
          <span class="status-badge">{{ t('localBadge') }}</span>
          <span>{{ lastSavedText }}</span>
          <button class="ghost-btn" @click="helpOpen = true">{{ t('help') }}</button>
        </div>
      </header>

      <WhiteboardView
        ref="whiteboardRef"
        :key="editorKey"
        :board-title="activeDraft.title"
        :autosave-key="activeDraft.autosaveKey"
        :performance-mode="activeDraft.performanceMode"
        :locale="locale"
        @request-exit="returnToWorkbench"
        @autosave-updated="handleAutosaveUpdated"
      />

      <div class="floating-status">{{ saveStatus }}</div>
    </template>

    <div v-if="helpOpen" class="modal-backdrop" @click.self="helpOpen = false">
      <section class="modal-card">
        <div class="section-head compact">
          <div>
            <h2>{{ t('helpTitle') }}</h2>
            <p>{{ t('helpHint') }}</p>
          </div>
          <button class="ghost-btn" @click="helpOpen = false">{{ t('close') }}</button>
        </div>

        <ul class="capability-list modal-list">
          <li>{{ t('helpItemSpace') }}</li>
          <li>{{ t('helpItemCtrlWheel') }}</li>
          <li>{{ t('helpItemRotate') }}</li>
          <li>{{ t('helpItemUndo') }}</li>
          <li>{{ t('helpItemSave') }}</li>
        </ul>
      </section>
    </div>
  </div>
</template>

<style scoped>
:global(body) {
  margin: 0;
  font-family: 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  background:
    radial-gradient(circle at top left, rgba(255, 212, 163, 0.6), transparent 30%),
    radial-gradient(circle at right center, rgba(180, 224, 214, 0.42), transparent 26%),
    #f4efe6;
  color: #172033;
}

:global(*) {
  box-sizing: border-box;
}

.app-shell {
  min-height: 100vh;
}

.hero-shell {
  display: grid;
  grid-template-columns: minmax(0, 1.3fr) minmax(280px, 0.7fr);
  gap: 24px;
  padding: 28px;
}

.hero-copy,
.hero-panel,
.panel,
.modal-card,
.editor-topbar,
.floating-status {
  border: 1px solid rgba(23, 32, 51, 0.08);
  box-shadow: 0 20px 60px rgba(93, 72, 40, 0.08);
}

.hero-copy,
.hero-panel,
.panel,
.modal-card {
  background: rgba(255, 251, 245, 0.84);
  backdrop-filter: blur(14px);
  border-radius: 24px;
}

.hero-copy {
  padding: 32px;
}

.eyebrow {
  display: inline-flex;
  align-items: center;
  min-height: 32px;
  padding: 0 12px;
  border-radius: 999px;
  background: rgba(21, 97, 109, 0.1);
  color: #15616d;
  font-size: 13px;
  font-weight: 700;
}

.hero-copy h1 {
  margin: 18px 0 12px;
  font-size: clamp(34px, 5vw, 64px);
  line-height: 1.02;
  letter-spacing: -0.04em;
}

.hero-copy p,
.section-head p,
.tips-box p,
.hero-note,
.draft-card p,
.empty-state p,
.modal-list,
.editor-status {
  color: #526079;
}

.hero-copy p {
  max-width: 720px;
  font-size: 16px;
  line-height: 1.7;
}

.hero-actions,
.draft-actions,
.editor-main,
.editor-status,
.section-head,
.section-actions,
.hero-panel {
  display: flex;
  align-items: center;
}

.hero-actions,
.draft-actions,
.editor-main,
.editor-status,
.section-head,
.section-actions {
  gap: 12px;
}

.section-actions {
  flex-wrap: wrap;
  justify-content: flex-end;
}

.hero-actions {
  margin-top: 22px;
  flex-wrap: wrap;
}

.primary-btn,
.ghost-btn,
.soft-btn,
.board-name-input,
.mode-select select,
.link-btn {
  border-radius: 14px;
  border: 1px solid rgba(23, 32, 51, 0.12);
  font: inherit;
}

.primary-btn,
.ghost-btn,
.soft-btn,
.link-btn {
  min-height: 44px;
  padding: 0 16px;
  cursor: pointer;
  transition: transform 0.18s ease, background 0.18s ease, border-color 0.18s ease;
}

.primary-btn {
  background: #172033;
  color: #fff7ec;
}

.ghost-btn {
  background: rgba(255, 255, 255, 0.72);
  color: #172033;
}

.soft-btn {
  background: rgba(245, 142, 54, 0.12);
  color: #8f4b08;
}

.primary-btn:hover,
.ghost-btn:hover,
.soft-btn:hover,
.link-btn:hover,
.draft-open:hover {
  transform: translateY(-1px);
}

.hero-panel {
  flex-direction: column;
  justify-content: center;
  gap: 18px;
  padding: 28px;
}

.hero-stat {
  width: 100%;
  padding: 18px 20px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.56);
}

.hero-stat strong {
  display: block;
  margin-bottom: 4px;
  font-size: 30px;
}

.hero-note {
  width: 100%;
  line-height: 1.7;
}

.workbench {
  display: grid;
  grid-template-columns: minmax(0, 1.3fr) minmax(300px, 0.7fr);
  gap: 24px;
  padding: 0 28px 28px;
}

.panel {
  padding: 24px;
}

.section-head {
  justify-content: space-between;
  margin-bottom: 18px;
}

.section-head h2,
.tips-box h3,
.empty-state h3,
.draft-card h3 {
  margin: 0;
}

.section-head p {
  margin: 6px 0 0;
}

.draft-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 16px;
}

.draft-card {
  overflow: hidden;
  border-radius: 20px;
  border: 1px solid rgba(23, 32, 51, 0.08);
  background: linear-gradient(180deg, color-mix(in srgb, var(--card-tint) 72%, white 28%), rgba(255, 255, 255, 0.94));
}

.draft-open {
  display: block;
  width: 100%;
  padding: 18px;
  border: 0;
  background: transparent;
  text-align: left;
  cursor: pointer;
}

.draft-chip,
.status-badge {
  display: inline-flex;
  align-items: center;
  min-height: 28px;
  padding: 0 10px;
  border-radius: 999px;
  background: rgba(21, 97, 109, 0.1);
  color: #15616d;
  font-size: 12px;
  font-weight: 700;
}

.draft-card h3 {
  margin: 14px 0 8px;
  font-size: 22px;
}

.draft-time {
  display: block;
  margin-top: 16px;
  font-size: 13px;
  color: #6b7588;
}

.draft-actions {
  justify-content: flex-end;
  padding: 0 18px 18px;
}

.link-btn {
  background: rgba(255, 255, 255, 0.76);
  color: #172033;
}

.danger {
  color: #b42318;
}

.capability-list {
  margin: 0;
  padding-left: 20px;
  line-height: 1.8;
}

.tips-box {
  margin-top: 22px;
  padding: 18px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.62);
}

.empty-state {
  padding: 42px 18px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.52);
  text-align: center;
}

.editor-topbar {
  position: sticky;
  top: 0;
  z-index: 20;
  display: flex;
  justify-content: space-between;
  gap: 18px;
  padding: 14px 18px;
  background: rgba(255, 250, 242, 0.88);
  backdrop-filter: blur(14px);
}

.editor-main,
.editor-status {
  flex-wrap: wrap;
}

.editor-main {
  flex: 1;
}

.editor-title-wrap,
.mode-select {
  display: flex;
  align-items: center;
  gap: 8px;
}

.editor-title-wrap label,
.mode-select span {
  font-size: 13px;
  color: #526079;
}

.board-name-input,
.mode-select select {
  min-height: 44px;
  padding: 0 14px;
  background: rgba(255, 255, 255, 0.92);
}

.board-name-input {
  min-width: min(42vw, 340px);
}

.floating-status {
  position: fixed;
  right: 18px;
  bottom: 18px;
  z-index: 18;
  padding: 12px 16px;
  border-radius: 16px;
  background: rgba(255, 250, 242, 0.92);
  color: #364055;
  backdrop-filter: blur(12px);
}

.modal-backdrop {
  position: fixed;
  inset: 0;
  z-index: 40;
  display: grid;
  place-items: center;
  padding: 20px;
  background: rgba(17, 24, 39, 0.3);
}

.modal-card {
  width: min(720px, 100%);
  padding: 24px;
}

.compact {
  margin-bottom: 12px;
}

.modal-list {
  margin-top: 0;
}

.hidden-file {
  display: none;
}

@media (max-width: 980px) {
  .hero-shell,
  .workbench {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 720px) {
  .hero-shell,
  .workbench {
    padding-left: 16px;
    padding-right: 16px;
  }

  .hero-copy,
  .hero-panel,
  .panel,
  .modal-card {
    border-radius: 18px;
  }

  .editor-topbar {
    padding: 12px;
  }

  .board-name-input {
    min-width: 0;
    width: 100%;
  }

  .editor-title-wrap,
  .mode-select,
  .editor-main,
  .editor-status {
    width: 100%;
  }

  .floating-status {
    left: 12px;
    right: 12px;
    bottom: 12px;
  }
}
</style>