<template>
  <div>
    <h1>Image to Video Converter</h1>
    <input type="file" multiple @change="handleFilesChange" />
    <button @click="convertImagesToVideo">Convert Images to Video</button>
    <video controls ref="videoRef" width="640" height="480"></video>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const videoRef = ref(null);
const images = ref([]);
const canvas = ref(null);
const mediaRecorder = ref(null);
const recordedChunks = ref([]);

const handleFilesChange = (event) => {
  images.value = event.target.files;
};

const convertImagesToVideo = () => {
  if (images.value.length === 0) {
    alert('Please select some images.');
    return;
  }

  const canvasElement = document.createElement('canvas');
  const ctx = canvasElement.getContext('2d');

  const videoWidth = images.value[0].width;
  const videoHeight = images.value[0].height;
  canvasElement.width = videoWidth;
  canvasElement.height = videoHeight;

  const mediaStream = new MediaStream();
  // const streamTrack = mediaStream.addTrack(new OffscreenCanvasMediaStreamTrack(canvasElement));

  mediaRecorder.value = new MediaRecorder(mediaStream, {
    mimeType: 'video/webm'
  });

  mediaRecorder.value.ondataavailable = (event) => {
    if (event.data.size > 0) {
      recordedChunks.value.push(event.data);
    }
  };

  mediaRecorder.value.onstop = () => {
    const blob = new Blob(recordedChunks.value, { type: 'video/webm' });
    videoRef.value.src = URL.createObjectURL(blob);
    videoRef.value.load();
    videoRef.value.play();
  };

  mediaRecorder.value.start();

  images.value.forEach((imageFile, index) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      ctx.drawImage(e.target.result, 0, 0, videoWidth, videoHeight);
      setTimeout(() => {
        mediaRecorder.value.ondataavailable({ data: null });
        if (index === images.value.length - 1) {
          mediaRecorder.value.stop();
        }
      }, 1000); // 每张图片显示1秒
    };
    reader.readAsDataURL(imageFile);
  });
};

const test = () => {
  if ('OffscreenCanvas' in window) {
    // 支持OffscreenCanvas，可以安全使用OffscreenCanvasMediaStreamTrack
    console.log('支持OffscreenCanvas，可以安全使用OffscreenCanvasMediaStreamTrack');
    
} else {
    // 不支持OffscreenCanvas，需要考虑替代方案
    console.log('不支持OffscreenCanvas，需要考虑替代方案');
}
}

onMounted(() => {
  // 初始化
  test()
});
</script>
