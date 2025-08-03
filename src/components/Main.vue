<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import ThemeToggle from './ThemeToggle.vue'
import BookmarkManager from './BookmarkManager.vue'

import bingIcon from '../assets/bing-1.svg'
import googleIcon from '../assets/google-g-2015.svg'
import duckduckgoIcon from '../assets/duckduckgo-4.svg'

const searchQuery = ref('')
const currentTime = ref('')
const greeting = ref('')
const currentEngine = ref(0)
const SAVED_ENGINE_KEY = 'savedSearchEngine'

const engines = [
  {
    name: 'Bing',
    placeholder: 'Search in Bing...',
    url: 'https://www.bing.com/search?q=',
    icon: bingIcon
  },
  {
    name: 'Google',
    placeholder: 'Search in Google...',
    url: 'https://www.google.com/search?q=',
    icon: googleIcon
  },
  {
    name: 'DuckDuckGo',
    placeholder: 'Search in DuckDuckGo...',
    url: 'https://duckduckgo.com/?q=',
    icon: duckduckgoIcon
  }
]

const updateTime = () => {
  const now = new Date()
  const hours = now.getHours()
  
  // 根据时间设置问候语
  if (hours < 12) {
    greeting.value = 'Good morning'
  } else if (hours < 18) {
    greeting.value = 'Good afternoon'
  } else {
    greeting.value = 'Good evening'
  }
  
  // 格式化时间，精确到秒，24小时制
  currentTime.value = now.toLocaleTimeString('en-GB', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  })
}

// 每秒更新时间
let timer = null
onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
  const savedEngine = localStorage.getItem(SAVED_ENGINE_KEY);
  if (savedEngine !== null) {
    currentEngine.value = parseInt(savedEngine);
  }
})

onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
  }
})

const handleSearch = () => {
  if (searchQuery.value.trim()) {
    window.location.href = engines[currentEngine.value].url + encodeURIComponent(searchQuery.value)
  }
}

const handleKeyPress = (event) => {
  if (event.key === 'Enter' && searchQuery.value.trim()) {
    window.location.href = engines[currentEngine.value].url + encodeURIComponent(searchQuery.value)
  }
}

const switchEngine = () => {
  currentEngine.value = (currentEngine.value + 1) % engines.length;
  localStorage.setItem(SAVED_ENGINE_KEY, currentEngine.value.toString());
}
</script>

<template>
  <div class="search-container">
    <div class="theme-toggle-container">
      <ThemeToggle />
    </div>
    <div class="header-info">
      <div class="greeting">{{ greeting }}</div>
      <div class="time">{{ currentTime }}</div>
    </div>
    <div class="InputContainer">
      <div class="search-wrapper">
        <button class="engine-toggle-button" @click="switchEngine" :title="`Switch to ${engines[(currentEngine + 1) % engines.length].name}`">
          <img :src="engines[currentEngine].icon" :alt="engines[currentEngine].name" class="engine-icon">
        </button>
        <input 
          :placeholder="engines[currentEngine].placeholder" 
          id="input" 
          class="input" 
          name="text" 
          type="text"
          v-model="searchQuery"
          @keypress="handleKeyPress"
        >
        <button class="search-button" @click="handleSearch">
          <span>
            <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path d="M0 0h24v24H0z" fill="none"></path>
              <path d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.41z" fill="currentColor"></path>
            </svg>
          </span>
        </button>
      </div>
    </div>
  </div>
  
  <BookmarkManager />
</template>

<style scoped>
.search-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  width: 100%;
  position: relative;
}

.theme-toggle-container {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

.header-info {
  margin-bottom: 15px;
  width: 500px;
}

.greeting {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
  text-align: left;
  margin-left: 16px;
  transition: color 0.3s ease;
}

body.dark-mode .greeting {
  color: #ffffff;
}

body.light-mode .greeting {
  color: #000000;
}

.time {
  font-size: 14px;
  font-weight: bold;
  color: #666;
  font-family: 'Courier New', monospace;
  text-align: left;
  margin-left: 16px;
  transition: color 0.3s ease;
}

body.dark-mode .time {
  color: #ffffff;
}

body.light-mode .time {
  color: #000000;
}

.InputContainer {
  width: 500px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-size: 100% 100%; 
  background-position: 0px 0px,0px 0px,0px 0px,0px 0px,0px 0px; 
  background-image: radial-gradient(49% 81% at 45% 47%, #FFE20345 0%, #073AFF00 100%),radial-gradient(113% 91% at 17% -2%, #FF5A00FF 1%, #FF000000 99%),radial-gradient(142% 91% at 83% 7%, #FFDB00FF 1%, #FF000000 99%),radial-gradient(142% 91% at -6% 74%, #FF0049FF 1%, #FF000000 99%),radial-gradient(142% 91% at 111% 84%, #FF7000FF 0%, #FF0000FF 100%);
  border-radius: 30px;
  overflow: hidden;
  cursor: pointer;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.075);
  transition: background 0.3s ease, color 0.3s ease;
}

body.dark-mode .InputContainer {
  background: #2a2a2a;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

body.light-mode .InputContainer {
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.search-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.engine-toggle-button {
  position: relative;
  width: 40px;
  height: 40px;
  margin-right: 8px;
  background: transparent;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  box-shadow: 0 0 0 rgba(254, 193, 149, 0.5);
}

.engine-toggle-button:hover {
  background: rgba(254, 193, 149, 0.1);
  box-shadow: 0 0 15px rgba(254, 193, 149, 0.5);
  transform: scale(1.1);
}

.engine-icon {
  width: 24px;
  height: 24px;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.engine-toggle-button:hover .engine-icon {
  transform: rotate(360deg);
}

.input {
  width: 380px;
  height: 40px;
  border: none;
  outline: none;
  caret-color: rgb(255, 81, 0);
  background-color: rgb(255, 255, 255);
  border-radius: 30px;
  padding: 0 45px 0 15px;
  letter-spacing: 0.8px;
  color: rgb(19, 19, 19);
  font-size: 13.4px;
  font-weight: bold;
  transition: background 0.3s ease, color 0.3s ease;
}

body.dark-mode .input {
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

body.light-mode .input {
  background: #ffffff;
  color: #000000;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

.search-button {
  position: absolute;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  font-family: inherit;
  font-weight: 500;
  font-size: 14px;
  border-radius: 50%;
  cursor: pointer;
  border: none;
  background: linear-gradient(to right, #8e2de2, #4a00e0);
  color: ghostwhite;
  overflow: hidden;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.1s ease;
}

.search-button:active {
  transform: translateY(-50%) scale(0.95);
}

.search-button span {
  position: relative;
  z-index: 10;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.search-button svg {
  width: 18px;
  height: 18px;
}

.search-button::before,
.search-button::after {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.search-button::before {
  content: "";
  background: #1a1a1a;
  width: 120%;
  left: -10%;
  transform: skew(30deg);
  transition: transform 0.4s cubic-bezier(0.3, 1, 0.8, 1);
}

.search-button:hover::before {
  transform: translate3d(100%, 0, 0);
}
</style>
