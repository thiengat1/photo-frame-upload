<template>
  <div class="container">
    <h1>Kỷ niệm ra trường</h1>

    <input type="file" accept="image/*" @change="onUpload" />

    <div class="tools" v-if="preview">
      <label>
        Zoom
        <input type="range" min="0.5" max="3" step="0.1" v-model="scale" />
      </label>
    </div>

    <div class="frame-wrapper" ref="captureArea">
      <div
        class="image-container"
        v-if="preview"
        @mousedown="startDrag"
        @mousemove="onDrag"
        @mouseup="stopDrag"
        @mouseleave="stopDrag"
        @touchstart="startDrag"
        @touchmove="onDrag"
        @touchend="stopDrag"
      >
        <img
          :src="preview"
          class="user-image"
          :style="imageStyle"
          draggable="false"
        />
      </div>

      <img src="/frame.png" class="frame-overlay" />
    </div>

    <button class="download-btn" @click="downloadImage" :disabled="!preview">
      Tải ảnh xuống
    </button>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue';
import { toPng } from 'html-to-image';
const preview = ref('');
const captureArea = ref(null);

const scale = ref(1);
const posX = ref(0);
const posY = ref(0);

const isDragging = ref(false);
const startX = ref(0);
const startY = ref(0);

const imageStyle = computed(() => ({
  transform: `translate(${posX.value}px, ${posY.value}px) scale(${scale.value})`,
}));
const onUpload = (e) => {
  const file = e.target.files[0];

  if (!file) return;

  const reader = new FileReader();

  reader.onload = (event) => {
    preview.value = event.target.result;

    posX.value = 0;
    posY.value = 0;
    scale.value = 1;
  };

  reader.readAsDataURL(file);
};
const getPoint = (event) => {
  if (event.touches && event.touches[0]) {
    return {
      x: event.touches[0].clientX,
      y: event.touches[0].clientY,
    };
  }

  return {
    x: event.clientX,
    y: event.clientY,
  };
};

const startDrag = (event) => {
  isDragging.value = true;

  const point = getPoint(event);

  startX.value = point.x - posX.value;
  startY.value = point.y - posY.value;
};
const onDrag = (event) => {
  if (!isDragging.value) return;

  event.preventDefault();

  const point = getPoint(event);

  posX.value = point.x - startX.value;
  posY.value = point.y - startY.value;
};

const stopDrag = () => {
  isDragging.value = false;
};
const downloadImage = async () => {
  try {
    if (!captureArea.value) return;

    const node = captureArea.value;
    const width = node.clientWidth;
    const height = node.clientHeight;
    const dataUrl = await toPng(node, {
      cacheBust: true,
      pixelRatio: 2,
      backgroundColor: 'transparent',
      canvasWidth: width * 2,
      canvasHeight: height * 2,
      width,
      height,
      style: {
        margin: '0',
        padding: '0',
      },
    });

    const link = document.createElement('a');
    link.download = `ky-niem-${Date.now()}.png`;
    link.href = dataUrl;
    link.click();
  } catch (error) {
    console.error(error);
    alert('Không thể tải ảnh');
  }
};
</script>
