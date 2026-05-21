<template>
  <div class="page">

    <!-- GLOBAL STARFIELD -->
    <canvas ref="globalStars" class="global-stars"></canvas>

    <!-- FOREGROUND ASH -->
    <canvas ref="ashLayer" class="ash-layer"></canvas>

    <!-- ================= EMAIL CAPTURE BUTTON ================= -->
    <button class="email-trigger" @click="showEmailModal = true" aria-label="Get notified">
      <span class="email-trigger-icon">✉</span>
      <span class="email-trigger-label">Stay Updated</span>
    </button>

    <!-- ================= EMAIL MODAL ================= -->
    <Transition name="modal">
      <div v-if="showEmailModal" class="modal-backdrop" @click.self="showEmailModal = false">
        <div class="modal-card">
          <button class="modal-close" @click="showEmailModal = false" aria-label="Close">✕</button>
          <p class="modal-eyebrow">THE LAST CAT</p>
          <h2 class="modal-title">Get notified on launch</h2>
          <p class="modal-sub">Enter your email and I'll give you a cookie!</p>

          <div v-if="!emailSubmitted">
            <!-- Netlify form: data-netlify="true" + hidden input are required -->
            <form
              name="notify"
              method="POST"
              data-netlify="true"
              class="email-form"
            >
              <input type="hidden" name="form-name" value="notify" />
              <input
                v-model="emailInput"
                type="email"
                name="email"
                placeholder="your@email.com"
                class="email-input"
                required
                autocomplete="email"
              />
              <button type="submit" class="email-submit" :disabled="submitting">
                <span v-if="submitting">Sending…</span>
                <span v-else>Notify Me</span>
              </button>
            </form>
            <p v-if="emailError" class="email-error">{{ emailError }}</p>
          </div>

          <div v-else class="email-success">
            <span class="success-icon">🐱</span>
            <p>You're on the list! We'll reach out when Timmy is ready.</p>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ================= HERO SECTION ================= -->
    <section class="hero-section">
      <div class="hero-background"></div>
      <div class="hero-content">
        <h1 class="game-title">The Last Cat</h1>
        <p class="game-tagline">Releasing on Steam and iOS in Summer 2026.</p>
        <div class="hero-buttons">
          <button @click="scrollToTrailer" class="primary-btn">Watch Trailer</button>
          <a href="https://thellamaway.itch.io/the-last-cat-on-earth"
             target="_blank"
             class="secondary-btn">
            View on Itch.io
          </a>
        </div>
        <p class="studio-credit">A game by TheLlamaWay Studio</p>
      </div>
    </section>

    <!-- ================= GAME SECTION ================= -->
    <section class="game-section">

      <h2 class="headline">A Heroic Tale in a 2D Platformer</h2>

      <div class="content-grid">

        <!-- STORY -->
        <div class="story-card reveal">
          <p>
            Your friends are gone.
            <br /><br />
            But loss is not the true burden.
            <br /><br />
            Resurrection demands ritual —
            and ritual demands something living.
            <br /><br />
            A cat.
            <br /><br />
            Journey through a desolate, atmospheric world filled with fractured souls and fading memories.
            Confront eternal damnation. Confront isolation.
            Confront the silence that waits when companionship is taken from you.
            <br /><br />
            A heroic tale woven into a haunting 2D platformer.
          </p>

          <div class="itch-wrapper">
            <iframe
              frameborder="0"
              src="https://itch.io/embed/4293036"
              width="552"
              height="167"
              allowfullscreen>
            </iframe>
          </div>
        </div>

        <!-- MEDIA -->
        <div class="media-stack">
          <img :src="screenshot1" class="media-card reveal" alt="Screenshot 1">
          <img :src="screenshot2" class="media-card reveal" alt="Screenshot 2">
          <img :src="gameplayGif" class="media-card reveal" alt="Gameplay">
        </div>

      </div>

      <!-- TRAILER -->
      <div id="trailer" ref="trailerRef" class="trailer reveal">
        <iframe
          src="https://www.youtube.com/embed/_A4sicLw2yA"
          title="The Last Cat Trailer"
          frameborder="0"
          allowfullscreen>
        </iframe>
      </div>

    </section>

    <!-- WHISPER FOOTER -->
    <footer class="whisper-footer">
      <p class="whisper-text">And you Timmy, you shall go down in history as the LAST CAT!</p>
      <p class="whisper-text">The GLORIOUS SAVIOUR of ALL cat kind!</p>
    </footer>

  </div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'

import gameplayGif from './assets/images/gameplay.gif'
import screenshot1 from './assets/images/screenshot1.png'
import screenshot2 from './assets/images/screenshot2.png'

const globalStars = ref(null)
const ashLayer = ref(null)
const trailerRef = ref(null)

/* ================= Email Modal State ================= */
const showEmailModal = ref(false)
const emailInput = ref('')
const emailSubmitted = ref(false)
const submitting = ref(false)
const emailError = ref('')

async function handleEmailSubmit() {
  submitting.value = true
  emailError.value = ''
  try {
    const body = new URLSearchParams({ 'form-name': 'notify', email: emailInput.value })
    const res = await fetch('/', { method: 'POST', headers: { 'Content-Type': 'application/x-www-form-urlencoded' }, body: body.toString() })
    if (res.ok) {
      emailSubmitted.value = true
    } else {
      emailError.value = 'Something went wrong. Please try again.'
    }
  } catch {
    emailError.value = 'Could not connect. Please try again.'
  } finally {
    submitting.value = false
  }
}

const isMobile = window.innerWidth < 768
const STAR_COUNT = isMobile ? 70 : 100
const ASH_COUNT = isMobile ? 70 : 100

let animationRunning = true
let stars = []
let ashParticles = []

/* ================= Scroll to Trailer ================= */
function scrollToTrailer() {
  trailerRef.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

/* ================= Canvas Resize ================= */
function resizeCanvas(canvas) {
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  const width = window.innerWidth
  const height = window.innerHeight
  canvas.style.width = width + 'px'
  canvas.style.height = height + 'px'
  canvas.width = width * dpr
  canvas.height = height * dpr
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
}

/* ================= Reveal Animation ================= */
function initRevealObserver() {
  const elements = document.querySelectorAll('.reveal')
  const observer = new IntersectionObserver(entries => {
    entries.forEach(entry => {
      if (entry.isIntersecting) entry.target.classList.add('visible')
    })
  }, { threshold: 0.1 })
  elements.forEach(el => observer.observe(el))
}

/* ================= Particle Init ================= */
function initStars() {
  stars = Array.from({ length: STAR_COUNT }, () => ({
    x: Math.random() * window.innerWidth,
    y: Math.random() * window.innerHeight,
    r: Math.random() * 2.5 + 0.8,
    speed: Math.random() * 0.4 + 0.05,
    depth: Math.random()
  }))
}

function initAsh() {
  ashParticles = Array.from({ length: ASH_COUNT }, () => ({
    x: Math.random() * window.innerWidth,
    y: Math.random() * window.innerHeight,
    r: Math.random() * 3 + 1,
    speed: Math.random() * 0.3 + 0.05,
    opacity: Math.random() * 0.3 + 0.05
  }))
}

/* ================= Drawing ================= */
function drawStars(ctx, width, height) {
  stars.forEach(star => {
    star.y += star.speed * (1 + star.depth)
    if (star.y > height) { star.y = 0; star.x = Math.random() * width }
    ctx.beginPath(); ctx.arc(star.x, star.y, star.r * 1.8, 0, Math.PI * 2)
    ctx.fillStyle = 'rgba(255,255,255,0.08)'; ctx.fill()
    ctx.beginPath(); ctx.arc(star.x, star.y, star.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(255,255,255,${0.5 + star.depth})`; ctx.fill()
  })
}

function drawAsh(ctx, width, height) {
  ashParticles.forEach(p => {
    p.y -= p.speed
    if (p.y < 0) { p.y = height; p.x = Math.random() * width }
    ctx.beginPath(); ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(220,50,70,${p.opacity})`; ctx.fill()
  })
}

/* ================= Animation Loop ================= */
function animate() {
  if (!animationRunning) return
  const starCanvas = globalStars.value
  const ashCanvas = ashLayer.value
  if (!starCanvas || !ashCanvas) return
  const starCtx = starCanvas.getContext('2d')
  const ashCtx = ashCanvas.getContext('2d')
  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  const width = starCanvas.width / dpr
  const height = starCanvas.height / dpr
  starCtx.clearRect(0, 0, starCanvas.width, starCanvas.height)
  ashCtx.clearRect(0, 0, ashCanvas.width, ashCanvas.height)
  drawStars(starCtx, width, height)
  drawAsh(ashCtx, width, height)
  requestAnimationFrame(animate)
}

/* ================= Resize Handling ================= */
let lastWidth = window.innerWidth
function handleResize() {
  const newWidth = window.innerWidth
  if (newWidth === lastWidth) return
  lastWidth = newWidth
  resizeCanvas(globalStars.value)
  resizeCanvas(ashLayer.value)
}

/* ================= Lifecycle ================= */
onMounted(() => {
  resizeCanvas(globalStars.value)
  resizeCanvas(ashLayer.value)
  initStars()
  initAsh()
  initRevealObserver()
  window.addEventListener('resize', handleResize, { passive: true })
  window.visualViewport?.addEventListener('resize', handleResize)
  requestAnimationFrame(animate)
})

onBeforeUnmount(() => {
  animationRunning = false
  window.removeEventListener('resize', handleResize)
  window.visualViewport?.removeEventListener('resize', handleResize)
})
</script>

<style>

/* ================= Font ================= */
@font-face {
  font-family: 'Jersey';
  src: url('./assets/font/Jersey.ttf') format('truetype');
}

/* ================= Base Layout ================= */

*, *::before, *::after {
  box-sizing: border-box;
}

html {
  margin: 0;
  padding: 0;
  width: 100%;
  /* Prevent horizontal scroll / zoom blowout on mobile */
  overflow-x: hidden;
}

body {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100%;
  overflow-x: hidden;
  background: #020510;
  font-family: 'Jersey', sans-serif;
  color: white;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
  /* Remove the flex centering from style.css that breaks mobile layout */
  display: block !important;
  place-items: unset !important;
}

#app {
  max-width: 100% !important;
  padding: 0 !important;
  text-align: unset !important;
}

.page {
  position: relative;
  width: 100%;
  min-height: 100%;
  background: linear-gradient(to bottom, #020510 0%, #0d1430 100%);
  overflow-x: hidden;
}

/* ================= Canvas Layers ================= */
.global-stars,
.ash-layer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}
.global-stars { z-index: 0; }
.ash-layer    { z-index: 1; }

/* ================= Email Trigger Button ================= */
.email-trigger {
  position: fixed;
  top: 16px;
  left: 16px;
  z-index: 100;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  background: rgba(10, 14, 36, 0.75);
  border: 1px solid rgba(220, 50, 70, 0.5);
  border-radius: 40px;
  color: white;
  font-family: 'Jersey', sans-serif;
  font-size: 1rem;
  letter-spacing: 1.5px;
  cursor: pointer;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  transition: border-color 0.3s ease, box-shadow 0.3s ease, background 0.3s ease;
}
.email-trigger:hover {
  border-color: rgba(220, 50, 70, 0.9);
  box-shadow: 0 0 20px rgba(220, 50, 70, 0.4);
  background: rgba(220, 50, 70, 0.15);
}
.email-trigger-icon {
  font-size: 1rem;
  line-height: 1;
}
.email-trigger-label {
  font-size: 0.9rem;
  letter-spacing: 2px;
}

/* ================= Modal ================= */
.modal-backdrop {
  position: fixed;
  inset: 0;
  z-index: 200;
  background: rgba(2, 5, 16, 0.85);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.modal-card {
  position: relative;
  background: linear-gradient(135deg, rgba(13, 18, 50, 0.98), rgba(20, 10, 30, 0.98));
  border: 1px solid rgba(220, 50, 70, 0.3);
  border-radius: 24px;
  padding: 48px 40px 40px;
  width: 100%;
  max-width: 420px;
  text-align: center;
  box-shadow: 0 0 80px rgba(220, 50, 70, 0.25), 0 20px 60px rgba(0, 0, 0, 0.6);
}

.modal-close {
  position: absolute;
  top: 16px;
  right: 16px;
  background: transparent;
  border: none;
  color: rgba(255,255,255,0.5);
  font-size: 1.1rem;
  cursor: pointer;
  padding: 6px 10px;
  border-radius: 50%;
  transition: color 0.2s, background 0.2s;
  line-height: 1;
}
.modal-close:hover {
  color: white;
  background: rgba(255,255,255,0.08);
}

.modal-eyebrow {
  margin: 0 0 12px;
  font-size: 0.75rem;
  letter-spacing: 4px;
  color: rgba(220, 50, 70, 0.85);
  text-transform: uppercase;
}

.modal-title {
  margin: 0 0 10px;
  font-size: 2rem;
  letter-spacing: 2px;
  line-height: 1.1;
}

.modal-sub {
  margin: 0 0 28px;
  font-size: 0.95rem;
  letter-spacing: 1px;
  color: rgba(255,255,255,0.55);
  line-height: 1.5;
}

/* ================= Email Form ================= */
.email-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.email-input {
  width: 100%;
  padding: 14px 20px;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 12px;
  color: white;
  font-family: 'Jersey', sans-serif;
  font-size: 1rem;
  letter-spacing: 1px;
  outline: none;
  transition: border-color 0.25s, box-shadow 0.25s;
}
.email-input::placeholder { color: rgba(255,255,255,0.3); }
.email-input:focus {
  border-color: rgba(220, 50, 70, 0.7);
  box-shadow: 0 0 16px rgba(220, 50, 70, 0.2);
}

.email-submit {
  width: 100%;
  padding: 14px;
  background: rgba(220, 50, 70, 0.9);
  border: none;
  border-radius: 12px;
  color: white;
  font-family: 'Jersey', sans-serif;
  font-size: 1.05rem;
  letter-spacing: 2px;
  cursor: pointer;
  transition: background 0.3s, box-shadow 0.3s, transform 0.2s;
}
.email-submit:hover:not(:disabled) {
  background: rgba(255, 70, 90, 1);
  box-shadow: 0 0 30px rgba(220, 50, 70, 0.7);
  transform: translateY(-1px);
}
.email-submit:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.email-error {
  font-size: 0.85rem;
  color: rgba(255, 120, 130, 0.9);
  margin: 4px 0 0;
  letter-spacing: 1px;
}

.email-success {
  padding: 12px 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}
.success-icon { font-size: 2.5rem; }
.email-success p {
  margin: 0;
  font-size: 1.05rem;
  letter-spacing: 1px;
  color: rgba(255,255,255,0.8);
  line-height: 1.5;
}

/* ================= Modal Transition ================= */
.modal-enter-active, .modal-leave-active {
  transition: opacity 0.25s ease;
}
.modal-enter-active .modal-card, .modal-leave-active .modal-card {
  transition: transform 0.25s ease, opacity 0.25s ease;
}
.modal-enter-from, .modal-leave-to { opacity: 0; }
.modal-enter-from .modal-card, .modal-leave-to .modal-card {
  transform: scale(0.94) translateY(12px);
  opacity: 0;
}

/* ================= Hero ================= */
.hero-section {
  position: relative;
  height: 100svh; /* Use svh for mobile browser chrome */
  min-height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.hero-background {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(to bottom, rgba(2,5,16,0.4), rgba(2,5,16,0.9)),
    url('./assets/images/background.png') center center no-repeat;
  background-size: cover;
  z-index: 0;
}

.hero-content {
  position: relative;
  z-index: 3;
  text-align: center;
  width: 100%;
  max-width: 860px;
  padding: 20px 24px;
}

.game-title {
  font-size: clamp(2.4rem, 8vw, 5.5rem);
  letter-spacing: clamp(2px, 1.5vw, 8px);
  margin: 0;
  text-transform: uppercase;
  text-shadow: 0 0 40px rgba(220,50,70,0.8);
  line-height: 1.1;
}

.game-tagline {
  font-size: clamp(1rem, 3vw, 1.6rem);
  margin-top: 16px;
  opacity: 0.85;
  letter-spacing: 1px;
}

.hero-buttons {
  margin-top: 36px;
  display: flex;
  justify-content: center;
  gap: 16px;
  flex-wrap: wrap;
}

.primary-btn,
.secondary-btn {
  padding: clamp(10px, 2.5vw, 14px) clamp(24px, 4vw, 36px);
  border-radius: 40px;
  color: white;
  text-decoration: none;
  letter-spacing: 2px;
  font-family: 'Jersey', sans-serif;
  font-size: clamp(0.85rem, 2vw, 1rem);
  transition: all 0.4s ease;
  cursor: pointer;
  white-space: nowrap;
}

.primary-btn {
  background: rgba(220,50,70,0.9);
  border: 1px solid transparent;
}
.primary-btn:hover {
  background: rgb(255,70,90);
  box-shadow: 0 0 35px rgba(220,50,70,0.9);
  transform: translateY(-2px);
}

.secondary-btn {
  border: 1px solid rgba(255,255,255,0.4);
  background: transparent;
}
.secondary-btn:hover {
  border-color: rgba(220,50,70,0.9);
  box-shadow: 0 0 20px rgba(220,50,70,0.6);
  transform: translateY(-2px);
}

.studio-credit {
  margin-top: 52px;
  font-size: 0.85rem;
  letter-spacing: 3px;
  opacity: 0.45;
  text-transform: uppercase;
}

/* ================= Game Section ================= */
.game-section {
  position: relative;
  z-index: 2;
  padding: clamp(48px, 8vw, 100px) clamp(16px, 8%, 10%);
  background:
    linear-gradient(to bottom, rgba(10,15,36,0.95), rgba(20,25,55,0.9)),
    url('./assets/images/background.png');
  background-position: center top;
  background-repeat: no-repeat;
  background-size: cover;
}

.headline {
  text-align: center;
  font-size: clamp(1.7rem, 4.5vw, 3rem);
  margin-bottom: 52px;
  letter-spacing: 2px;
}

.content-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  align-items: start;
}

.story-card {
  background: rgba(10,15,36,0.7);
  padding: clamp(18px, 3vw, 28px);
  border-radius: 22px;
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  border: 0.5px solid rgba(255,255,255,0.07);
  box-shadow: 0 0 60px rgba(220,50,70,0.2);
  font-size: clamp(1rem, 2vw, 1.3rem);
  line-height: 1.4;
}

.story-card p {
  margin: 0 0 24px;
}

/* ================= Itch Iframe ================= */
.itch-wrapper {
  width: 100%;
  max-width: 552px;
  margin: 0 auto;
  overflow: hidden;
  border-radius: 10px;
}

.itch-wrapper iframe {
  /* Scale down on mobile so it never overflows */
  max-width: 100%;
  width: 552px;
  height: 167px;
  display: block;
  transform-origin: top left;
}

@media (max-width: 600px) {
  .itch-wrapper {
    /* Scale iframe to fit container width */
    overflow: hidden;
  }
  .itch-wrapper iframe {
    transform: scale(0.58);
    width: 552px;
    height: 167px;
    margin-bottom: calc(-167px * 0.42);
  }
}

/* ================= Media ================= */
.media-stack {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.media-card {
  width: 100%;
  border-radius: 16px;
  transition: transform 0.5s ease, box-shadow 0.5s ease;
  display: block;
}
.media-card:hover {
  transform: translateY(-10px) scale(1.03);
  box-shadow: 0 20px 60px rgba(0,0,0,0.4);
}

/* ================= Trailer ================= */
.trailer {
  display: flex;
  justify-content: center;
  margin-top: 60px;
}
.trailer iframe {
  width: 100%;
  max-width: 960px;
  aspect-ratio: 16 / 9;
  border-radius: 20px;
  box-shadow: 0 0 80px rgba(220,50,70,0.5);
  transition: transform 0.5s ease;
}
.trailer iframe:hover {
  transform: scale(1.01);
}

/* ================= Footer ================= */
.whisper-footer {
  text-align: center;
  padding: clamp(60px, 10vw, 140px) 20px;
}
.whisper-text {
  font-size: clamp(0.85rem, 2vw, 1.1rem);
  letter-spacing: 3px;
  color: rgba(255,255,255,0.5);
  text-transform: uppercase;
  margin: 8px 0;
}

/* ================= Reveal Animation ================= */
.reveal {
  opacity: 0;
  transform: translateY(50px);
  transition: opacity 0.9s ease, transform 0.9s ease;
}
.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}

/* ================= Responsive ================= */
@media (max-width: 900px) {
  .content-grid {
    grid-template-columns: 1fr;
    gap: 36px;
  }
}

@media (max-width: 480px) {
  .email-trigger-label {
    display: none; /* icon only on very small screens */
  }
  .email-trigger {
    padding: 10px 14px;
  }
  .modal-card {
    padding: 40px 24px 32px;
  }
}

</style>