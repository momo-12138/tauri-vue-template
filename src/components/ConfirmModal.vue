<script setup lang="ts">
defineProps({
  visible: { type: Boolean, default: false },
  title: { type: String, default: '提示' },
  message: { type: String, default: '' },
  confirmText: { type: String, default: '确认' },
  cancelText: { type: String, default: '取消' },
  isDanger: { type: Boolean, default: false } // 如果是危险操作，确认按钮显示红色
});

const emit = defineEmits(['update:visible', 'confirm', 'cancel']);

const close = () => {
  emit('update:visible', false);
  emit('cancel');
};

const confirm = () => {
  emit('update:visible', false);
  emit('confirm');
};
</script>

<template>
  <Transition name="mac-modal">
    <div v-if="visible" class="modal-overlay" @click.self="close">
      <div class="modal-box">
        <div class="modal-content">
          <h3 class="modal-title">{{ title }}</h3>
          <p class="modal-message">{{ message }}</p>
        </div>
        
        <div class="modal-actions">
          <button class="action-btn cancel-btn" @click="close">{{ cancelText }}</button>
          <button class="action-btn confirm-btn" :class="{ 'is-danger': isDanger }" @click="confirm">{{ confirmText }}</button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
/* 遮罩层渐变动画 */
.mac-modal-enter-active, .mac-modal-leave-active {
  transition: opacity 0.2s ease;
}
.mac-modal-enter-from, .mac-modal-leave-to {
  opacity: 0;
}

/* Mac 原生窗口微小缩放与下落感 */
.mac-modal-enter-active .modal-box {
  animation: mac-slide-in 0.25s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
.mac-modal-leave-active .modal-box {
  transition: transform 0.15s ease, opacity 0.15s ease;
  transform: scale(0.96);
  opacity: 0;
}

@keyframes mac-slide-in {
  0% { transform: scale(0.96) translateY(-10px); opacity: 0; }
  100% { transform: scale(1) translateY(0); opacity: 1; }
}

.modal-overlay {
  position: fixed;
  top: 0; left: 0; width: 100vw; height: 100vh;
  background: rgba(0, 0, 0, 0.25);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

/* ============================================================================
   🧱 macOS 动态宽度引擎 (核心修正点)
   ============================================================================ */
.modal-box {
  width: max-content;        /* 🌟 核心：根据内容自适应宽度 */
  min-width: 280px;          /* 🌟 限制最小宽度，防止字太少时弹窗缩得太小 */
  max-width: 420px;          /* 🌟 限制最大宽度，防止长文本无限拉伸弹窗 */
  background-color: var(--bg-modal); 
  box-shadow: var(--shadow-modal); 
  color: var(--text-main); 
  border: 1px solid var(--border-main);
  transition: background-color 0.3s, border-color 0.3s, box-shadow 0.3s;
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 文本排版区 */
.modal-content {
  padding: 24px 24px 16px;   /* 稍微加宽左右内边距，留出视觉呼吸感 */
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.modal-title {
  margin: 0 0 8px 0;
  font-size: 14.5px;
  font-weight: 600;
  color: var(--text-main);
  width: 100%;
}

.modal-message {
  margin: 0;
  font-size: 12.5px;
  line-height: 1.5;
  color: var(--text-secondary);
  white-space: pre-line;
  width: 100%;
  
  /* 🌟 核心：如果文件名实在太长，在超出 max-width 时自动切断并优雅地显示出省略号 */
  word-break: break-all;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* ============================================================================
   🔘 macOS 独立按钮组（锁死单行，拒绝换行）
   ============================================================================ */
.modal-actions {
  display: flex;
  justify-content: flex-end;
  flex-wrap: nowrap;         /* 🌟 锁死单行：绝对不允许按钮被挤下去换行 */
  align-items: center;
  gap: 8px;
  padding: 0 24px 16px;      /* 与内容区边距对齐 */
  width: 100%;
}

.action-btn {
  min-width: 76px;           /* Mac 标准按钮最小宽度 */
  padding: 5px 14px;
  border-radius: 6px;
  font-size: 12.5px;
  cursor: pointer;
  border: none;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  white-space: nowrap;       /* 🌟 按钮文字绝对不换行 */
  transition: background-color 0.1s, box-shadow 0.1s;
}

/* 1️⃣ 取消按钮样式 */
.cancel-btn {
  background: rgba(0, 0, 0, 0.04);
  color: var(--text-main);
  border: 0.5px solid rgba(0, 0, 0, 0.08);
}
.cancel-btn:hover { background: rgba(0, 0, 0, 0.08); }
.cancel-btn:active { background: rgba(0, 0, 0, 0.12); }

:global(.dark) .cancel-btn {
  background: rgba(255, 255, 255, 0.07);
  border: 0.5px solid rgba(255, 255, 255, 0.08);
}
:global(.dark) .cancel-btn:hover { background: rgba(255, 255, 255, 0.11); }

/* 2️⃣ 确认按钮样式 */
.confirm-btn {
  background: #007aff;
  color: #ffffff;
  font-weight: 500;
  box-shadow: 0 1px 2px rgba(0, 122, 255, 0.15);
}
.confirm-btn:hover { background: #006ee5; }
.confirm-btn:active { background: #0062cc; }

:global(.dark) .confirm-btn { background: #147efb; }
:global(.dark) .confirm-btn:hover { background: #006ee5; }

/* 3️⃣ 危险操作按钮（你的截图对应的删除红） */
.confirm-btn.is-danger {
  background: #ff3b30;
  box-shadow: 0 1px 2px rgba(255, 59, 48, 0.15);
}
.confirm-btn.is-danger:hover { background: #e03228; }
.confirm-btn.is-danger:active { background: #c72c23; }
</style>