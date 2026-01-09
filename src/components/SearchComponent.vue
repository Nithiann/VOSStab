<script setup>
import { ref } from 'vue';

const query = ref('');
const engines = [
  { name: 'Google', url: 'https://www.google.com/search?q=', icon: 'G' },
  { name: 'DuckDuckGo', url: 'https://duckduckgo.com/?q=', icon: 'D' },
  { name: 'Bing', url: 'https://www.bing.com/search?q=', icon: 'B' },
  { name: 'Ecosia', url: 'https://www.ecosia.org/search?q=', icon: 'E' }
];
const selectedEngine = ref(engines[0]);
const isDropdownOpen = ref(false);

const performSearch = () => {
  const q = query.value.trim();
  if (!q) return;

  // Regex for explicit URL (starts with http/https) or looks like domain (no spaces, has dot)
  const isUrl = /^(https?:\/\/)/i.test(q) || (/^[^ ]+\.[^ ]+$/.test(q) && !q.includes(' '));

  if (isUrl) {
    let url = q;
    if (!/^https?:\/\//i.test(url)) {
      url = `https://${url}`;
    }
    window.location.href = url;
  } else {
    window.location.href = `${selectedEngine.value.url}${encodeURIComponent(q)}`;
  }
};

const selectEngine = (engine) => {
  selectedEngine.value = engine;
  isDropdownOpen.value = false;
};
</script>

<template>
  <div class="search-container">
    <div class="glass-bar">
      <div class="engine-selector" @click="isDropdownOpen = !isDropdownOpen" v-click-outside="() => isDropdownOpen = false">
        <span class="engine-icon">{{ selectedEngine.icon }}</span>
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="arrow-icon">
          <path d="M7.41 8.59L12 13.17l4.59-4.58L18 10l-6 6-6-6 1.41-1.41z"/>
        </svg>
        
        <div v-if="isDropdownOpen" class="dropdown">
          <div 
            v-for="engine in engines" 
            :key="engine.name" 
            class="dropdown-item"
            @click.stop="selectEngine(engine)"
          >
            {{ engine.name }}
          </div>
        </div>
      </div>
      
      <input 
        type="text" 
        v-model="query" 
        @keydown.enter="performSearch"
        placeholder="Search..."
        class="search-input"
        autofocus
      />
      
      <button class="search-btn" @click="performSearch">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="search-icon">
          <path d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
.search-container {
  position: fixed;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 800px;
  max-width: 95vw;
  margin: 0;
  padding: 0 20px;
  z-index: 50;
}

.glass-bar {
  display: flex;
  align-items: center;
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid var(--glass-border);
  box-shadow: var(--glass-shadow);
  border-radius: 24px;
  padding: 8px 16px;
  transition: box-shadow 0.3s, transform 0.3s;
}

.glass-bar:focus-within {
  box-shadow: 0 12px 40px rgba(0,0,0,0.2);
  transform: translateY(-2px);
  border-color: var(--primary-color);
}

.engine-selector {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 8px;
  position: relative;
  user-select: none;
}

.engine-icon {
  font-weight: bold;
  margin-right: 4px;
  color: var(--primary-color);
}

.arrow-icon {
  width: 16px;
  height: 16px;
  opacity: 0.6;
}

.dropdown {
  position: absolute;
  top: 120%;
  left: 0;
  background: var(--surface-color);
  color: var(--on-surface);
  border-radius: 12px;
  box-shadow: 0 8px 20px rgba(0,0,0,0.2);
  overflow: hidden;
  z-index: 10;
  min-width: 140px;
}

.dropdown-item {
  padding: 10px 16px;
  cursor: pointer;
  transition: background 0.2s;
}

.dropdown-item:hover {
  background: var(--primary-color-alpha);
}

.search-input {
  flex: 1;
  background: none;
  border: none;
  padding: 12px;
  font-size: 1.1rem;
  color: var(--on-background);
  outline: none;
}

.search-input::placeholder {
  color: var(--on-background);
  opacity: 0.5;
}

.search-btn {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--primary-color);
  padding: 8px;
  border-radius: 50%;
  transition: background 0.2s;
}

.search-btn:hover {
  background: var(--primary-color-alpha);
}

.search-icon {
  width: 24px;
  height: 24px;
}
</style>
