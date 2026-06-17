<template>
  <div class="controls">
    <h3>[ TERMINAL CONTROLS ]</h3>
    
    <div class="control-section">
      <label class="section-title">>> SYSTEM TEXT</label>
      <textarea v-model="config.text" rows="4"></textarea>
      <div class="action-row">
        <button class="retro-btn" @click="randomizeText">
          <Icon icon="pixelarticons:dice" class="btn-icon" /> RANDOMIZE
        </button>
        <button class="retro-btn save-btn" @click="$emit('save-image')">
          <Icon icon="pixelarticons:image" class="btn-icon" /> SAVE IMAGE
        </button>
      </div>
    </div>

    <div class="control-section">
      <label class="section-title">>> FONT STYLE</label>
      <div class="flex-column">
        <div class="radio-label" v-for="font in FONT_PRESETS" :key="font.value" @click="config.fontFamily = font.value">
          <Icon class="ui-icon" :icon="config.fontFamily === font.value ? 'pixelarticons:bullseye' : 'pixelarticons:circle'" /> 
          <span>{{ font.name }}</span>
        </div>
      </div>

      <div class="slider-group">
        <label>SIZE: <span class="val-display">{{ config.fontSize }}px</span></label>
        <input type="range" class="retro-range" v-model="config.fontSize" min="16" max="100" step="1">
      </div>

      <div class="action-row" style="margin-top: 15px; margin-bottom: 5px;">
        <button class="retro-btn" style="flex: 1" :class="{'active-align': config.textAlign === 'left'}" @click="config.textAlign = 'left'">
          <Icon icon="pixelarticons:align-left" class="btn-icon" />
        </button>
        <button class="retro-btn" style="flex: 1" :class="{'active-align': config.textAlign === 'center'}" @click="config.textAlign = 'center'">
          <Icon icon="pixelarticons:align-center" class="btn-icon" />
        </button>
        <button class="retro-btn" style="flex: 1" :class="{'active-align': config.textAlign === 'right'}" @click="config.textAlign = 'right'">
          <Icon icon="pixelarticons:align-right" class="btn-icon" />
        </button>
      </div>
      
      <div class="slider-group">
        <label>GLOW: <span class="val-display">{{ config.glowIntensity }}px</span></label>
        <input type="range" class="retro-range" v-model="config.glowIntensity" min="0" max="25" step="1">
      </div>
    </div>

    <div class="control-section">
      <label class="section-title">>> HARDWARE RENDERING</label>
      <div class="toggles">
        <div class="checkbox-label" @click="config.showCursor = !config.showCursor">
          <Icon class="ui-icon" :icon="config.showCursor ? 'pixelarticons:checkbox-on' : 'pixelarticons:checkbox'" />
          <span>BLINKING CURSOR</span>
        </div>
        <div class="checkbox-label" @click="config.scanlines = !config.scanlines">
          <Icon class="ui-icon" :icon="config.scanlines ? 'pixelarticons:checkbox-on' : 'pixelarticons:checkbox'" />
          <span>SCANLINES</span>
        </div>
        <div class="checkbox-label" @click="config.vignette = !config.vignette">
          <Icon class="ui-icon" :icon="config.vignette ? 'pixelarticons:checkbox-on' : 'pixelarticons:checkbox'" />
          <span>VIGNETTE overlay</span>
        </div>
        <div class="checkbox-label" @click="config.curvature = !config.curvature">
          <Icon class="ui-icon" :icon="config.curvature ? 'pixelarticons:checkbox-on' : 'pixelarticons:checkbox'" />
          <span>CRT CURVATURE lens</span>
        </div>
        <div class="checkbox-label" @click="config.distortion = !config.distortion">
          <Icon class="ui-icon" :icon="config.distortion ? 'pixelarticons:checkbox-on' : 'pixelarticons:checkbox'" />
          <span>DREAMCORE WARP</span>
        </div>
        <div class="checkbox-label" @click="config.showCamcorder = !config.showCamcorder">
          <Icon class="ui-icon" :icon="config.showCamcorder ? 'pixelarticons:checkbox-on' : 'pixelarticons:checkbox'" />
          <span>CAMCORDER MODE</span>
        </div>
      </div>

      <div class="slider-group crt-offset-slider">
        <label>CHROMATIC: <span class="val-display">{{ config.chromaticIntensity }}px</span></label>
        <input type="range" class="retro-range" v-model="config.chromaticIntensity" min="0" max="10" step="1">
      </div>
      
      <div class="slider-group">
        <label>GRAIN NOISE: <span class="val-display">{{ config.noiseIntensity }}%</span></label>
        <input type="range" class="retro-range" v-model="config.noiseIntensity" min="0" max="50" step="1">
      </div>
    </div>

    <div class="control-section">
      <label class="section-title">>> CUSTOM COLOR</label>
      <div class="color-row">
        <label class="color-pick-wrap">TEXT <input type="color" class="color-picker" v-model="config.textColor"></label>
        <label class="color-pick-wrap">BG <input type="color" class="color-picker" v-model="config.bgColor"></label>
      </div>
    </div>

    <div class="control-section">
      <label class="section-title">>> QUICK PRESETS</label>
      <div class="preset-buttons">
        <button 
          v-for="(preset, index) in COLOR_PRESETS" 
          :key="index"
          @click="applyColor(preset)"
          class="retro-btn preset-btn"
          :style="{ background: preset.bg, color: preset.text, borderColor: preset.text }"
        >
          {{ preset.name }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { Icon } from '@iconify/vue'
import { TEXT_PRESETS, COLOR_PRESETS, FONT_PRESETS } from '../constants.js'

const props = defineProps({
  config: {
    type: Object,
    required: true
  }
})

defineEmits(['save-image'])

const randomizeText = () => {
  const randomIndex = Math.floor(Math.random() * TEXT_PRESETS.length)
  props.config.text = TEXT_PRESETS[randomIndex]
}

const applyColor = (preset) => {
  props.config.textColor = preset.text
  props.config.bgColor = preset.bg
}
</script>

<style scoped>
.controls {
  padding: 30px;
  color: var(--ui-primary);
  font-family: inherit;
  font-weight: normal;
  transition: color 0.3s;
}

h3 {
  margin-top: 0;
  margin-bottom: 30px;
  border-bottom: 2px dashed #444;
  padding-bottom: 10px;
  font-size: 1.4rem;
  letter-spacing: 2px;
}

.control-section {
  margin-bottom: 30px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.section-title {
  color: #000;
  background: var(--ui-primary);
  display: inline-block;
  padding: 4px 10px;
  margin-bottom: 5px;
  font-weight: bold;
  transition: background 0.3s;
}

textarea {
  width: 100%;
  padding: 12px;
  background: #000;
  color: inherit;
  border: 1px solid #444;
  border-left: 5px solid var(--ui-primary);
  font-family: inherit;
  font-size: 1rem;
  resize: vertical;
  outline: none;
  transition: all 0.3s;
}

textarea:focus {
  border-color: var(--ui-primary);
  box-shadow: 0 0 10px var(--ui-primary-glow);
}

.action-row {
  display: flex;
  gap: 10px;
}

.retro-btn {
  cursor: pointer;
  padding: 10px;
  background: #000;
  color: var(--ui-primary);
  border: 1px solid var(--ui-primary);
  font-family: inherit;
  text-transform: uppercase;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.btn-icon {
  font-size: 1.2rem;
}

.save-btn {
  flex: 1;
  font-weight: bold;
}

.retro-btn:hover {
  background: var(--ui-primary);
  color: #000;
  box-shadow: 0 0 15px var(--ui-primary-glow);
}

.active-align {
  background: var(--ui-primary);
  color: #000;
  box-shadow: inset 0 0 10px rgba(0,0,0,0.5);
}

.flex-column {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding-bottom: 10px;
}

.ui-icon {
  font-size: 1.3rem;
  color: var(--ui-primary);
  flex-shrink: 0;
  margin-right: 4px;
}

.radio-label, .checkbox-label {
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
  user-select: none;
  transition: opacity 0.2s;
}

.radio-label:active, .checkbox-label:active {
  opacity: 0.6;
}

.slider-group {
  display: flex;
  align-items: center;
  gap: 15px;
}

.slider-group label {
  min-width: 140px;
  display: flex;
  justify-content: space-between;
}

.val-display {
  display: inline-block;
  min-width: 35px;
  text-align: right;
  opacity: 0.7;
}

.crt-offset-slider {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px dashed #333;
}

.retro-range {
  appearance: none;
  -webkit-appearance: none;
  background: #222;
  height: 4px;
  flex: 1;
  outline: none;
}

.retro-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 16px;
  height: 16px;
  background: var(--ui-primary);
  cursor: pointer;
  border-radius: 0;
  transition: background 0.3s;
}

.color-row {
  display: flex;
  align-items: center;
  gap: 30px;
}

.color-row label {
  display: flex;
  align-items: center;
  gap: 10px;
}

.color-picker {
  appearance: none;
  -webkit-appearance: none;
  border: 1px solid #555;
  border-radius: 0;
  background: #000;
  width: 46px;
  min-width: 46px;
  height: 30px;
  padding: 0;
  cursor: pointer;
  overflow: hidden;
}

.color-picker::-webkit-color-swatch-wrapper {
  padding: 2px;
}

.color-picker::-webkit-color-swatch {
  border: none;
  border-radius: 0;
}

.color-picker::-moz-color-swatch {
  border: none;
  border-radius: 0;
}

.preset-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.preset-btn {
  border-width: 1px;
  opacity: 0.8;
  transition: opacity 0.2s, filter 0.2s;
}

.preset-btn:hover {
  opacity: 1;
  filter: brightness(1.2);
  background: auto !important; /* Ignore retro-btn override for hover */
  color: auto;
  box-shadow: 0 0 8px rgba(255,255,255,0.2);
}

.toggles {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.toggles label {
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
}

.toggles input[type="checkbox"] {
  accent-color: var(--ui-primary);
  width: 16px;
  height: 16px;
  cursor: pointer;
}
</style>
