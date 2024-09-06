<template>
  <div>
    <div class="video-wrapper">
      <video :src="videoSrc" :autoplay="false" :controls="true" class="i-video"></video>
    </div>
    <div>{{ messageText }}</div>
    <button type="button" @click="handleTranscode" style="margin-right: 42px;">转码</button>
    <input type="file" accept="video/*" @change="handleFileInputChange" @click="handleFileInputClick" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { FFmpeg } from '@ffmpeg/ffmpeg';


const ffmpeg = new FFmpeg();

const videoSrc = ref('');
const messageText = ref('');

let isLoadFfmpegCore = false;

let isTranscoding = false;

ffmpeg.on('log', (e) => {
  console.log(e.message);
});


// methods
const handleTranscode = async () => {
  if (!isLoadFfmpegCore) {
    messageText.value = '加载ffmpeg-core.js';
    await ffmpeg.load({
      coreURL: '/static/esm/ffmpeg-core.js',
      // wasmURL: '/static/esm/ffmpeg-core.wasm'
    });
    isLoadFfmpegCore = true;
  }
  if (isTranscoding) return;
  isTranscoding = true;
  messageText.value = '开始转码';

  await ffmpeg.writeFile('test.avi', await fetchFile('/static/shopDetail.avi'));
  await ffmpeg.exec(['-i', 'test.avi', 'test.mp4']);
  messageText.value = '转码完成';
  isTranscoding = false;
  const data = await ffmpeg.readFile('test.mp4');
  videoSrc.value = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
}

const handleFileInputChange = async (e) => {
  const file = e.target.files[0];
  const name = file.name.replace(/\.[0-9a-zA-Z]+$/, '');
  if (!isLoadFfmpegCore) {
    messageText.value = '加载ffmpeg-core.js';
    await ffmpeg.load({
      coreURL: '/static/esm/ffmpeg-core.js',
      // wasmURL: '/static/esm/ffmpeg-core.wasm'
    });
    isLoadFfmpegCore = true;
  }
  if (isTranscoding) return;
  isTranscoding = true;
  messageText.value = '开始上传的文件转码';
  await ffmpeg.writeFile(file.name, await fetchFile(file));
  await ffmpeg.exec(['-i', file.name, `${name}.mp4`]);
  messageText.value = '转码完成';
  isTranscoding = false;
  const data = await ffmpeg.readFile(`${name}.mp4`);
  videoSrc.value = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
}

const readFromBlobOrFile = (blob) => (
  new Promise((resolve, reject) => {
    const fileReader = new FileReader();
    fileReader.onload = () => {
      resolve(fileReader.result);
    };
    fileReader.onerror = ({ target: { error: { code } } }) => {
      reject(Error(`File could not be read! Code=${code}`));
    };
    fileReader.readAsArrayBuffer(blob);
  })
);

const fetchFile = async (_data) => {
  let data = _data;
  if (typeof _data === 'undefined') {
    return new Uint8Array();
  }

  if (typeof _data === 'string') {
    /* From base64 format */
    if (/data:_data\/([a-zA-Z]*);base64,([^"]*)/.test(_data)) {
      data = atob(_data.split(',')[1])
        .split('')
        .map((c) => c.charCodeAt(0));
    /* From remote server/URL */
    } else {
      const res = await fetch(new URL(_data, import.meta.url).href);
      data = await res.arrayBuffer();
    }
  /* From Blob or File */
  } else if (_data instanceof File || _data instanceof Blob) {
    data = await readFromBlobOrFile(_data);
  }

  return new Uint8Array(data);
};

const handleFileInputClick = (e) => {
  e.target.value = '';
}

</script>

<style scoped>
.video-wrapper {
  width: 1400px;
  height: 788px;
}
.i-video {
  width: 100%;
  height: 100%;
}
</style>