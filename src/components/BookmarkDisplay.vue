<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  bookmarks: {
    type: Array,
    required: true
  },
  folders: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['edit-bookmark', 'delete-bookmark'])

const expandedFolders = ref(new Set())

const groupedBookmarks = computed(() => {
  const grouped = {}
  
  // 初始化文件夹分组
  props.folders.forEach(folder => {
    grouped[folder.id] = {
      folder,
      bookmarks: []
    }
  })
  
  // 分配书签到对应文件夹
  props.bookmarks.forEach(bookmark => {
    if (bookmark.folder && grouped[bookmark.folder]) {
      grouped[bookmark.folder].bookmarks.push(bookmark)
    }
  })
  
  return grouped
})

const ungroupedBookmarks = computed(() => {
  return props.bookmarks.filter(bookmark => !bookmark.folder)
})

const toggleFolder = (folderId) => {
  if (expandedFolders.value.has(folderId)) {
    expandedFolders.value.delete(folderId)
  } else {
    expandedFolders.value.add(folderId)
  }
}

const goToUrl = (url) => {
  if (url.startsWith('http')) {
    window.open(url, '_blank')
  } else {
    window.open('https://' + url, '_blank')
  }
}

const isFolderExpanded = (folderId) => {
  return expandedFolders.value.has(folderId)
}
</script>

<template>
  <div class="bookmark-display">
    <div class="display-container">
      <!-- Ungrouped bookmarks -->
      <div v-for="bookmark in ungroupedBookmarks" :key="bookmark.id" class="bookmark-item">
        <div class="container">
          <button 
                  class="button" 
                  @click="goToUrl(bookmark.url)"
                  :title="bookmark.url"
                >
                  <p>{{ bookmark.name }}</p>
                </button>
        </div>
        <div class="item-actions">
          <button @click="emit('edit-bookmark', bookmark)" class="action-btn edit-btn">✎</button>
          <button @click="emit('delete-bookmark', bookmark.id)" class="action-btn delete-btn">×</button>
        </div>
      </div>

      <!-- Folders and grouped bookmarks -->
      <div v-for="(group, folderId) in groupedBookmarks" :key="folderId" class="folder-group">
        <div class="folder-item">
          <div class="container">
            <button 
              class="button" 
              @click="toggleFolder(folderId)"
            >
              <p>{{ group.folder.name }}</p>
            </button>
          </div>
          <div class="item-actions">
            <button @click="emit('delete-bookmark', group.folder.id)" class="action-btn delete-btn">×</button>
          </div>
        </div>

        <!-- Expanded bookmarks in folder - popup upwards -->
        <div v-if="isFolderExpanded(folderId)" class="folder-bookmarks-popup">
          <div v-for="bookmark in group.bookmarks" :key="bookmark.id" class="bookmark-popup-item">
            <div class="container">
              <button 
                  class="button" 
                  @click="goToUrl(bookmark.url)"
                  :title="bookmark.url"
                >
                  <p>{{ bookmark.name }}</p>
                </button>
            </div>
            <div class="item-actions">
              <button @click="emit('edit-bookmark', bookmark)" class="action-btn edit-btn">✎</button>
              <button @click="emit('delete-bookmark', bookmark.id)" class="action-btn delete-btn">×</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bookmark-display {
  position: fixed;
  bottom: 0px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  align-items: flex-end;
  padding-bottom: 5px;
  z-index: 1000;
  pointer-events: none;
}

.display-container {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
  align-items: flex-end;
  padding: 20px;
  pointer-events: auto;
}

.bookmark-item, .folder-item {
  position: relative;
  display: inline-block;
}

.folder-group {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.item-actions {
  position: absolute;
  top: -10px;
  right: -10px;
  display: flex;
  gap: 5px;
  opacity: 0;
  transition: opacity 0.3s;
}

.bookmark-item:hover .item-actions,
.folder-item:hover .item-actions,
.bookmark-popup-item:hover .item-actions {
  opacity: 1;
}

.action-btn {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: none;
  font-size: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.edit-btn {
  background: #2196F3;
  color: white;
}

.delete-btn {
  background: #f44336;
  color: white;
}

.action-btn:hover {
  transform: scale(1.2);
}

/* Button styles from user specification */
button {
  font-size: 1.1em;
  padding: 0.1em 0.2em;
  border-radius: 0.2em;
  border: none;
  background-color: #000;
  color: #fff;
  cursor: pointer;
  box-shadow: 2px 2px 3px #000000b4;
}

.container {
  position: relative;
  padding: 3px;
  background: linear-gradient(90deg, #03a9f4, #f441a5);
  border-radius: 0.8em;
  transition: all 0.4s ease;
}

.container::before {
  content: "";
  position: absolute;
  inset: 0;
  margin: auto;
  border-radius: 0.7em;
  z-index: -10;
  filter: blur(0);
  transition: filter 0.4s ease;
}

.container:hover::before {
  background: linear-gradient(90deg, #03a9f4, #f441a5);
  filter: blur(1em);
}
.container:active::before {
  filter: blur(0.15em);
}

.button {
  font-size: 1.1em;
  padding: 0em 0.5em;
  border-radius: 0.5em;
  border: none;
  background-color: #000;
  color: #fff;
  cursor: pointer;
  box-shadow: 2px 2px 3px #000000b4;
}

/* Popup styles for upward expansion */
.folder-bookmarks-popup {
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 10px;
  display: flex;
  flex-direction: column-reverse;
  gap: 10px;
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.bookmark-popup-item {
  position: relative;
  display: flex;
  justify-content: center;
}

.bookmark-popup-item .item-actions {
  position: absolute;
  top: -10px;
  right: -10px;
  display: flex;
  gap: 5px;
  opacity: 0;
  transition: opacity 0.3s;
  z-index: 20;
}
</style>