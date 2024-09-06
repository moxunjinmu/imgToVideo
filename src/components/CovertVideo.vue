<template>
  <div>
    <button @click="convertImagesToVideo">合成视频</button>
  </div>
</template>
 
<script setup>
import { ref } from 'vue';
import { FFmpeg } from '@ffmpeg/ffmpeg';
import html2canvas from 'html2canvas';
 import {list} from '../mock/imagesList.js'
const ffmpeg = new FFmpeg()
const imageUrls = list.map(item => item.image);
const images = ref(imageUrls); // 图片列表
 
async function convertImagesToVideo() {
  console.log('convertImagesToVideo start');
  
  // 加载ffmpeg.wasm模块
  await ffmpeg.load({
    coreURL: '/static/esm/ffmpeg-core.js'
  });
  console.log('ffmpeg', ffmpeg);
  
  for (const image of images.value) {
    console.log('image', image);
    
    // 将图片转换为canvas
    const canvas = await html2canvas(document.querySelector(`img[src='${image}']`));
    console.log('canvas', canvas);
    
    // 将canvas图像转换为ffmpeg可接受的格式
    ffmpeg.FS('writeFile', 'image.png', await fetchFile(canvas.toDataURL('image/png')));
 
    // 使用ffmpeg命令将图片转换为视频帧
    ffmpeg.run('-i', 'image.png', '-c:v', 'libvpx-vp9', '-b:v', '0', '-pixel_format', 'yuva420p', 'frame%d.webm');
 
    // 重命名输出文件
    ffmpeg.FS('rename', 'frame1.webm', 'frame.webm');
  }
 
  // 将所有帧合成为最终的视频文件
  ffmpeg.run('-f', 'image2', '-i', 'frame%d.webm', '-c:v', 'libvpx-vp9', 'output.webm');
 
  // 获取输出视频文件
  const data = ffmpeg.FS('readFile', 'output.webm');
  
  
  console.log('可以将data转换为Blob进行下载或者其他操作', data);
  // 可以将data转换为Blob进行下载或者其他操作
  const blob = new Blob([data.buffer], { type: 'video/webm' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'output.webm';
  a.click();
 
  // 清理文件系统
  ffmpeg.FS('unlink', 'output.webm');
  for (let i = 0; i < images.value.length; i++) {
    ffmpeg.FS('unlink', `frame${i}.webm`);
  }
}
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

</script>