<template>
  <div class="crt-container" :style="{ 
    '--crt-bg': config.bgColor, 
    '--crt-text': config.textColor,
    '--crt-font': `'${config.fontFamily}'`,
    '--crt-font-size': config.fontSize + 'px',
    '--crt-glow': config.glowIntensity + 'px',
    '--crt-chromatic': config.chromaticIntensity + 'px',
    '--crt-align': config.textAlign,
    '--crt-noise-opacity': config.noiseIntensity / 100
  }">
    <!-- SVG 滤镜，用于制造液体/迷幻失真效果 -->
    <svg style="width:0; height:0; position:absolute;">
      <filter id="dreamcore-warp">
        <feTurbulence type="fractalNoise" baseFrequency="0.005" numOctaves="1" result="noise">
          <animate attributeName="baseFrequency" values="0.005; 0.008; 0.005" dur="8s" repeatCount="indefinite" />
        </feTurbulence>
        <feDisplacementMap in="SourceGraphic" in2="noise" scale="12" xChannelSelector="R" yChannelSelector="G" />
      </filter>
    </svg>

    <div 
      class="crt-screen"
      :class="{
        'exporting': isExporting,
        'curvature': config.curvature,
        'vignette': config.vignette,
        'scanlines': config.scanlines
      }"
    >
      <div class="crt-noise"></div>
      
      <!-- Camcorder overlay -->
      <div v-if="config.showCamcorder" class="camcorder-overlay">
        <div class="camcorder-top">
          <span class="camcorder-rec"><span class="rec-dot"></span> REC</span>
          <span>SP</span>
        </div>
        <div class="camcorder-bottom">
          <span>{{ currentTime }}</span>
          <span class="camcorder-battery"><span></span></span>
        </div>
      </div>

      <div class="crt-content" :class="{'dreamcore-active': config.distortion}">
        <div class="text-wrapper">
          <span class="crt-text">{{ config.text }}</span><span v-if="config.showCursor" class="blinking-cursor"></span>
        </div>
      </div>
      <!-- 移动的扫描线横条 -->
      <div v-if="config.scanlines" class="scanline-overlay"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  config: {
    type: Object,
    required: true
  },
  isExporting: {
    type: Boolean,
    default: false
  }
})

const currentTime = ref('')
let timer = null

const updateTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  currentTime.value = `AM ${hours}:${minutes}`
}

onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<style scoped>
.crt-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  max-width: 900px;
  max-height: 70vh; /* Avoid overflowing height on small landscape screens */
  aspect-ratio: 4/3;
  margin: 0 auto;
}

.crt-screen {
  width: 100%;
  height: 100%;
  background-color: var(--crt-bg);
  position: relative;
  overflow: hidden;
  border-radius: 20px; /* 基础微圆角 */
  box-shadow: 
    inset 0 0 20px rgba(0,0,0,0.8),
    0 0 30px rgba(0,0,0,0.5); /* 移除外边框，用阴影隔离 */
}

/* 屏幕雪花噪点效果 (Improvement 1) */
.crt-noise {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  pointer-events: none;
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)'/%3E%3C/svg%3E");
  opacity: var(--crt-noise-opacity); 
  z-index: 6;
  mix-blend-mode: overlay;
}

/* 曲面边缘 / 鱼眼：加强变形 */
.crt-screen.curvature {
  border-radius: 12% / 15%; /* 更明显的物理鱼眼圆角变形 */
  transform: perspective(800px) scale(0.96) rotateX(1deg);
  box-shadow: 
    inset 0px 0px 60px 20px rgba(0,0,0,0.9),
    inset 0px 0px 20px 5px rgba(0,0,0,0.5),
    0 0 40px rgba(0,0,0,0.5);
}

/* 导出时去掉外边框阴影，让成品图更接近“截图感”。 */
.crt-screen.exporting {
  box-shadow: none;
}

.crt-screen.exporting.curvature {
  box-shadow:
    inset 0 0 38px 10px rgba(0,0,0,0.75),
    inset 0 0 16px 4px rgba(0,0,0,0.35);
}

.crt-content {
  color: var(--crt-text);
  font-family: var(--crt-font), monospace;
  font-size: min(var(--crt-font-size), 8vw);
  line-height: 1.5;
  padding: 8% 10%; 
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  text-shadow: 
    var(--crt-chromatic) 0 1px rgba(255, 0, 0, 0.7),
    calc(var(--crt-chromatic) * -1) 0 1px rgba(0, 255, 255, 0.7),
    0 0 var(--crt-glow) var(--crt-text);
  animation: flicker 0.15s infinite;
  z-index: 2;
  position: relative;
  width: 100%;
}

.text-wrapper {
  text-align: var(--crt-align);
  white-space: pre-wrap;
  word-break: break-word;
  width: 100%;
}

.dreamcore-active {
  filter: url(#dreamcore-warp) blur(0.3px);
  transform: scale(1.02); /* Prevents filter bleeding on edges */
}

.crt-text {
  display: inline;
}

.blinking-cursor {
  display: inline-block;
  width: 0.55em;
  height: 1.1em;
  background-color: var(--crt-text);
  margin-left: 0.2em;
  vertical-align: text-bottom;
  position: relative;
  /* Top offset removed and adjusted alignment, using background rather than full block glyph */
  animation: blink 1s step-end infinite;
  box-shadow: 0 0 var(--crt-glow) var(--crt-text);
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

/* 扫描线背景 */
.crt-screen.scanlines::before {
  content: " ";
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background: linear-gradient(
    to bottom,
    rgba(255,255,255,0),
    rgba(255,255,255,0) 50%,
    rgba(0,0,0,0.15) 50%,
    rgba(0,0,0,0.15)
  );
  background-size: 100% 4px;
  z-index: 3;
  pointer-events: none;
}

.scanline-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 6px;
  background: rgba(255, 255, 255, 0.04);
  opacity: 0.6;
  animation: scanline 6s linear infinite;
  z-index: 4;
  pointer-events: none;
}

/* Camcorder UI */
.camcorder-overlay {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  padding: 5% 8%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  color: var(--crt-text);
  font-family: 'Courier New', Courier, monospace;
  font-size: clamp(1rem, 3vw, 1.8rem);
  font-weight: bold;
  z-index: 3;
  pointer-events: none;
  text-shadow: 0 0 calc(var(--crt-glow) / 2) var(--crt-text);
  opacity: 0.85;
}

.camcorder-top, .camcorder-bottom {
  display: flex;
  justify-content: space-between;
}

.camcorder-rec {
  display: flex;
  align-items: center;
  gap: 8px;
  animation: flicker 0.1s infinite;
}

.rec-dot {
  width: 0.6em;
  height: 0.6em;
  background-color: var(--crt-text);
  border-radius: 50%;
  animation: blink 1s step-end infinite;
}

.camcorder-battery {
  width: 2em;
  height: 1em;
  border: 2px solid var(--crt-text);
  padding: 2px;
  position: relative;
}

.camcorder-battery::after {
  content: '';
  position: absolute;
  right: -5px;
  top: 25%;
  height: 50%;
  width: 3px;
  background: var(--crt-text);
}

.camcorder-battery span {
  display: block;
  height: 100%;
  width: 70%;
  background-color: var(--crt-text);
  animation: blink 2s step-end infinite;
}

/* 暗角 */
.crt-screen.vignette::after {
  content: " ";
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background: radial-gradient(
    circle,
    rgba(0,0,0,0) 50%,
    rgba(0,0,0,0.85) 120%
  );
  z-index: 5;
  pointer-events: none;
}

@media (max-width: 900px) {
  .crt-content {
    font-size: min(var(--crt-font-size), 7vw);
  }
}

@media (max-width: 600px) {
  .crt-content {
    font-size: min(var(--crt-font-size), 6.2vw);
    line-height: 1.38;
    padding: 9% 9%;
  }
}
</style>
