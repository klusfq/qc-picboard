<template>
  <div class="thumbnail-container" :class="{ 'collapsed': isCollapsed }">
    <div class="toggle-button" @click="toggleCollapse">
      {{ isCollapsed ? '展开' : '折叠' }}
      <span class="badge" v-if="thumbnails.length">{{ thumbnails.length }}</span>
    </div>
    
    <Transition name="fade">
      <div class="thumbnail-list" v-show="!isCollapsed">
        <div 
          v-for="(item, index) in thumbnails" 
          :key="index" 
          class="thumbnail-item"
          :class="{ 'active': activeIndex === index }"
          @click="handleThumbnailClick(index)"
        >
          <img :src="item.image" :alt="item.title" class="thumbnail-image">
          <div class="thumbnail-title">{{ item.title }}</div>
          <button class="close-btn" @click.stop="removeThumbnail(index)" v-if="removable">
            ×
          </button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const props = defineProps({
  thumbnails: {
    type: Array,
    required: true,
    default: () => [],
    validator: (value) => {
      return value.every(item => item.image && item.title);
    }
  },
  activeIndex: {
    type: Number,
    default: 0
  },
  removable: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['thumbnail-click', 'thumbnail-remove']);

const isCollapsed = ref(false);

const toggleCollapse = () => {
  isCollapsed.value = !isCollapsed.value;
};

const handleThumbnailClick = (index) => {
  emit('thumbnail-click', index);
};

const removeThumbnail = (index) => {
  emit('thumbnail-remove', index);
};
</script>

<style scoped>
.thumbnail-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  background-color: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 12px;
  transition: all 0.3s ease;
  max-height: 80vh;
  overflow-y: auto;
  backdrop-filter: blur(5px);
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.thumbnail-container.collapsed {
  padding: 6px;
  background-color: rgba(255, 255, 255, 0.85);
}

.toggle-button {
  position: absolute;
  top: -28px;
  right: 0;
  background-color: var(--el-color-primary);
  color: white;
  padding: 4px 12px;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  user-select: none;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: all 0.2s;
}

.toggle-button:hover {
  background-color: var(--el-color-primary-light-3);
  transform: translateY(-2px);
}

.badge {
  background-color: white;
  color: var(--el-color-primary);
  border-radius: 50%;
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  font-weight: bold;
}

.thumbnail-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  width: 120px;
}

.thumbnail-item {
  position: relative;
  width: 100%;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.thumbnail-item:hover {
  transform: scale(1.03);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.thumbnail-item.active {
  border: 2px solid var(--el-color-primary);
}

.thumbnail-image {
  width: 100%;
  height: 72px;
  object-fit: cover;
  display: block;
}

.thumbnail-title {
  font-size: 12px;
  padding: 6px;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  background-color: rgba(0, 0, 0, 0.03);
}

.close-btn {
  position: absolute;
  top: 4px;
  right: 4px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 12px;
  padding: 0;
  opacity: 0;
  transition: opacity 0.2s;
}

.thumbnail-item:hover .close-btn {
  opacity: 1;
}

.close-btn:hover {
  background-color: rgba(255, 0, 0, 0.8);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>