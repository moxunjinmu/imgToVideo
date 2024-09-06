<template>
  <div>
    <button @click="convertImagesToVideo">合成视频</button>
  </div>
</template>

<script setup>
import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg';
import html2canvas from 'html2canvas';
import {list} from '../mock/imagesList.js'
const convertImagesToVideo = async () => {
  const ffmpeg = createFFmpeg({ log: true });
  await ffmpeg.load();

  // 假设你有一个包含图片的数组
  const imageUrls = list.map(item => item.image);

  // 载入所有图片
  const images = await Promise.all(imageUrls.map(url => fetchFile(url)));

  // 将所有图片加入ffmpeg队列
  images.forEach((image, index) => {
    ffmpeg.addInput(image.arrayBuffer());
    ffmpeg.addInput(image.arrayBuffer()); // 重复输入以设置帧率
    ffmpeg.setInputFrameRate(index, 2); // 设置帧率为每秒2帧
  });

  // 设置视频参数
  ffmpeg.setOutputOptions([
    '-c:v libx264', // 使用H.264编码
    '-pix_fmt yuv420p', // 设置像素格式
    '-r 30', // 输出帧率
    '-s 640x360', // 分辨率
    '-crf 20', // 压缩质量
    'output.mp4' // 输出文件
  ]);

  // 执行转换
  try {
    await ffmpeg.run();
    console.log('视频合成成功');
    // 下载生成的视频文件
    const a = document.createElement('a');
    a.href = ffmpeg.FS('readFile', 'output.mp4');
    a.download = 'output.mp4';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  } catch (e) {
    console.error('转换失败: ', e);
  }
}


onMounted(() => {
  // convertImagesToVideo()
})
</script>
