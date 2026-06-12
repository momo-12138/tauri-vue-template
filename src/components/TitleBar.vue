<script setup lang="ts">
import { onMounted, ref } from 'vue';

const isDark = ref(false);
// 给 appWindow 赋予正确的类型提示（如果未安装相关类型，可以暂时保留 any）
const appWindow = ref<any>(null);

onMounted(async () => {
  initThemeEngine();
  
  try {
    // 🌟 核心修正：从 @tauri-apps/api/window 引入 getCurrentWindow
    const { getCurrentWindow } = await import('@tauri-apps/api/window');
    appWindow.value = getCurrentWindow();
  } catch (e) {
    console.warn("当前未运行在 Tauri 环境（普通浏览器模式）");
  }
});

// Windows/Mac 窗口交互指令
const handleClose = () => appWindow.value?.close();
const handleMinimize = () => appWindow.value?.minimize();
const handleMaximize = async () => {
  if (!appWindow.value) return;
  
  // 检查是否已最大化
  if (await appWindow.value.isMaximized()) {
    appWindow.value.unmaximize();
  } else {
    appWindow.value.maximize();
  }
};

// 跨端通用深浅色主题切换引擎
const initThemeEngine = () => {
  const savedTheme = localStorage.getItem('theme');
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  isDark.value = savedTheme === 'dark' || (!savedTheme && prefersDark);
  document.documentElement.classList.toggle('dark', isDark.value);
};

const toggleTheme = () => {
  isDark.value = !isDark.value;
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light');
  document.documentElement.classList.toggle('dark', isDark.value);
};
</script>

<template>
  <div class="header">
    <div class="mac-controls">
      <button @click="handleClose" class="mac-btn close"></button>
      <button @click="handleMinimize" class="mac-btn minimize"></button>
      <button @click="handleMaximize" class="mac-btn maximize"></button>
    </div>
    
    <div data-tauri-drag-region class="drag-bar"></div>
    
    <div class="right-tools">
      <button @click="toggleTheme" class="tool-btn" title="切换主题">
        <svg v-if="!isDark" class="icon-moon" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
        </svg>
        <svg v-else class="icon-sun" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="5"></circle>
          <line x1="12" y1="1" x2="12" y2="3"></line>
          <line x1="12" y1="21" x2="12" y2="23"></line>
          <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
          <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
          <line x1="1" y1="12" x2="3" y2="12"></line>
          <line x1="21" y1="12" x2="23" y2="12"></line>
          <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
          <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
        </svg>
      </button>
    </div>
  </div>

</template>