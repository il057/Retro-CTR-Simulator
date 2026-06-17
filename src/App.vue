<template>
  <div class="app-wrapper">
    <header class="app-header">
      <div class="header-content">
        <h1>RETRO CRT</h1>
        <span class="pulse-dot"></span>
      </div>
      <p class="subtitle">// TERMINAL CONFIGURATION //</p>
    </header>
    <div class="main-content">
      <div class="monitor-section">
        <!-- ref is attached to CRTMonitor component, we use $el or target the wrapper -->
        <CRTMonitor ref="monitorRef" :config="monitorConfig" :is-exporting="isExporting" />
      </div>
      <div class="controls-section retro-scroll">
        <Controls :config="monitorConfig" @save-image="handleSaveImage" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, watchEffect } from 'vue'
import * as htmlToImage from 'html-to-image'
import html2canvas from 'html2canvas'
import CRTMonitor from './components/CRTMonitor.vue'
import Controls from './components/Controls.vue'
import { TEXT_PRESETS, COLOR_PRESETS } from './constants.js'

const monitorConfig = reactive({
  text: TEXT_PRESETS[0],
  textColor: COLOR_PRESETS[0].text,
  bgColor: COLOR_PRESETS[0].bg,
  fontFamily: 'ZSFT-2140',
  fontSize: window.matchMedia('(max-width: 900px)').matches ? 24 : 32,
  textAlign: 'left',
  glowIntensity: 5,
  showCursor: true,
  scanlines: true,
  vignette: true,
  curvature: true,
  chromaticIntensity: 2,
  distortion: false,
  noiseIntensity: 8,
  showCamcorder: false
})

const monitorRef = ref(null)
const isExporting = ref(false)
let fontEmbedCSSCache = null

const isIOS = () => /iPhone|iPad|iPod/i.test(navigator.userAgent)
const isMobileDevice = () => /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)

const waitForFonts = async () => {
  if (!document.fonts) return
  try {
    await document.fonts.ready
    await Promise.all([
      document.fonts.load(`400 ${monitorConfig.fontSize}px "${monitorConfig.fontFamily}"`),
      document.fonts.load(`700 ${monitorConfig.fontSize}px "${monitorConfig.fontFamily}"`)
    ])
  } catch (error) {
    console.warn('Font loading check skipped:', error)
  }
}

const buildExportBlob = async (targetElement) => {
  // iOS Safari 对 html-to-image 兼容性不稳定，优先走 html2canvas。
  if (isIOS()) {
    const canvas = await html2canvas(targetElement, {
      backgroundColor: monitorConfig.bgColor,
      useCORS: true,
      scale: Math.min(window.devicePixelRatio || 1, 2),
      logging: false
    })
    return await new Promise((resolve) => canvas.toBlob(resolve, 'image/png', 1))
  }

  if (!fontEmbedCSSCache) {
    fontEmbedCSSCache = await htmlToImage.getFontEmbedCSS(targetElement)
  }

  const dataUrl = await htmlToImage.toPng(targetElement, {
    quality: 1,
    pixelRatio: Math.min(window.devicePixelRatio || 1, 2),
    cacheBust: true,
    skipAutoScale: true,
    fontEmbedCSS: fontEmbedCSSCache,
    backgroundColor: monitorConfig.bgColor,
    preferredFontFormat: 'woff2'
  })

  return await (await fetch(dataUrl)).blob()
}

const handleSaveImage = async () => {
  if (!monitorRef.value) return
        // 1. 先获取组件根节点
        const rootNode = monitorRef.value.$el || monitorRef.value
        // 2. 向下查询真正带有圆角的 .crt-screen 元素
        const targetElement = rootNode.querySelector('.crt-screen')

        // 保存原始样式
        const originalBorderRadius = targetElement.style.borderRadius;
        const originalBoxShadow = targetElement.style.boxShadow;
        const originalTransform = targetElement.style.transform;

        // 临时移除圆角和阴影
        targetElement.style.borderRadius = '0px';
        targetElement.style.boxShadow = 'none';
        targetElement.style.transform = 'none';
   
  try {
    await waitForFonts()
    isExporting.value = true
    await new Promise((resolve) => requestAnimationFrame(resolve))

    const blob = await buildExportBlob(targetElement)
    if (!blob) {
      throw new Error('Canvas toBlob failed')
    }

    const file = new File([blob], 'retro-crt-screen.png', { type: 'image/png' })

    if (isMobileDevice() && navigator.share) {
      if (navigator.canShare && navigator.canShare({ files: [file] })) {
        await navigator.share({
          files: [file],
          title: 'Retro CRT Screen',
        })
        return
      }
    }

    const link = document.createElement('a')
    link.download = 'retro-crt-screen.png'
    link.href = URL.createObjectURL(blob)
    link.click()
    URL.revokeObjectURL(link.href)
  } catch (err) {
    console.error('Save image failed', err);
  } finally {
    isExporting.value = false
    targetElement.style.borderRadius = originalBorderRadius;
    targetElement.style.boxShadow = originalBoxShadow;
    targetElement.style.transform = originalTransform;
  }
}

// Write the primary UI color to the document root so the global scrollbar picks it up
watchEffect(() => {
  let baseRGB = monitorConfig.textColor;
  if (!baseRGB.startsWith('#')) {
    baseRGB = '#33ff00';
  }
  document.documentElement.style.setProperty('--ui-primary', baseRGB);
  document.documentElement.style.setProperty('--ui-primary-glow', baseRGB + '40');
  document.documentElement.style.setProperty('--ui-text-muted', baseRGB + '99');
})
</script>

<style>
/* Global Retro Scrollbar for the App UI */
::-webkit-scrollbar {
  width: 14px;
  background: #000;
  border-left: 1px solid #222;
}

::-webkit-scrollbar-thumb {
  background: var(--ui-primary, #33ff00);
  border: 4px solid #000;
  border-radius: 0;
}

::-webkit-scrollbar-thumb:hover {
  background: #fff;
}
</style>

<style scoped>
.app-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
  background: #000;
}

.app-header {
  padding: 15px 30px;
  background: #050505;
  border-bottom: 2px solid var(--ui-primary);
  box-shadow: 0 4px 10px var(--ui-primary-glow);
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 10;
  transition: all 0.3s;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 15px;
}

.app-header h1 {
  color: var(--ui-primary);
  font-size: 1.8rem;
  margin: 0;
  text-shadow: 0 0 8px var(--ui-primary-glow);
  letter-spacing: 2px;
  transition: color 0.3s, text-shadow 0.3s;
}

.pulse-dot {
  width: 12px;
  height: 12px;
  background-color: var(--ui-primary);
  border-radius: 50%;
  box-shadow: 0 0 8px var(--ui-primary);
  animation: pulse 1.5s infinite alternate;
}

@keyframes pulse {
  0% { opacity: 1; }
  100% { opacity: 0.3; }
}

.subtitle {
  color: var(--ui-text-muted);
  font-size: 1rem;
  margin: 0;
  transition: color 0.3s;
}

.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.monitor-section {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: #020202;
  background-image: 
    linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px);
  background-size: 20px 20px;
}

.controls-section {
  width: 450px;
  min-width: 350px;
  overflow-y: auto;
  border-left: 1px dashed var(--ui-primary);
  background: #0a0a0a;
  transition: border-color 0.3s;
}

@media (max-width: 900px) {
  .app-wrapper {
    height: auto;
    min-height: 100vh;
  }
  .app-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
  .main-content {
    flex-direction: column;
  }
  .controls-section {
    width: 100%;
    border-left: none;
    border-top: 1px dashed var(--ui-primary);
  }
  .monitor-section {
    min-height: 50vh;
  }
}
</style>
