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
      <a href="#trailer" class="primary-btn">Watch Trailer</a>
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
      <div id="trailer" class="trailer reveal">
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

const STAR_COUNT = isMobile ? 120 : 250
const ASH_COUNT = isMobile ? 60 : 120

let animationRunning = true
let stars = []
let ashParticles = []
let resizeTimeout

/* ================= Canvas Utilities ================= */

function resizeCanvas(canvas) {
  if (!canvas) return

  const ctx = canvas.getContext('2d')
  const dpr = Math.min(window.devicePixelRatio || 1, 2)

  const width = window.innerWidth
  const height = window.innerHeight

  canvas.style.width = width + "px"
  canvas.style.height = height + "px"

  canvas.width = width * dpr
  canvas.height = height * dpr

  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
}

/* ================= Particle Initialization ================= */

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
    y: Math.random() * document.documentElement.scrollHeight,
    r: Math.random() * 3 + 1,
    speed: Math.random() * 0.3 + 0.05,
    opacity: Math.random() * 0.3 + 0.05
  }))
}

/* ================= Animation Loops ================= */

function animateStars() {
  if (!animationRunning) return

  const canvas = globalStars.value
  const ctx = canvas?.getContext('2d')
  if (!ctx) return

  ctx.clearRect(0, 0, canvas.width, canvas.height)

  ctx.shadowBlur = 12
  ctx.shadowColor = "white"

  const height = canvas.height / (window.devicePixelRatio || 1)

  stars.forEach(star => {
    star.y += star.speed * (1 + star.depth)

    if (star.y > height) {
      star.y = 0
      star.x = Math.random() * window.innerWidth
    }

    ctx.beginPath()
    ctx.arc(star.x, star.y, star.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(255,255,255,${0.4 + star.depth})`
    ctx.fill()
  })

  requestAnimationFrame(animateStars)
}

function animateAsh() {
  if (!animationRunning) return

  const canvas = ashLayer.value
  const ctx = canvas?.getContext('2d')
  if (!ctx) return

  ctx.clearRect(0, 0, canvas.width, canvas.height)

  const height = canvas.height / (window.devicePixelRatio || 1)

  ashParticles.forEach(p => {
    p.y -= p.speed

    if (p.y < 0) p.y = height

    ctx.beginPath()
    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
    ctx.fillStyle = `rgba(220,50,70,${p.opacity})`
    ctx.fill()
  })

  requestAnimationFrame(animateAsh)
}

/* ================= Resize Handling ================= */

function handleResize() {
  clearTimeout(resizeTimeout)

  resizeTimeout = setTimeout(() => {
    resizeCanvas(globalStars.value)
    resizeCanvas(ashLayer.value)

    initStars()
    initAsh()
  }, 150)
}

/* ================= Reveal Observer ================= */

function initRevealObserver() {
  const observer = new IntersectionObserver(entries => {
    entries.forEach(entry => {
      if (entry.isIntersecting)
        entry.target.classList.add('visible')
    })
  }, { threshold: 0.15 })

  document.querySelectorAll('.reveal')
    .forEach(el => observer.observe(el))
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

  requestAnimationFrame(animateStars)
  requestAnimationFrame(animateAsh)
})

onBeforeUnmount(() => {
  animationRunning = false
  window.removeEventListener('resize', handleResize)
})

</script>

<style>

@font-face {
  font-family: 'Jersey';
  src: url('./assets/font/Jersey.ttf') format('truetype');
}

html {
  scroll-behavior: smooth;
}

#trailer {
  scroll-margin-top: 120px;
}

/* ================= Layout Base ================= */

body {
  margin: 0;
  font-family: 'Jersey', sans-serif;
  background: linear-gradient(to bottom, #020510 0%, #0d1430 100%);
  color: white;
  overflow-x: hidden;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

/* ================= Canvas Layers ================= */

.global-stars,
.ash-layer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
}

.global-stars { z-index: 0; }
.ash-layer { z-index: 2; }

/* ================= Hero ================= */

.hero-section {
  position: relative;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.hero-background { position: absolute; inset: 0; background: linear-gradient(to bottom, rgba(2,5,16,0.4), rgba(2,5,16,0.9)), url('./assets/images/background.png') center center no-repeat; background-size: cover; z-index: 0; }

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
  animation: glowPulse 3s ease-in-out infinite;
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

@keyframes glowPulse {
  0%,100% { box-shadow: 0 0 15px rgba(220,50,70,0.4); }
  50% { box-shadow: 0 0 35px rgba(220,50,70,0.7); }
}

/* ================= Game Section ================= */

.game-section {
  padding: 80px 10%;
  background:
    linear-gradient(to bottom, rgba(10,15,36,0.95), rgba(20,25,55,0.9)),
    url('./assets/images/background.png') center top no-repeat;
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
  will-change: transform;
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

.whisper-footer {
  text-align: center;
  padding: 140px 20px;
  opacity: 0;
  animation: fadeInWhisper 3s ease forwards 2s;
}

.whisper-text {
  font-size: 1.1rem;
  letter-spacing: 4px;
  color: rgba(255,255,255,0.6);
  text-transform: uppercase;
}

@keyframes fadeInWhisper {
  to { opacity: 1; }
}

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
    padding: 60px 6%;
  }
}

</style>
