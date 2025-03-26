<template>
  <div
    ref="line"
    class="vertical-line"
    :style="{ left: position + 'px' }"
    @mousedown="startDrag"
  >
    <!-- 可编辑的宽度指示器 -->
    <div 
      v-if="!isEditing"
      class="width-indicator"
      @click="startEditing"
    >
      {{ position }}px
    </div>
    <input
      v-else
      ref="widthInput"
      v-model.number="inputValue"
      type="number"
      class="width-input"
      @blur="stopEditing"
      @keyup.enter="stopEditing"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  // 初始位置（像素）
  initialPosition: {
    type: Number,
    default: window.innerWidth / 2
  },
  // 线条颜色
  color: {
    type: String,
    default: '#ff0000'
  },
  // 线条宽度（像素）
  width: {
    type: Number,
    default: 2
  },
  minWidth: {
    type: Number,
    default: 10
  },
  maxWidth: {
    type: Number,
    default: 2000
  }
});

const emit = defineEmits(['position-change']);

const position = ref(props.initialPosition);
const line = ref(null);
const isDragging = ref(false);const isEditing = ref(false);
const inputValue = ref(props.initialPosition);
const widthInput = ref(null);

// 开始拖动
const startDrag = (e) => {
  // 如果正在编辑，不触发拖动
  if (isEditing.value) return;

  isDragging.value = true;
  e.preventDefault(); // 防止文本选中
  document.addEventListener('mousemove', handleDrag);
  document.addEventListener('mouseup', stopDrag);
};

// 处理拖动
const handleDrag = (e) => {
  if (!isDragging.value) return;
  
  // 限制在屏幕范围内
  const newPosition = Math.max(0, Math.min(e.clientX, window.innerWidth));
  updatePosition(newPosition);
};

// 停止拖动
const stopDrag = () => {
  isDragging.value = false;
  document.removeEventListener('mousemove', handleDrag);
  document.removeEventListener('mouseup', stopDrag);
};

const startEditing = (e) => {
  e.stopPropagation(); // 防止触发拖动
  isEditing.value = true;
  inputValue.value = position.value;
  nextTick(() => {
    widthInput.value.focus();
    widthInput.value.select();
  });
};

const stopEditing = () => {
  isEditing.value = false;
  const newWidth = Math.max(props.minWidth, Math.min(inputValue.value, props.maxWidth));
  updatePosition(newWidth);
};


const updatePosition = (newPosition) => {
  position.value = newPosition;
  emit('position-change', newPosition);
};

// 响应窗口大小变化
const handleResize = () => {
  position.value = Math.min(position.value, window.innerWidth);
};

onMounted(() => {
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  stopDrag();
});
</script>

<style scoped>
.vertical-line {
  position: fixed;
  top: 0;
  bottom: 0;
  width: v-bind(props.width + 'px');
  background-color: v-bind(props.color);
  cursor: ew-resize;
  z-index: 9999;
  user-select: none;
  transition: background-color 0.2s;
}

.vertical-line:hover {
  background-color: v-bind('props.color + "cc"'); /* 添加透明度 */
}
/* 宽度指示器样式 */
.width-indicator {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  cursor: text;
  z-index: 10000;
}

/* 宽度输入框样式 */
.width-input {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  border: 1px solid #666;
  border-radius: 4px;
  padding: 2px 6px;
  font-size: 12px;
  text-align: center;
  z-index: 10000;
}

/* 移除输入框的数字选择器 */
.width-input::-webkit-outer-spin-button,
.width-input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.width-input[type=number] {
  -moz-appearance: textfield;
}
</style>