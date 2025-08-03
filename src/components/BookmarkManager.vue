<script setup>
import { ref, onMounted } from 'vue'
import BookmarkDisplay from './BookmarkDisplay.vue'

const bookmarks = ref([])
const folders = ref([])
const showEditModal = ref(false)
const showAddModal = ref(false)
const editingBookmark = ref(null)
const newBookmark = ref({ name: '', url: '', folder: null })
const newFolder = ref({ name: '' })

// 从localStorage加载书签和文件夹
const loadBookmarks = () => {
  const savedBookmarks = localStorage.getItem('bookmarks')
  const savedFolders = localStorage.getItem('bookmarkFolders')
  
  if (savedBookmarks) {
    bookmarks.value = JSON.parse(savedBookmarks)
  }
  
  if (savedFolders) {
    folders.value = JSON.parse(savedFolders)
  }
}

// 保存到localStorage
const saveBookmarks = () => {
  localStorage.setItem('bookmarks', JSON.stringify(bookmarks.value))
  localStorage.setItem('bookmarkFolders', JSON.stringify(folders.value))
}

// Add bookmark
const addBookmark = () => {
  // Handle new folder creation
  if (newFolder.value.name) {
    // Check if folder with same name already exists
    const existingFolder = folders.value.find(f => 
      f.name.toLowerCase() === newFolder.value.name.toLowerCase()
    )
    
    if (existingFolder) {
      alert('Folder with this name already exists!')
      return
    }

    const folderId = Date.now()
    folders.value.push({
      id: folderId,
      name: newFolder.value.name
    })
    newBookmark.value.folder = folderId
  }

  if (newBookmark.value.name && newBookmark.value.url) {
    bookmarks.value.push({
      id: Date.now(),
      name: newBookmark.value.name,
      url: newBookmark.value.url,
      folder: newBookmark.value.folder
    })
    saveBookmarks()
    newBookmark.value = { name: '', url: '', folder: null }
    newFolder.value = { name: '' }
    showAddModal.value = false
  }
}

// 添加文件夹
const addFolder = () => {
  if (newFolder.value.name) {
    folders.value.push({
      id: Date.now(),
      name: newFolder.value.name
    })
    saveBookmarks()
    newFolder.value = { name: '' }
  }
}

// Delete bookmark or folder
const deleteBookmark = (id) => {
  // Check if this is a folder ID (positive number)
  const isFolder = folders.value.some(f => f.id === id)
  
  if (isFolder) {
    // Handle folder deletion
    folders.value = folders.value.filter(f => f.id !== id)
    // Also delete bookmarks in this folder
    bookmarks.value = bookmarks.value.filter(b => b.folder !== id)
  } else {
    // Handle bookmark deletion
    bookmarks.value = bookmarks.value.filter(b => b.id !== id)
  }
  saveBookmarks()
}

// 编辑书签
const editBookmark = (bookmark) => {
  editingBookmark.value = { ...bookmark }
  showEditModal.value = true
}

// 保存编辑
const saveEdit = () => {
  if (editingBookmark.value) {
    const index = bookmarks.value.findIndex(b => b.id === editingBookmark.value.id)
    if (index !== -1) {
      bookmarks.value[index] = { ...editingBookmark.value }
      saveBookmarks()
    }
    showEditModal.value = false
    editingBookmark.value = null
  }
}

// 跳转到书签
const goToBookmark = (url) => {
  if (url.startsWith('http')) {
    window.open(url, '_blank')
  } else {
    window.open('https://' + url, '_blank')
  }
}

// 初始化
onMounted(() => {
  loadBookmarks()
})
</script>

<template>
  <div class="bookmark-manager">
    <!-- Bookmark Display -->
    <BookmarkDisplay 
      :bookmarks="bookmarks" 
      :folders="folders"
      @edit-bookmark="editBookmark"
      @delete-bookmark="deleteBookmark"
    />

    <!-- Floating Edit Button -->
    <div class="bookmark-trigger">
      <button class="edit-button" @click="showAddModal = true">
        <svg class="edit-svgIcon" viewBox="0 0 512 512">
          <path d="M410.3 231l11.3-11.3-33.9-33.9-62.1-62.1L291.7 89.8l-11.3 11.3-22.6 22.6L58.6 322.9c-10.4 10.4-18 23.3-22.2 37.4L1 480.7c-2.5 8.4-.2 17.5 6.1 23.7s15.3 8.5 23.7 6.1l120.3-35.4c14.1-4.2 27-11.8 37.4-22.2L387.7 253.7 410.3 231zM160 399.4l-9.1 22.7c-4 3.1-8.5 5.4-13.3 6.9L59.4 452l23-78.1c1.4-4.9 3.8-9.4 6.9-13.3l22.7-9.1v32c0 8.8 7.2 16 16 16h32zM362.7 18.7L348.3 33.2 325.7 55.8 314.3 67.1l33.9 33.9 62.1 62.1 33.9 33.9 11.3-11.3 22.6-22.6 14.5-14.5c25-25 25-65.5 0-90.5L453.3 18.7c-25-25-65.5-25-90.5 0zm-47.4 168l-144 144c-6.2 6.2-16.4 6.2-22.6 0s-6.2-16.4 0-22.6l144-144c6.2-6.2 16.4-6.2 22.6 0s6.2 16.4 0 22.6z"></path>
        </svg>
      </button>
    </div>

    <!-- Add Bookmark Modal -->
    <div class="modal" v-if="showAddModal">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Add Bookmark</h3>
          <button class="close-btn" @click="showAddModal = false">×</button>
        </div>
        <div class="modal-body">
          <input v-model="newBookmark.name" placeholder="Bookmark Name" class="input-field">
          <input v-model="newBookmark.url" placeholder="URL" class="input-field">
          <select v-model="newBookmark.folder" class="input-field">
            <option :value="null">No Folder</option>
            <option v-for="folder in folders" :key="folder.id" :value="folder.id">
              {{ folder.name }}
            </option>
          </select>
          <input v-model="newFolder.name" placeholder="New Folder Name (optional)" class="input-field">
        </div>
        <div class="modal-footer">
          <button @click="addBookmark" class="submit-btn">Add</button>
          <button @click="showAddModal = false" class="cancel-btn">Cancel</button>
        </div>
      </div>
    </div>

    <!-- Edit Bookmark Modal -->
    <div class="modal" v-if="showEditModal">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Edit Bookmark</h3>
          <button class="close-btn" @click="showEditModal = false">×</button>
        </div>
        <div class="modal-body">
          <input v-model="editingBookmark.name" placeholder="Bookmark Name" class="input-field">
          <input v-model="editingBookmark.url" placeholder="URL" class="input-field">
          <select v-model="editingBookmark.folder" class="input-field">
            <option :value="null">No Folder</option>
            <option v-for="folder in folders" :key="folder.id" :value="folder.id">
              {{ folder.name }}
            </option>
          </select>
        </div>
        <div class="modal-footer">
          <button @click="saveEdit" class="submit-btn">Save</button>
          <button @click="showEditModal = false" class="cancel-btn">Cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bookmark-manager {
  position: fixed;
  bottom: 30px;
  right: 30px;
  z-index: 1000;
}

.bookmark-trigger {
  position: relative;
}

.edit-button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: rgb(20, 20, 20);
  border: none;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.164);
  cursor: pointer;
  transition-duration: 0.3s;
  overflow: hidden;
  position: relative;
  text-decoration: none !important;
}

.edit-svgIcon {
  width: 17px;
  transition-duration: 0.3s;
}

.edit-svgIcon path {
  fill: white;
}

.edit-button:hover {
  width: 120px;
  border-radius: 50px;
  transition-duration: 0.3s;
  background-color: rgb(211, 211, 211);
  align-items: center;
}

.edit-button:hover .edit-svgIcon {
  width: 20px;
  transition-duration: 0.3s;
  transform: translateY(60%);
  -webkit-transform: rotate(360deg);
  -moz-transform: rotate(360deg);
  -o-transform: rotate(360deg);
  -ms-transform: rotate(360deg);
  transform: rotate(360deg);
}

.edit-button::before {
  display: none;
  content: "New Bookmark";
  color: rgb(50, 50, 50);
  transition-duration: 0.3s;
  font-size: 2px;
}

.edit-button:hover::before {
  display: block;
  padding-right: 10px;
  font-size: 13px;
  opacity: 1;
  transform: translateY(0px);
  transition-duration: 0.3s;
}

.bookmark-panel {
  position: absolute;
  bottom: 60px;
  right: 0;
  width: 300px;
  max-height: 400px;
  background: white;
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow-y: auto;
  padding: 20px;
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.panel-header h3 {
  margin: 0;
  color: #333;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: #666;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.section-header h4 {
  margin: 0;
  color: #555;
  font-size: 14px;
}

.add-btn, .delete-btn, .edit-small-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  padding: 2px 6px;
  border-radius: 3px;
  transition: background 0.2s;
}

.add-btn:hover {
  background: #4CAF50;
  color: white;
}

.delete-btn:hover {
  background: #f44336;
  color: white;
}

.edit-small-btn:hover {
  background: #2196F3;
  color: white;
}

.folders-list, .bookmarks-list {
  margin-bottom: 15px;
}

.folder-item, .bookmark-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px;
  margin: 5px 0;
  background: #f5f5f5;
  border-radius: 8px;
  font-size: 14px;
}

.bookmark-btn {
  flex: 1;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  color: #2196F3;
  text-decoration: underline;
}

.bookmark-btn:hover {
  color: #1976D2;
}

.input-field {
  width: 100%;
  padding: 8px;
  margin: 5px 0;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
}

.folder-input {
  width: 100%;
  padding: 8px;
  margin: 5px 0;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
}

.submit-btn, .cancel-btn {
  padding: 8px 16px;
  margin: 5px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
}

.submit-btn {
  background: #2196F3;
  color: white;
}

.cancel-btn {
  background: #f44336;
  color: white;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 15px;
  width: 300px;
  max-width: 90%;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.modal-header h3 {
  margin: 0;
  color: #333;
}

.modal-body {
  margin-bottom: 15px;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
</style>