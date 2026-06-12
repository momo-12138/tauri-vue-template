<script lang="ts">
import { ref, defineComponent } from 'vue';

const toasts = ref<{ id: number; message: string; show: boolean }[]>([]);
let toastId = 0;

// 暴露出一个供全局调用的公共函数
export const showToast = (message: string) => {
  const id = toastId++;
  toasts.value.push({ id, message, show: false });
  
  // 动效延迟触发
  setTimeout(() => {
    const target = toasts.value.find(t => t.id === id);
    if (target) target.show = true;
  }, 10);

  // 2.5秒后淡出并移除
  setTimeout(() => {
    const target = toasts.value.find(t => t.id === id);
    if (target) target.show = false;
    setTimeout(() => {
      toasts.value = toasts.value.filter(t => t.id !== id);
    }, 300);
  }, 2500);
};

export default defineComponent({
  name: 'ToastContainer',
  setup() {
    return { toasts };
  }
});
</script>

<template>
  <div id="mac-toast-container">
    <div 
      v-for="toast in toasts" 
      :key="toast.id" 
      class="mac-toast" 
      :class="{ 'show': toast.show }"
    >
      {{ toast.message }}
    </div>
  </div>
</template>