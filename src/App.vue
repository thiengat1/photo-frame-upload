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

    <div
      class="frame-wrapper"
      @mousedown="startDrag"
      @mousemove="onDrag"
      @mouseup="stopDrag"
      @mouseleave="stopDrag"
      @touchstart="startDrag"
      @touchmove="onDrag"
      @touchend="stopDrag"
    >
      <img
        v-if="preview"
        :src="preview"
        class="user-image"
        :style="imageStyle"
        draggable="false"
      />

      <img src="/frame.png" class="frame-overlay" />
    </div>

    <canvas
      ref="canvasRef"
      width="600"
      height="600"
      style="display: none"
    ></canvas>

    <button class="download-btn" @click="downloadImage" :disabled="!preview">
      Tải ảnh xuống
    </button>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue';

const preview = ref('');
const canvasRef = ref(null);

const scale = ref(1);
const posX = ref(0);
const posY = ref(0);

const isDragging = ref(false);
const startX = ref(0);
const startY = ref(0);

const imageStyle = computed(() => ({
  transform: `translate(${posX.value}px, ${posY.value}px) scale(${scale.value})`,
  transformOrigin: 'top left',
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
const loadImage = (src) => {
  return new Promise((resolve, reject) => {
    const img = new Image();

    img.crossOrigin = 'anonymous';

    img.onload = () => resolve(img);
    img.onerror = reject;

    img.src = src;
  });
};

const drawCanvas = async () => {
  const canvas = canvasRef.value;

  const ctx = canvas.getContext('2d');

  ctx.clearRect(0, 0, canvas.width, canvas.height);

  const userImg = await loadImage(preview.value);
  const frameImg = await loadImage('/frame.png');

  ctx.save();

  ctx.translate(posX.value, posY.value);
  ctx.scale(scale.value, scale.value);

  ctx.drawImage(userImg, 0, 0, 600, 600);

  ctx.restore();

  ctx.drawImage(frameImg, 0, 0, 600, 600);
};
const downloadImage = async () => {
  try {
    await drawCanvas();

    const canvas = canvasRef.value;

    canvas.toBlob((blob) => {
      const url = URL.createObjectURL(blob);

      const link = document.createElement('a');

      link.href = url;
      link.download = `ky-niem-${Date.now()}.png`;

      document.body.appendChild(link);

      link.click();

      document.body.removeChild(link);

      URL.revokeObjectURL(url);
    }, 'image/png');
  } catch (error) {
    console.error(error);
    alert('Không thể tải ảnh');
  }
};
</script>
