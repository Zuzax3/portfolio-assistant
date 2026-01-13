<script setup>
import { onMounted, onBeforeUnmount, ref, computed, nextTick } from 'vue'

/**
 * stages:
 * idle -> line1 -> line2 -> choose -> done
 */
const stage = ref('idle')
const quietLine = ref('')         // zeigt einmal "Take your time."
const selected = ref(null)
const showButtons = ref(false)

const line1Options = [
  "You're in my portfolio. It's quieter here.",
  "Welcome. It won't explain itself unless you ask.",
  "This is a space for work. And the gaps around it."
]

const line2Options = [
  "I can show you projects, posters, or fragments.",
  "Some things are finished. Some things are just placed here.",
  "Pick a direction. Or don’t. I’ll survive."
]

const askOptions = [
  "What do you want to see?",
  "Where should we start?",
  "Choose. I’ll do the pointing."
]

const panelOpen = ref(false)
const panelView = ref('') // 'projects' | 'unfold' | 'sketches' | ''
const panelDetail = ref(null) // item object or null

const DATA = {
  projects: [
    { id: 'doubt-it', title: 'Doubt It', meta: 'interactive / AI / satire' },
    { id: 'listener', title: 'Listener', meta: 'speculative object' },
    { id: 'poppies', title: 'Data/Drama – Poppies', meta: 'installation' },
  ],
  unfold: [
    { id: 'unfold-01', title: 'UNFOLD// 01', meta: 'poster / typography' },
    { id: 'unfold-02', title: 'UNFOLD// 02', meta: 'poster / layout' },
  ],
  sketches: [
    { id: 'sketch-01', title: 'Sketch// 01', meta: 'ink / fragment' },
    { id: 'sketch-02', title: 'Sketch// 02', meta: 'study' },
  ]
}

const assistantPose = ref('idle') 
// 'idle' | 'active' | 'back'

const heroImg = computed(() => {
  if (assistantPose.value === 'back') return '/assistant_back.png'
  if (assistantPose.value === 'active') return '/assistant_active.png'
  return '/assistant_idle.png'
})

const pick = (arr) => arr[Math.floor(Math.random() * arr.length)]
const line1 = ref(pick(line1Options))
const line2 = ref(pick(line2Options))
const ask = ref(pick(askOptions))

const message = computed(() => {
     if (quietLine.value) return quietLine.value

  if (stage.value === 'idle') return ''
  if (stage.value === 'line1') return line1.value
  if (stage.value === 'line2') return line2.value
  if (stage.value === 'choose') return ask.value
  if (stage.value === 'done') {
    if (selected.value === 'projects') return "Alright. These took time. And compromise."
    if (selected.value === 'unfold') return "Not answers. Visual thoughts."
    if (selected.value === 'sketches') return "Fragments. Fast. Mostly honest."
    return "Fair enough. Scroll."
  }
  return ''
})

function advance() {
  if (panelOpen.value) return

  if (stage.value === 'idle') {
    assistantPose.value = 'active'  // ✅ nur hier
    stage.value = 'line1'
    return
  }

  if (stage.value === 'line1') {
    stage.value = 'line2'
    return
  }
  if (stage.value === 'line2') {
    stage.value = 'choose'
    showButtons.value = false
    // kleine Verzögerung, bis die Buttons auftauchen
    setTimeout(() => {
      showButtons.value = true
    }, 450)
    return
  }
  // wenn choose/done: klick irgendwo macht nichts (sonst nervig)
}

function goTo(which) {
  selected.value = which

  panelOpen.value = true
  panelView.value = which
  panelDetail.value = null

  // ✅ wichtig: beim Öffnen der Liste sofort idle
  assistantPose.value = 'idle'
  quietLine.value = ''

  // falls du stage für message nutzt: hier ruhig lassen
  stage.value = 'idle'
  showButtons.value = false
}


function openItem(item) {
  panelDetail.value = item
   // Assistent tritt zurück
  assistantPose.value = 'back'

  // Einmaliger Satz
  quietLine.value = 'Take your time.'
  setTimeout(() => {
    quietLine.value = ''
  }, 1400)
}

function backToList() {
  panelDetail.value = null

  assistantPose.value = 'idle'
  quietLine.value = ''
}

function closePanel() {
  panelOpen.value = false
  panelView.value = ''
  panelDetail.value = null
  // optional: wieder zum Auswahlmodus
  
     assistantPose.value = 'active'
  stage.value = 'choose'
  showButtons.value = false
  quietLine.value = ''

  setTimeout(() => (showButtons.value = true), 250)
}


function reset() {
  selected.value = null
  line1.value = pick(line1Options)
  line2.value = pick(line2Options)
  ask.value = pick(askOptions)
  stage.value = 'choose'
  showButtons.value = false
  setTimeout(() => (showButtons.value = true), 250)
}

function handleGlobalClick(e) {
  // Wenn Panel offen ist, Assistent nicht weiterlaufen lassen
  if (panelOpen.value) return

  // Wenn auf Button geklickt wurde, nicht weiter-advancen
  const target = e.target
  if (target && target.closest && target.closest('button')) return

  advance()
}


onMounted(() => {
  window.addEventListener('pointerdown', handleGlobalClick)
})

onBeforeUnmount(() => {
  window.removeEventListener('pointerdown', handleGlobalClick)
})
</script>


<template>
  <section class="hero" aria-label="assistant hero">
    <!-- Background drawing -->
    <div class="hero-media" @click="engageOnce">
      <img class="hero-img"  :src="heroImg" alt="" draggable="false" />
    </div>

    <!-- Overlay: message + buttons -->
    <div class="overlay" :class="{ show: stage !== 'idle' }">
        <div v-if="stage === 'idle'" class="hint">…</div>


      <div v-else class="bubble">
        <div class="text">{{ message }}</div>

     <div v-if="stage === 'choose' && showButtons" class="choices">
         <button class="choice" type="button" @click="goTo('projects')">projects//</button>
         <button class="choice" type="button" @click="goTo('unfold')">UNFOLD//</button>
         <button class="choice" type="button" @click="goTo('sketches')">sketches//</button>
    </div>


    <div v-else-if="stage === 'done'" class="choices">
         <button class="choice subtle" type="button" @click="reset">tell me again</button>
            </div>
        </div>
    </div>

    <!-- RIGHT PANEL -->
<aside class="panel" :class="{open: panelOpen, detail: !!panelDetail }">
  <div class="panel-inner">
    <div class="panel-top">
      <div class="panel-title">{{ panelView }}//</div>
      <button class="panel-close" type="button" @click="closePanel">×</button>
    </div>

    <!-- LIST VIEW -->
    <div v-if="panelOpen && !panelDetail" class="panel-list">
      <button
        v-for="item in DATA[panelView]"
        :key="item.id"
        class="panel-item"
        type="button"
        @click="openItem(item)"
      >
        <div class="panel-item-title">{{ item.title }}</div>
        <div class="panel-item-meta">{{ item.meta }}</div>
      </button>
    </div>

    <!-- DETAIL VIEW (placeholder for next step) -->
    <div v-else-if="panelOpen && panelDetail" class="panel-detail">
      <div class="detail-title">{{ panelDetail.title }}</div>
      <div class="detail-meta">{{ panelDetail.meta }}</div>

      <div class="detail-body">
        (next: project overview / images / tell me more)
      </div>

      <button class="panel-back" type="button" @click="backToList">← back</button>
    </div>
  </div>
</aside>

  </section>
</template>

<style scoped>

.hero {
  position: relative;
  width: 100%;
  height: 100svh; /* bessere mobile viewport units */
  min-height: 680px;
  max-height: 980px;
  overflow: hidden;
}


.hero-media {
  position: relative;
  width: 100%;
  height: 100%;
  cursor: pointer;
  user-select: none;
}

.hero-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  object-position: 18% center;
  display: block;
  transform: scale(clamp(1.02, 1vw + 1.02, 1.10));
}


.overlay {
  position: absolute;
  inset: 0;
  display: flex;
  justify-content: flex-end;
  align-items: flex-start;


  padding-right: clamp(25vw, 10vw, 14vw);
  padding-left: clamp(200px, 4vw, 64px);
  padding-top: clamp(6vw, 24vh, 220px);
}


.overlay.show .bubble {
  opacity: 1;
  transform: translateY(0);
}

.hint {
  pointer-events: none;
  font-style: italic;
  font-size: 12px;
  color: #6b7280;
  opacity: 0.9;
  margin-right: 2px;
}

/* Message “bubble” als ruhiger UI-Block */
.bubble {
  pointer-events: auto;
  width: min(260px, 42vw);
}

.text {
  font-size: clamp(13px, 1.1vw, 16px);
  line-height: 1.7;
  margin-bottom: clamp(28px, 6vh, 64px); 
}



/* Buttons wie in deinem Mock */
.choices {
  display: grid;
  gap: 14px;
  margin-top: 12px;
}


.choice {
  width: 100%;
  background: #111;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 14px 16px;
  font-size: 12px;
  letter-spacing: 0.8px;
  text-transform: uppercase;
  cursor: pointer;
}

.choice:hover {
  transform: translateY(-1px);
}

.choice.subtle {
  background: transparent;
  color: #111;
  border: 1px solid #e5e5e5;
  text-transform: none;
  letter-spacing: 0;
  font-style: italic;
}

.panel {
  position: absolute;
  top: 0;
  right: 0;
  height: 100%;
  width: min(440px, 36vw);
  background: #151515;
  color: #fff;
  transform: translateX(102%);
  transition: transform 280ms ease;
  border-top-left-radius: 14px;
  border-bottom-left-radius: 14px;
  overflow: hidden;
  pointer-events: none;
}

.panel.open {
  transform: translateX(0);
  pointer-events: auto;
}

.panel.detail {
  width: min(760px, 54vw);
}

.panel-inner {
  height: 100%;
  max-width: 680px;
  margin: 0 auto;
  padding: 22px;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.panel-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.panel-title {
  font-size: 12px;
  letter-spacing: 0.8px;
  text-transform: lowercase;
  opacity: 0.9;
}

.panel-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.panel-item {
  text-align: left;
  background: transparent;
  border: none;
  color: #fff;
  padding: 14px 12px;
  border-radius: 10px;
  cursor: pointer;
  transition: background 160ms ease, transform 160ms ease;
}

.panel-item:hover {
  background: rgba(255,255,255,0.06);
  transform: translateY(-1px);
}

.panel-item-title {
  font-size: 13px;
  letter-spacing: 0.2px;
}

.panel-item-meta {
  margin-top: 6px;
  font-size: 11px;
  opacity: 0.65;
}

.panel-detail .detail-title {
  font-size: 16px;
  font-weight: 600;
}

.panel-detail .detail-meta {
  margin-top: 6px;
  font-size: 12px;
  opacity: 0.7;
}

.detail-body {
  margin-top: 18px;
  font-size: 12px;
  opacity: 0.75;
  line-height: 1.6;
}

.panel-back {
  margin-top: auto;
  background: transparent;
  border: 1px solid rgba(255,255,255,0.2);
  color: #fff;
  padding: 10px 12px;
  border-radius: 10px;
  cursor: pointer;
  opacity: 0.9;
}

</style>

