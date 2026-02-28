<template>
  <div class="page">

    <!-- GLOBAL STARFIELD -->
    <canvas ref="globalStars" class="global-stars"></canvas>

    <!-- FOREGROUND ASH -->
    <canvas ref="ashLayer" class="ash-layer"></canvas>

    <!-- ================= HERO SECTION ================= -->
<section class="hero-section">

  <div class="hero-background"></div>

  <div class="hero-content">
    <h1 class="game-title">The Last Cat</h1>
    <p class="game-tagline">
      Releasing on Steam and IOS in Summer 2026.
    </p>

    <div class="hero-buttons">
	    <button @click="scrollToTrailer" class="primary-btn">
		    Watch Trailer
	    </button>
      <a href="https://thellamaway.itch.io/the-last-cat-on-earth" 
         target="_blank" 
         class="secondary-btn">
         View on Itch.io
      </a>
    </div>

    <p class="studio-credit">
      A game by TheLlamaWay Studio
    </p>
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
    height="167">
  </iframe>
  <a href="https://thellamaway.itch.io/the-last-cat-on-earth">
    The Last Cat (on earth) by TheLlamaWay
  </a>
</div>

        </div>

        <!-- MEDIA -->
        <div class="media-stack">
          <img :src="screenshot1" class="media-card reveal">
          <img :src="screenshot2" class="media-card reveal">
          <img :src="gameplayGif" class="media-card reveal">
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

const isMobile = window.innerWidth < 768

const STAR_COUNT = isMobile ? 70 : 100
const ASH_COUNT = isMobile ? 70: 100

let animationRunning = true
let stars = []
let ashParticles = []
let resizeTimeout

const trailerRef = ref(null)

function scrollToTrailer() {
  trailerRef.value?.scrollIntoView({
    behavior: 'smooth',
    block: 'start'
  })
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

function initRevealObserver() {
  const elements = document.querySelectorAll('.reveal')

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible')
      }
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

    if (star.y > height) {
      star.y = 0
      star.x = Math.random() * width
    }

    // soft glow (no shadowBlur)
    ctx.beginPath()
    ctx.arc(star.x, star.y, star.r * 1.8, 0, Math.PI * 2)
    ctx.fillStyle = 'rgba(255,255,255,0.08)'
    ctx.fill()

    ctx.beginPath()
    ctx.arc(star.x, star.y, star.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(255,255,255,${0.5 + star.depth})`
    ctx.fill()
  })
}

function drawAsh(ctx, width, height) {
  ashParticles.forEach(p => {
    p.y -= p.speed

    if (p.y < 0) {
      p.y = height
      p.x = Math.random() * width
    }

    ctx.beginPath()
    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(220,50,70,${p.opacity})`
    ctx.fill()
  })
}

/* ================= Single Animation Loop ================= */

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

  // Ignore height-only changes (mobile address bar scroll)
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
  initRevealObserver() // ← add this back

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

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100%;
  background: #020510;
}

body {
  font-family: 'Jersey', sans-serif;
  color: white;
  overflow-x: clip;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

/* Main wrapper gradient */
.page {
  position: relative;
  width: 100%;
  min-height: 100%;
  background: linear-gradient(to bottom, #020510 0%, #0d1430 100%);
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
.ash-layer { z-index: 1; }

.hero-section {
  position: relative;
  height: 100vh;
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
  max-width: 900px;
  padding: 20px;
}

.game-title {
  font-size: 5rem;
  letter-spacing: 6px;
  margin: 0;
  text-transform: uppercase;
  text-shadow: 0 0 40px rgba(220,50,70,0.8);
}

.game-tagline {
  font-size: 1.6rem;
  margin-top: 20px;
  opacity: 0.85;
}

.hero-buttons {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
}

.primary-btn {
  padding: 14px 36px;
  background: rgba(220,50,70,0.9);
  border-radius: 40px;
  color: white;
  text-decoration: none;
  letter-spacing: 2px;
  transition: all 0.4s ease;
}

.primary-btn:hover {
  background: rgba(255,70,90,1);
  box-shadow: 0 0 30px rgba(220,50,70,0.9);
}

.secondary-btn {
  padding: 14px 36px;
  border: 1px solid rgba(255,255,255,0.4);
  border-radius: 40px;
  color: white;
  text-decoration: none;
  letter-spacing: 2px;
  transition: all 0.4s ease;
}

.secondary-btn:hover {
  border-color: rgba(220,50,70,0.9);
  box-shadow: 0 0 20px rgba(220,50,70,0.6);
}

.studio-credit {
  margin-top: 60px;
  font-size: 0.9rem;
  letter-spacing: 3px;
  opacity: 0.5;
}

/* ================= Buttons ================= */

.hero-buttons {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
}

.primary-btn,
.secondary-btn {
  padding: 14px 36px;
  border-radius: 40px;
  color: white;
  text-decoration: none;
  letter-spacing: 2px;
  transition: all 0.4s ease;
}

.primary-btn {
  background: rgba(220,50,70,0.9);
}

.primary-btn:hover {
  background: rgb(255,70,90);
  box-shadow: 0 0 35px rgba(220,50,70,0.9);
  transform: translateY(-2px);
}

.secondary-btn {
  border: 1px solid rgba(255,255,255,0.4);
}

.secondary-btn:hover {
  border-color: rgba(220,50,70,0.9);
  box-shadow: 0 0 20px rgba(220,50,70,0.6);
}

/* ================= Game Section ================= */

.game-section {
  position: relative;
  z-index: 2;
  padding: 80px 10%;

  background:
    linear-gradient(to bottom, rgba(10,15,36,0.95), rgba(20,25,55,0.9)),
    url('./assets/images/background.png');

  background-position: center top;
  background-repeat: no-repeat;
  background-size: cover;
}

.headline {
  text-align: center;
  font-size: 3rem;
  margin-bottom: 60px;
}

.content-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 80px;
}

.story-card {
  background: rgba(10,15,36,0.7);
  padding: 30px;
  border-radius: 22px;
  backdrop-filter: blur(14px);
  border: 0.5px solid rgba(255,255,255,0.05);
  box-shadow: 0 0 60px rgba(220,50,70,0.25);
  font-size: 1.3rem;
  line-height: 1.2;
}

/* ================= Media ================= */

.media-stack {
  display: flex;
  flex-direction: column;
  gap: 50px;
}

.media-card {
  width: 100%;
  border-radius: 20px;
  transition: transform 0.5s ease;
}

.media-card:hover {
  transform: translateY(-16px) scale(1.05);
}

/* ================= Trailer ================= */

.trailer iframe {
  margin-top: 40px;
  width: 100%;
  aspect-ratio: 16 / 9;
  max-width: 1000px;
  border-radius: 24px;
  box-shadow: 0 0 80px rgba(220,50,70,0.6);
  transition: transform 0.5s ease;
}

.trailer iframe:hover {
  transform: scale(1.02);
}

/* ================= Footer ================= */


.whisper-footer { text-align: center; padding: 140px 20px; } .whisper-text { font-size: 1.1rem; letter-spacing: 4px; color: rgba(255,255,255,0.6); text-transform: uppercase; }

/* ================= Reveal Animation ================= */

.reveal {
  opacity: 0;
  transform: translateY(60px);
  transition: all 1s ease;
}

.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}

/* ================= Responsive ================= */

@media (max-width: 1000px) {
  .content-grid {
    grid-template-columns: 1fr;
    gap: 40px;
  }

  .trailer iframe {
    height: 350px;
  }
}

@media (max-width: 768px) {
  .game-title {
    font-size: 2.8rem;
    letter-spacing: 3px;
  }

  .headline {
    font-size: 2rem;
  }

  .story-card {
    font-size: 1.05rem;
    padding: 28px;
  }

.game-section {
    padding: 40px 15px; /* Use small fixed padding instead of percentages */
  }

  .content-grid {
    display: block; /* Force stack instead of grid to be safe */
    width: 100%;
  }

  .story-card {
    width: auto;
    word-wrap: break-word; /* Prevents long words from pushing the box wide */
}
}

</style>

