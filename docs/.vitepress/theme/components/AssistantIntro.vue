<script setup>
import { onMounted, onBeforeUnmount, ref, computed, nextTick } from 'vue'
import { portfolioContent as DATA } from '../content/portfolioContent'
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
const showMore = ref(false)   // toggelt die extra Inhalte in der Detail-View



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

  showMore.value = false
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

function tellMeMore(item) {
  // 1) Assistent wird wieder "active"
  assistantPose.value = 'active'
  stage.value = 'done' // oder ein eigener stage, je nachdem wie du’s nutzt
  message.value =
    item?.more?.assistantLine ||
    "Sure. Here's the part people usually skim — try not to."

  // 2) More-Content aufklappen
  showMore.value = true

  // 3) Optional: wenn du willst, dass die Buttons im Overlay verschwinden:
  showButtons.value = false
}

function toggleMore(item){
  if (!showMore.value) {
    tellMeMore(item)
  } else {
    showMore.value = false
    assistantPose.value = 'idle'  // optional wieder wegdrehen
    message.value = ''
    stage.value = 'choose'        // oder was bei dir passt
  }
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
  <section class="hero" :class="{ panelOpen: panelOpen }" aria-label="assistant hero">
    <!-- Background drawing -->
    <div class="hero-media" @click="engageOnce">
      <img class="hero-img"  :src="heroImg" alt="" draggable="false" />
    </div>

  <!-- Overlay RIGHT (nur wenn Panel zu) -->
<div class="overlay overlay-right" v-if="!panelOpen">
  <div v-if="stage === 'idle'" class="hint">…</div>

  <div v-else class="sheet">
    <div class="sheet-text">{{ message }}</div>

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

<!-- Overlay TORSO (nur wenn Panel offen) -->
<div class="overlay overlay-torso" v-else>
  <!-- optional: im Panel-open idle gar nichts zeigen -->
  <div v-if="stage !== 'idle'" class="sheet">
    <div class="sheet-text">{{ message }}</div>
  </div>
</div>

    <!-- RIGHT PANEL -->
<aside class="panel" :class="{open: panelOpen, detail: !!panelDetail }">
  <div class="panel-inner">

    <!-- unified nav: back (left) when in detail, close (right) always -->
    <div class="detail-nav">
      <div class="detail-nav-left">
        <button
          v-if="panelDetail"
          class="detail-back"
          type="button"
          @click="backToList"
          aria-label="Back to list"
        >
          ←
        </button>
      </div>
      <button class="panel-close" type="button" @click="closePanel" aria-label="Close panel">×</button>
    </div>
 
   <!-- LIST VIEW -->
   <template v-if="panelOpen && !panelDetail">
     <div class="panel-title">{{ panelView }}//</div>
 
     <div class="panel-list">
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
   </template>
 
   <!-- DETAIL VIEW -->
   <template v-else-if="panelOpen && panelDetail">
  <div class="detail">
 
   <div class="detail-kicker">{{ panelDetail.meta }}</div>
   <div class="detail-title">{{ panelDetail.title }}</div>

  <p class="detail-summary">
    {{ panelDetail.summary }}
  </p>

  <div v-if="panelDetail.image" class="detail-media">
    <img :src="panelDetail.image" alt="" />
  </div>

  <div class="detail-actions">
  <button class="btn-outline" type="button" @click="toggleMore(panelDetail)">
  {{ showMore ? 'Hide' : 'Tell me more' }}
  </button>

    <button class="btn-outline" type="button" @click="goSeeAll(panelView)">
      See all
    </button>
  </div>
</div>

<div v-if="showMore && panelDetail.more" class="detail-more">

  <div class="more-text">
    <p v-for="(p, i) in panelDetail.more.text" :key="i" class="more-p">
      {{ p }}
    </p>
  </div>

  <div v-if="panelDetail.more.gallery?.length" class="more-gallery">
    <img
      v-for="(src, i) in panelDetail.more.gallery"
      :key="i"
      :src="src"
      alt=""
      class="more-img"
    />
  </div>
</div>

  </template>

</div>

</aside>

  </section>
</template>

<style scoped>
/* =========================
   Variables / palette
   ========================= */
:root{
  --panel-bg: #151515;
  --panel-radius: 14px;
  --muted: rgba(255,255,255,.65);
  --muted-strong: rgba(255,255,255,.75);
  --muted-subtle: rgba(255,255,255,.06);
  --accent-hover: rgba(255,255,255,.06);
}

/* =========================
   HERO / background
   ========================= */
.hero {
  position: relative;
  width: 100%;
  height: 100svh;
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
  object-position: 10% center;
  display: block;
  transform: scale(clamp(1.02, 1vw + 1.02, 1.10));
  transition: transform 280ms ease, object-position 280ms ease;
}

.hero.panelOpen .hero-img{
  /* move more to the left when panel is open */
  object-position: -30% center;
  transform: scale(clamp(1.00, 1vw + 1.00, 1.06));
}


/* =========================
   OVERLAY / assistant sheet
   ========================= */
/* shared overlay base */
.overlay{
  position: absolute;
  inset: 0;
  pointer-events: none;
}

/* shared sheet */
.sheet{
  pointer-events: auto;
  position: absolute;
  width: min(420px, 40vw);
  padding: 18px 18px 16px;
  border-radius: 14px;

  background: rgba(255,255,255,0.42);
  border: 1px solid rgba(0,0,0,0.06);
  box-shadow: 0 18px 40px rgba(0,0,0,0.10);

  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
}

/* text */
.sheet-text{
  font-size: clamp(13px, 1.1vw, 16px);
  line-height: 1.75;
  color: rgba(10,10,10,0.78);
  margin-bottom: 14px;
}

/* RIGHT position (default, panel closed) */
.overlay-right .sheet{
  right: clamp(80px, 10vw, 160px);
  top: clamp(18%, 22vh, 28%);
}

/* TORSO position (panel open) */
.overlay-torso .sheet{
  left: clamp(18%, 12vw, 30%);
  top: clamp(44%, 50vh, 58%);
  transform: translateY(-40%);
  transform: translateX(-45%)
}


/* hint */
.hint{
  position: absolute;
  right: clamp(80px, 10vw, 160px);
  top: clamp(18%, 22vh, 28%);
  font-style: italic;
  font-size: 12px;
  color: #6b7280;
  opacity: 0.9;
  pointer-events: none;
}



/* =========================
   CHOICES / buttons
   ========================= */
.choices{
  display: grid;
  gap: 10px;
}

.choice{
  width: 100%;
  background: rgba(20,20,20,0.90);
  color: #fff;
  border: none;
  border-radius: 10px;
  padding: 12px 14px;
  font-size: 12px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  cursor: pointer;
}

.choice:hover{ transform: translateY(-1px); }

.choice.subtle{
  background: rgba(255,255,255,0.0);
  color: rgba(10,10,10,0.70);
  border: 1px solid rgba(0,0,0,0.12);
  text-transform: none;
  letter-spacing: 0;
  font-style: italic;
}


/* =========================
   PANEL container
   ========================= */
.panel {
  position: absolute;
  top: 0;
  right: 0;
  height: 100%;
  width: min(440px, 36vw);
  background:#151515;;
  color: #fff;
  transform: translateX(102%);
  transition: transform 280ms ease;
  border-top-left-radius: var(--panel-radius);
  border-bottom-left-radius: var(--panel-radius);
  overflow: hidden;
  pointer-events: none;
}

/* open = sichtbar, aber width NICHT pauschal setzen */
.panel.open{
  transform: translateX(0);
  pointer-events: auto;
}

/* LIST VIEW WIDTH */
.panel.open:not(.detail){
  width: min(520px, 42vw);
}

/* DETAIL VIEW WIDTH (dein 30/70 Ziel) */
.panel.open.detail{
  width: min(980px, 70vw);
} 

/* inner wrapper that scrolls */
.panel-inner {
  height: 100%;
  max-width: 680px;
  margin: 0 auto;
  padding: 22px;
  display: flex;
  flex-direction: column;
  gap: 18px;
  overflow-y: auto;
  overflow-x: hidden;
  -webkit-overflow-scrolling: touch;
}
.panel-inner::-webkit-scrollbar{ width: 5px; } 
.panel-inner::-webkit-scrollbar-thumb{
  background: rgba(255,255,255,.12);
  border-radius: 10px;
}
.panel-inner::-webkit-scrollbar-track{ background: transparent; }


/* header / title */
.panel-top { display: flex; align-items: center; justify-content: space-between; }
.panel-title {
  font-size: 12px;
  letter-spacing: 0.8px;
  text-transform: lowercase;
  opacity: 0.9;
}

/* =========================
   NAV (unified top row)
   - left: optional back when in detail
   - right: always the close button
   ========================= */
.detail-nav{
  position: absolute;
  top: 16px;
  left: 16px;
  right: 16px;
  z-index: 5;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0;                 /* important: no inner padding */
  pointer-events: none;       /* won't steal layout/click focus */
}

/* make buttons clickable inside nav */
.detail-nav button { pointer-events: auto; }

.detail-nav-left { display: flex; align-items: center; gap: 8px; }

/* shared button styles for nav controls */
.detail-back,
.panel-close{
  width: 38px;
  height: 38px;
  display: grid;
  place-items: center;
  background: transparent;
  border: none;
  color: var(--muted-strong);
  font-size: 22px;
  cursor: pointer;
  border-radius: 10px;
}

.detail-back:hover,
.panel-close:hover{
  color: rgba(255,255,255,.95);
  background: var(--accent-hover);
}

/* =========================
   LIST view
   ========================= */
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

.panel-item-title { font-size: 13px; letter-spacing: 0.2px; }
.panel-item-meta { margin-top: 6px; font-size: 11px; opacity: 0.65; }

/* =========================
   DETAIL view
   ========================= */
.panel-detail .detail-title { font-size: 16px; font-weight: 600; }
.panel-detail .detail-meta { margin-top: 6px; font-size: 12px; opacity: 0.7; }

.detail {
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 18px;
  padding-top: 30px;   /* space under nav buttons */
}

.detail-top { margin: 0; }

.detail-kicker {
  font-size: 12px;
  letter-spacing: .08em;
  opacity: .7;
  margin-bottom: 10px;
}

.detail-title {
  font-size: clamp(26px, 2.2vw, 40px);
  font-weight: 600;
  letter-spacing: -0.01em;
  margin-bottom: 10px;
}

.detail-summary {
  max-width: 62ch;
  font-size: 14px;
  line-height: 1.8;
  opacity: .92;
  margin: 0 0 18px 0;
}

/* media block inside detail */
.detail-media {
  width: 100%;
  border-radius: 14px;
  overflow: hidden;
  background: rgba(255,255,255,.06);
  max-height: 360px;
}

.detail-media img {
  width: 100%;
  height: 360px;
  object-fit: cover;
  display: block;
}

/* actions at bottom of detail (kept inside flow) */
.detail-actions {
  display: flex;
  gap: 18px;
  justify-content: center;
  padding-top: 18px;
  margin-top: 10px;
}

/* small utility for navigation inside detail content */
.detail-nav { /* already defined above for placement */ }

/* =========================
   MORE / expanded content
   ========================= */
.detail-more {
  margin-top: 10px;
  display: grid;
  gap: 18px;
}

.more-text { max-width: 62ch; opacity: .92; }

.more-p {
  font-size: 14px;
  line-height: 1.85;
  margin: 0 0 12px 0;
}

.more-gallery {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.more-img {
  width: 100%;
  height: 220px;
  object-fit: cover;
  border-radius: 12px;
  display: block;
  background: rgba(255,255,255,.06);
}

/* =========================
   responsive tweaks
   ========================= */
@media (max-width: 720px){
  .more-gallery { grid-template-columns: 1fr; }
  .more-img { height: 260px; }
}

/* end of organized styles */
</style>

