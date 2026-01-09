<script setup>
import { ref, onMounted } from 'vue';

const bookmarks = ref([]);
const currentFolder = ref(null);
const breadcrumbs = ref([]);
const error = ref(null);
const isDevMode = ref(false);

const fetchBookmarks = async () => {
  try {
    const processTree = (tree) => {
      const root = tree[0];
      // Find toolbar by common IDs or titles
      const toolbar = root.children.find(node => 
        node.id === '1' || 
        node.id === 'toolbar_____' || 
        node.title === 'Bookmarks Bar' || 
        node.title === 'Bookmarks Toolbar'
      );
       
      if (toolbar) {
        bookmarks.value = toolbar.children;
      } else {
        // Fallback to first child's contents (usually the bar)
        bookmarks.value = root.children[0]?.children || [];
      }
    };

    // Firefox uses 'browser' namespace and Promises
    if (typeof browser !== 'undefined' && browser.bookmarks) {
      const tree = await browser.bookmarks.getTree();
      processTree(tree);
    } 
    // Chrome uses 'chrome' namespace and Callbacks (or Promises in MV3)
    else if (typeof chrome !== 'undefined' && chrome.bookmarks) {
      chrome.bookmarks.getTree((tree) => {
        processTree(tree);
      });
    } else {
      // Dev mode or no permissions
      isDevMode.value = true;
      error.value = "Extensions API not available (Dev Mode)";
      // Only show empty state or dev message, NOT random bookmarks
    }
  } catch (e) {
    console.error("Failed to fetch bookmarks:", e);
    error.value = "Failed to load bookmarks. Permission denied?";
  }
};

const openFolder = (folder) => {
  breadcrumbs.value.push(currentFolder.value);
  currentFolder.value = folder;
};

const goBack = () => {
  const prev = breadcrumbs.value.pop();
  currentFolder.value = prev || null;
};

const itemsToShow = () => {
  if (currentFolder.value) {
    return currentFolder.value.children;
  }
  return bookmarks.value;
};

onMounted(() => {
  fetchBookmarks();
});
</script>

<template>
  <div class="bookmarks-container">
    <div v-if="error || isDevMode" class="message-box">
      <p v-if="isDevMode">Running in Dev Mode. Install as extension to view real bookmarks.</p>
      <p v-if="error && !isDevMode">{{ error }}</p>
    </div>

    <div class="header">
      <div v-if="currentFolder" class="header-content">
        <button class="back-btn" @click="goBack">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="icon">
            <path d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z"/>
          </svg>
        </button>
        <span>{{ currentFolder.title }}</span>
      </div>
      <!-- Empty state spacer if needed, but min-height in CSS is better -->
      <div v-else class="header-spacer"></div>
    </div>

    <div class="grid">
      <div 
        v-for="item in itemsToShow()" 
        :key="item.id" 
        class="bookmark-item"
        @click="item.url ? null : openFolder(item)"
      >
        <a v-if="item.url" :href="item.url" class="bookmark-link">
          <div class="icon-placeholder file-icon">
            <img :src="`https://www.google.com/s2/favicons?domain=${item.url}&sz=32`" alt="" />
          </div>
          <span class="title">{{ item.title }}</span>
        </a>
        
        <div v-else class="folder-item">
          <div class="icon-placeholder folder-icon">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
              <path d="M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z"/>
            </svg>
          </div>
          <span class="title">{{ item.title }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>

<style scoped>
.message-box {
  padding: 20px;
  background: var(--glass-bg);
  border-radius: 12px;
  text-align: center;
  margin-bottom: 20px;
  border: 1px dashed var(--glass-border);
}

.bookmarks-container {
  position: fixed;
  top: 48%;
  left: 50%;
  transform: translateX(-50%);
  width: 800px; /* Align with search bar */
  max-width: 95vw;
  margin: 0;
  padding: 0 20px;
  max-height: 45vh;
  overflow-y: auto;
  /* Hide scrollbar visually but allow scrolling */
  scrollbar-width: none;
  -ms-overflow-style: none; 
}

.bookmarks-container::-webkit-scrollbar {
  display: none;
}

.header {
  display: flex;
  align-items: center;
  margin-bottom: 2rem;
  font-size: 1.2rem;
  color: var(--on-background);
  width: 100%;
  padding: 0 10px;
  /* Reserve space to prevent layout jump */
  min-height: 44px; 
}

.header-content {
  display: flex;
  align-items: center;
  width: 100%;
}

.header-spacer {
  height: 100%;
  width: 100%;
}

.back-btn {
  background: none;
  border: none;
  color: var(--primary-color);
  cursor: pointer;
  margin-right: 0.8rem;
  padding: 4px;
  border-radius: 50%;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.back-btn:hover {
  background: var(--primary-color-alpha);
}

.icon {
  width: 20px;
  height: 20px;
}

.grid {
  display: grid;
  /* 6 columns to fit nicer in width */
  grid-template-columns: repeat(6, 1fr);
  gap: 16px;
  width: 100%;
}

/* Responsive adjustment for smaller screens */
@media (max-width: 850px) {
  .grid {
    grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
  }
}

.bookmark-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 10px;
  border-radius: 12px;
  transition: background 0.2s, transform 0.2s;
  cursor: pointer;
  width: 100%;
  aspect-ratio: 1; /* Keep square aspect ratio */
}

.bookmark-item:hover {
  background: var(--glass-bg);
  transform: translateY(-2px);
}

.bookmark-link, .folder-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-decoration: none;
  color: var(--on-background);
  width: 100%;
  height: 100%;
}

.icon-placeholder {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
  background: var(--surface-color);
  color: var(--primary-color);
}

.folder-icon {
  background: var(--primary-color-alpha);
}

.title {
  font-size: 0.85rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  width: 100%;
  max-width: 90px;
}

img {
  width: 24px;
  height: 24px;
}
</style>
