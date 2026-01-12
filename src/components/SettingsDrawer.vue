<script setup>
import { ref, watch, onMounted } from 'vue';

const props = defineProps(['isOpen']);
const emit = defineEmits(['close', 'update:theme', 'update:color', 'update:timeFormat']);

const primaryColor = ref('#6200ee');
const selectedThemeMode = ref('system');
const selectedTimeFormat = ref('24h');

const colors = [
  '#6200ee', '#3700b3', '#03dac6', '#018786',
  '#b00020', '#cf6679', '#FF4081', '#E040FB',
  '#536DFE', '#448AFF', '#40C4FF', '#18FFFF',
  '#64FFDA', '#69F0AE', '#B2FF59', '#EEFF41',
  '#FFFF00', '#FFD740', '#FFAB40', '#FF6E40'
];

onMounted(() => {
  const savedColor = localStorage.getItem('primary-color');
  const savedTheme = localStorage.getItem('theme-mode'); // Changed key to differentiate
  
  if (savedColor) primaryColor.value = savedColor;
  if (savedTheme) selectedThemeMode.value = savedTheme;
  const savedTimeFormat = localStorage.getItem('time-format');
  if (savedTimeFormat) selectedTimeFormat.value = savedTimeFormat;
});

watch(primaryColor, (newColor) => {
  emit('update:color', newColor);
  localStorage.setItem('primary-color', newColor);
});

watch(selectedThemeMode, (newMode) => {
  emit('update:theme', newMode);
  localStorage.setItem('theme-mode', newMode);
});

watch(selectedTimeFormat, (newFormat) => {
  emit('update:timeFormat', newFormat);
  localStorage.setItem('time-format', newFormat);
});

const selectColor = (color) => {
  primaryColor.value = color;
};
</script>

<template>
  <div 
    class="settings-drawer" 
    :class="{ open: isOpen }"
  >
    <div class="header">
      <h2>Appearance</h2>
      <button class="close-btn" @click="$emit('close')">&times;</button>
    </div>

    <div class="section">
      <h3>Theme</h3>
      <div class="theme-options">
        <label class="theme-option">
          <input type="radio" value="light" v-model="selectedThemeMode">
          <span class="option-label">Light</span>
        </label>
        <label class="theme-option">
          <input type="radio" value="dark" v-model="selectedThemeMode">
          <span class="option-label">Dark</span>
        </label>
        <label class="theme-option">
          <input type="radio" value="system" v-model="selectedThemeMode">
          <span class="option-label">System</span>
        </label>
      </div>
    </div>

    <div class="section">
      <h3>Time Format</h3>
      <div class="theme-options">
        <label class="theme-option">
          <input type="radio" value="12h" v-model="selectedTimeFormat">
          <span class="option-label">12h</span>
        </label>
        <label class="theme-option">
          <input type="radio" value="24h" v-model="selectedTimeFormat">
          <span class="option-label">24h</span>
        </label>
      </div>
    </div>

    <div class="section">
      <h3>Accent Color</h3>
      <div class="color-grid">
        <button 
          v-for="color in colors" 
          :key="color" 
          :style="{ backgroundColor: color }"
          class="color-btn"
          :class="{ active: primaryColor === color }"
          @click="selectColor(color)"
        ></button>
        <input type="color" v-model="primaryColor" class="custom-color-picker">
      </div>
    </div>
  </div>
  <div class="overlay" :class="{ open: isOpen }" @click="$emit('close')"></div>
</template>

<style scoped>
.settings-drawer {
  position: fixed;
  top: 0;
  right: -350px;
  width: 320px;
  height: 100vh;
  background: var(--surface-color);
  color: var(--on-surface);
  box-shadow: -2px 0 10px rgba(0,0,0,0.1);
  transition: right 0.3s ease;
  z-index: 100;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.settings-drawer.open {
  right: 0;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0,0,0,0.5);
  transition: opacity 0.3s ease, visibility 0.3s;
  opacity: 0;
  visibility: hidden;
  z-index: 90;
  backdrop-filter: blur(2px);
}

.overlay.open {
  opacity: 1;
  visibility: visible;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.close-btn {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: var(--on-surface);
}

.section {
  margin-bottom: 2rem;
}

.color-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
}

.color-btn {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid transparent;
  cursor: pointer;
  transition: transform 0.2s;
}

.color-btn:hover {
  transform: scale(1.1);
}

.color-btn.active {
  border-color: var(--on-surface);
}

.theme-options {
  display: flex;
  gap: 15px;
  background: var(--surface-color);
  padding: 10px;
  border-radius: 12px;
}

.theme-option {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.theme-option input {
  margin-right: 8px;
}

.custom-color-picker {
  width: 40px;
  height: 40px;
  padding: 0;
  border: none;
  background: none;
  cursor: pointer;
}
</style>
