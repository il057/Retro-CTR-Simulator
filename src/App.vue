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
        <CRTMonitor ref="monitorRef" :config="monitorConfig" />
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
import CRTMonitor from './components/CRTMonitor.vue'
import Controls from './components/Controls.vue'
import { TEXT_PRESETS, COLOR_PRESETS } from './constants.js'

const monitorConfig = reactive({
  text: TEXT_PRESETS[0],
  textColor: COLOR_PRESETS[0].text,
  bgColor: COLOR_PRESETS[0].bg,
  fontFamily: 'ZSFT-2140',
  fontSize: 32,
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

const handleSaveImage = async () => {
  if (!monitorRef.value) return;
  const targetElement = monitorRef.value.$el || monitorRef.value;
  try {
    const dataUrl = await htmlToImage.toPng(targetElement, { 
      quality: 1,
      backgroundColor: monitorConfig.bgColor // Use config bg to prevent white edges
    });
    
    // Check if device is mobile to try native share to Photos
    const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
    
    if (isMobile && navigator.share) {
      const blob = await (await fetch(dataUrl)).blob();
      const file = new File([blob], 'retro-crt-screen.png', { type: 'image/png' });
      if (navigator.canShare && navigator.canShare({ files: [file] })) {
        await navigator.share({
          files: [file],
          title: 'Retro CRT Screen',
        });
        return; // Success, stop here
      }
    }

    // Fallback: Desktop download trigger
    const link = document.createElement('a');
    link.download = 'retro-crt-screen.png';
    link.href = dataUrl;
    link.click();
  } catch (err) {
    console.error('Save image failed', err);
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
