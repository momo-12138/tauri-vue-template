<script setup lang="ts">
import TitleBar from './components/TitleBar.vue';
import ToastContainer from './components/Toast.vue';
</script>

<template>
  <TitleBar />

  <ToastContainer />

  <div class="main-container">
    <div id="app-workspace">
      
    </div>
  </div>
</template>

<style>
/* 核心样式与皮肤引擎 */
@import './assets/css/styles.css';
</style>