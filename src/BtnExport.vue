<template>
  <div class="floating-export-container" :class="positionClass">
    <button
      class="floating-export-btn"
      :class="{ 'is-loading': isLoading }"
      :disabled="isLoading"
      @click="handleClick"
      :title="tooltip"
    >
      <span v-if="!isLoading" class="btn-content">
        <slot name="icon">
          <span class="icon">↓</span>
        </slot>
        <span class="text" v-if="showText">
          <slot>{{ buttonText }}</slot>
        </span>
      </span>
      <span v-else class="loading-content">
        <span class="loader"></span>
        <span class="progress" v-if="showProgress && progress > 0">
          {{ progress }}%
        </span>
      </span>
    </button>

    <transition name="fade">
      <div class="export-tooltip" v-if="showTooltip && tooltipMessage">
        {{ tooltipMessage }}
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
  buttonText: {
    type: String,
    default: '导出'
  },
  tooltip: {
    type: String,
    default: '点击导出内容'
  },
  showText: {
    type: Boolean,
    default: true
  },
  showProgress: {
    type: Boolean,
    default: true
  },
  position: {
    type: String,
    default: 'bottom-right',
    validator: (value) => ['bottom-right', 'bottom-left', 'top-right', 'top-left'].includes(value)
  },
  autoHideTooltip: {
    type: Boolean,
    default: true
  },
  tooltipDuration: {
    type: Number,
    default: 3000
  }
});

const emit = defineEmits(['click']);

const isLoading = ref(false);
const progress = ref(0);
const showTooltip = ref(false);
const tooltipMessage = ref('');

const positionClass = computed(() => `position-${props.position.replace('-', '_')}`);

let tooltipTimeout = null;

const showMessage = (message) => {
  tooltipMessage.value = message;
  showTooltip.value = true;
  
  if (props.autoHideTooltip) {
    clearTimeout(tooltipTimeout);
    tooltipTimeout = setTimeout(() => {
      showTooltip.value = false;
    }, props.tooltipDuration);
  }
};

const handleClick = async () => {
  try {
    isLoading.value = true;
    progress.value = 0;
    showMessage('正在准备导出...');
    
    // 触发父组件事件
    emit('click', {
      updateProgress: (value) => {
        progress.value = Math.min(100, Math.max(0, value));
        showMessage(`导出中... ${progress.value}%`);
      },
      showMessage,
      complete: () => {
        isLoading.value = false;
      }
    });
  } catch (error) {
    showMessage(`导出失败: ${error.message}`);
    isLoading.value = false;
  }
};

watch(isLoading, (newVal) => {
  if (!newVal) {
    progress.value = 0;
  }
});
</script>

<style scoped>
.floating-export-container {
  position: fixed;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

/* 位置类 */
.position-bottom_right {
  right: 20px;
  bottom: 20px;
}

.position-bottom_left {
  left: 20px;
  bottom: 20px;
}

.position-top_right {
  right: 20px;
  top: 20px;
}

.position-top_left {
  left: 20px;
  top: 20px;
}

/* 按钮基础样式 */
.floating-export-btn {
  width: 60px;
  height: 60px;
  background-color: #1890ff;
  color: white;
  border: none;
  border-radius: 50%;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  outline: none;
}

.floating-export-btn:hover:not(:disabled) {
  background-color: #40a9ff;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.25);
}

.floating-export-btn:disabled {
  background-color: #bae7ff;
  cursor: not-allowed;
}

.floating-export-btn.is-loading {
  background-color: #096dd9;
}

/* 按钮内容 */
.btn-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.icon {
  font-size: 20px;
  line-height: 1;
}

.text {
  font-size: 12px;
  margin-top: 4px;
}

/* 加载状态 */
.loading-content {
  display: flex;
  align-items: center;
  justify-content: center;
}

.loader {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.progress {
  margin-left: 8px;
  font-size: 12px;
}

/* 提示框 */
.export-tooltip {
  margin-top: 10px;
  padding: 8px 12px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  border-radius: 4px;
  font-size: 14px;
  white-space: nowrap;
}

/* 过渡动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .floating-export-btn {
    width: 50px;
    height: 50px;
  }
  
  .text {
    display: none;
  }
  
  .icon {
    font-size: 18px;
  }
  
  .export-tooltip {
    font-size: 12px;
    max-width: 150px;
    text-align: center;
    white-space: normal;
  }
}
</style>