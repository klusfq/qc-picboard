<script setup>
import { ref, computed, onMounted } from "vue";
import { listen } from "@tauri-apps/api/event";
import { readImage, writeImage } from "@tauri-apps/plugin-clipboard-manager";
import ThumbnailList from "./Thumbnail.vue";
import CryptoJS from "crypto-js";
import VerticalLine from "./YLine.vue";
import { nanoid } from "nanoid";
import BtnExport from "./BtnExport.vue";

const linePosition = ref(360);
const thumbnailImage = ref({
  Index: 0,
  List: [],
  Hset: new Set(),
});

onMounted(() => {
  handlePasteImg();
});

listen("tauri://focus", (event) => {
  handlePasteImg();
});

listen("tauri://blur", (event) => {
  console.log("blur event", event);
});

const exportAsSingleImage = async ({updateProgress, showMessage, complete}) => {
  // 1. 创建一个canvas元素
  const canvas = document.createElement('canvas');
  const container = document.querySelector('.img-list');
  
  // 2. 设置canvas尺寸为容器大小
  canvas.width = container.offsetWidth;
  canvas.height = container.offsetHeight;
  const ctx = canvas.getContext('2d');
  
  // 3. 绘制背景（可选）
  ctx.fillStyle = '#ffffff'; // 白色背景
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  
  // 4. 绘制所有图片
  const images = document.querySelectorAll('.resizable-image');

  if (images.length === 0) {
    showMessage('请先粘贴图片');
    complete();
    return;
  }

  let yOffset = 0;
  
  for (const img of images) {
    await new Promise((resolve) => {
      const imgEl = new Image();
      imgEl.crossOrigin = 'Anonymous';
      imgEl.src = img.src;
      imgEl.onload = () => {
        ctx.drawImage(
          imgEl,
          img.offsetLeft,
          yOffset,
          img.offsetWidth,
          img.offsetHeight
        );
        yOffset += img.offsetHeight; // 0px间距
        resolve();
      };
    });
  }
  
  // 5. 导出为图片
  const dataURL = canvas.toDataURL('image/png');
  const link = document.createElement('a');
  link.download = 'qcmerge-image-' + nanoid(6) + '.png';
  link.href = dataURL;
  link.click();

  showMessage('导出成功');
  complete();
};

const handlePasteImg = () => {
  readImage().then((image) => {
    image.size().then((size) => {
      image.rgba().then((x) => {
        const wordArray = CryptoJS.lib.WordArray.create(x);
        const picKey = CryptoJS.MD5(wordArray).toString();
        if (thumbnailImage.value.Hset.has(picKey)) {
          return;
        }
        thumbnailImage.value.Hset.add(picKey);

        // 需要将RGBA数据转换为标准图像格式
        const canvas = document.createElement('canvas');
        canvas.width = size.width;
        canvas.height = size.height;
        const ctx = canvas.getContext('2d');
        const imageData = new ImageData(new Uint8ClampedArray(x), size.width);
        ctx.putImageData(imageData, 0, 0);

        canvas.toBlob((blob) => {
          const url = URL.createObjectURL(blob);
          thumbnailImage.value.List.push({
            image: url,
            title: picKey,
          });
        }, 'image/png');
      });
    });
  });
}

const handleThumbnailClick = (index) => {
  console.log(thumbnailImage.value);
  thumbnailImage.value.Index = index;
};

const handleThumbnailRemove = (index) => {
  const url = thumbnailImage.value.List[index].image;
  URL.revokeObjectURL(url);
  thumbnailImage.value.Hset.delete(thumbnailImage.value.List[index].title);

  thumbnailImage.value.List.splice(index, 1);
  if (thumbnailImage.value.Index >= index && thumbnailImage.value.Index > 0) {
    thumbnailImage.value.Index--;
  }
};

// 计算图片宽度（竖线位置就是图片宽度）
const imageWidth = computed(() => linePosition.value);
// 处理竖线位置变化
const handleLinePositionChange = (newPosition) => {
  linePosition.value = newPosition;
};
</script>

<template>
  <main class="container">
    <VerticalLine
      :initial-position="linePosition"
      color="#0096FF"
      :width="3"
      @position-change="handleLinePositionChange"
    />

    <div class="img-list" :style="{ width: imageWidth + 'px'}">
      <div v-for="(img, index) in thumbnailImage?.List" :key="index">
        <img
          :src="img.image"
          :alt="img.title"
          :style="{ width: imageWidth + 'px'}"
          class="resizable-image"
        />
      </div>
    </div>

    <ThumbnailList :thumbnails="thumbnailImage.List" :active-index="thumbnailImage.Index" :removable="true"
      @thumbnail-click="handleThumbnailClick" @thumbnail-remove="handleThumbnailRemove" />

    <BtnExport @click="exportAsSingleImage"></BtnExport>
  </main>
</template>

<style scoped>
.container {
  position: fixed; /* 或 absolute */
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: auto; /* 根据需要设置 */
}
.resizable-image {
  display: block;
  height: auto;
  max-width: 100%;
  transition: width 0.2s ease; /* 添加平滑过渡效果 */
}
</style>
