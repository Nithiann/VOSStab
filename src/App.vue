<script setup>
import { ref, onMounted } from 'vue';
import SettingsDrawer from './components/SettingsDrawer.vue';
import SearchComponent from './components/SearchComponent.vue';
import BookmarksComponent from './components/BookmarksComponent.vue';
import TimeDisplay from './components/TimeDisplay.vue';
import TaskManager from './components/TaskManager.vue';

const isSettingsOpen = ref(false);
const isTaskManagerOpen = ref(false);
const showTaskManager = ref(true);
const currentTimeFormat = ref('24h');

const applyTheme = (theme) => {
  if (theme === 'dark') {
    document.body.classList.add('dark-mode');
  } else {
    document.body.classList.remove('dark-mode');
  }
};

const updateTheme = (mode) => {
  if (mode === 'system') {
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    applyTheme(prefersDark ? 'dark' : 'light');
  } else {
    applyTheme(mode);
  }
};

const hexToRgb = (hex) => {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? {
    r: parseInt(result[1], 16),
    g: parseInt(result[2], 16),
    b: parseInt(result[3], 16)
  } : null;
};

const updateColor = (color) => {
  document.body.style.setProperty('--primary-color', color);
  document.body.style.setProperty('--primary-color-alpha', `${color}33`);
  // Add some tonal variations
  document.body.style.setProperty('--primary-container', `${color}1A`); // 10% opacity
  
  const rgb = hexToRgb(color);
  if (rgb) {
    const rgbString = `${rgb.r}, ${rgb.g}, ${rgb.b}`;
    document.body.style.setProperty('--primary-rgb', rgbString);
    document.body.style.setProperty('--glass-shadow', `0 8px 32px 0 rgba(${rgbString}, 0.37)`);
  }
};

// System theme listener
const onSystemThemeChange = (e) => {
  const currentMode = localStorage.getItem('theme-mode');
  if (currentMode === 'system') {
    applyTheme(e.matches ? 'dark' : 'light');
  }
};

onMounted(() => {
  const savedColor = localStorage.getItem('primary-color');
  const savedThemeMode = localStorage.getItem('theme-mode') || 'system';
  const savedTimeFormat = localStorage.getItem('time-format') || '24h';
  
  if (savedColor) updateColor(savedColor);
  updateTheme(savedThemeMode);
  currentTimeFormat.value = savedTimeFormat;
  
  const savedShowTasks = localStorage.getItem('show-task-manager');
  if (savedShowTasks !== null) showTaskManager.value = savedShowTasks === 'true';
  
  window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', onSystemThemeChange);
});
</script>

<template>
  <div class="app-container">
    <button class="settings-toggle" @click="isSettingsOpen = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="icon">
        <path d="M12 15.5A3.5 3.5 0 0 1 8.5 12 3.5 3.5 0 0 1 12 8.5a3.5 3.5 0 0 1 3.5 3.5 3.5 3.5 0 0 1-3.5 3.5m7.43-2.53c.04-.32.07-.64.07-.97 0-.33-.03-.66-.07-1l2.11-1.63c.19-.15.24-.42.12-.64l-2-3.46c-.12-.22-.39-.3-.61-.22l-2.49 1c-.52-.4-1.08-.73-1.69-.98l-.38-2.65C14.46 2.18 14.25 2 14 2h-4c-.25 0-.46.18-.5.42l-.38 2.65c-.61.25-1.17.59-1.69.98l-2.49-1c-.23-.09-.49 0-.61.22l-2 3.46c-.13.22-.07.49.12.64l2.11 1.63c-.04.34-.07.67-.07 1 0 .33.03.66.07 1l-2.11 1.63c-.19.15-.24.42-.12.64l2 3.46c.12.22.39.3.61.22l2.49-1c.52.4 1.08.73 1.69.98l.38 2.65c.04.24.25.42.5.42h4c.25 0 .46-.18.5-.42l.38-2.65c.61-.25 1.17-.59 1.69-.98l2.49 1c.23.09.49 0 .61-.22l2-3.46c.12-.22.07-.49-.12-.64l-2.11-1.63z"/>
      </svg>
    </button>

    <SettingsDrawer 
      :isOpen="isSettingsOpen" 
      @close="isSettingsOpen = false"
      @update:theme="updateTheme"
      @update:color="updateColor"
      @update:timeFormat="format => currentTimeFormat = format"
      @update:showTaskManager="val => showTaskManager = val"
    />

    <button v-if="showTaskManager" class="tasks-toggle" @click="isTaskManagerOpen = !isTaskManagerOpen" :class="{ active: isTaskManagerOpen }">
      <i class="fas fa-tasks"></i>
    </button>

    <TaskManager :isOpen="isTaskManagerOpen" @close="isTaskManagerOpen = false" />

    <main class="main-content">
      <TimeDisplay :timeFormat="currentTimeFormat" />
      <SearchComponent />
      <BookmarksComponent />
    </main>
    
    <h1 class="clock-display">VOSStab</h1>
  </div>
</template>

<style scoped>
.app-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  /* Fixed elements don't need flex centering */
  background: 
    radial-gradient(at 0% 0%, var(--primary-color-alpha) 0px, transparent 50%),
    radial-gradient(at 100% 0%, var(--primary-color-alpha) 0px, transparent 50%),
    radial-gradient(at 100% 100%, var(--primary-color-alpha) 0px, transparent 50%),
    radial-gradient(at 0% 100%, var(--primary-color-alpha) 0px, transparent 50%);
  background-size: 200% 200%;
  animation: gradientMoved 15s ease infinite;
}

@keyframes gradientMoved {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.clock-display {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.5rem;
  color: var(--on-background);
  opacity: 0.8;
  margin: 0;
  pointer-events: none;
  font-weight: 300;
  letter-spacing: 2px;
}

.settings-toggle {
  position: absolute;
  top: 20px;
  right: 20px;
  background: none;
  border: none;
  color: var(--on-background);
  cursor: pointer;
  opacity: 0.5;
  transition: opacity 0.3s;
}

.settings-toggle:hover {
  opacity: 1;
}

.icon {
  width: 24px;
  height: 24px;
}

.tasks-toggle {
  position: absolute;
  top: 20px;
  left: 20px;
  background: none;
  border: none;
  color: var(--on-background);
  font-size: 1.2rem;
  cursor: pointer;
  opacity: 0.5;
  transition: all 0.3s;
  z-index: 10;
}

.tasks-toggle:hover, .tasks-toggle.active {
  opacity: 1;
  color: var(--primary-color);
}
</style>
