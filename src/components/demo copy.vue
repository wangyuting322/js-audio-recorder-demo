<template>
  <div class="demo">
    <button @click="getPermission">获取权限</button>
    <button @click="start">开始</button>
    <button @click="pause">暂停</button>
    <button @click="resume">继续</button>
    <button @click="stop">结束</button>
    <button @click="getData">获取音频数据</button>
    <button @click="play">播放</button>
    <button @click="stopPlay">停止播放</button>
    <div>录音时长：{{ recorderInfo.duration.toFixed(1) }}秒</div>
    <div style="width:100%;height:200px;border:1px solid red;">
      <canvas id="canvas"></canvas>
      <span style="padding: 0 10%;"></span>
      <canvas id="playChart"></canvas>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import Recorder from 'js-audio-recorder';
// 录音实例
let recorder = ref<Recorder | null>()
// 录音的详情
let recorderInfo = reactive({
  duration: 0,// 获取录音的总时长（单位：秒）
  fileSize: 0// 录音文件大小（单位：字节）
})
//波浪图-录音
let drawRecordId = ref<object, null>(null)
let oCanvas = ref<HTMLCanvasElement, null>(null)
let ctx = ref<object, null>(null)

// 获取页面录音权限
function getPermission() {
  return Recorder.getPermission().then(() => {
    console.log('给权限了');
  }, (error) => {
    alert(`${error.name} : ${error.message}`)
    console.log(`${error.name} : ${error.message}`)
  })
}
onMounted(() => {
  startCanvas();
})
/**
 * 波浪图配置
 * */
function startCanvas() {
  //录音波浪
  oCanvas.value = document.getElementById('canvas');
  ctx.value = oCanvas.value.getContext("2d");
}
/**
      * 绘制波浪图-录音
      * */
function drawRecord() {
  // 用requestAnimationFrame稳定60fps绘制
  drawRecordId.value = requestAnimationFrame(drawRecord);
  // 实时获取音频大小数据
  let dataArray = recorder.value.getRecordAnalyseData(),
    bufferLength = dataArray.length;
  // 填充背景色
  ctx.value.fillStyle = 'rgb(200, 200, 200)';
  ctx.value.fillRect(0, 0, oCanvas.value.width, oCanvas.value.height);
  // 设定波形绘制颜色
  ctx.value.lineWidth = 2;
  ctx.value.strokeStyle = 'rgb(0, 0, 0)';
  ctx.value.beginPath();
  let sliceWidth = oCanvas.value.width * 1.0 / bufferLength, // 一个点占多少位置，共有bufferLength个点要绘制
    x = 0;          // 绘制点的x轴位置
  for (let i = 0; i < bufferLength; i++) {
    let v = dataArray[i] / 128.0;
    let y = v * oCanvas.value.height / 2;
    if (i === 0) {
      // 第一个点
      ctx.value.moveTo(x, y);
    } else {
      // 剩余的点
      ctx.value.lineTo(x, y);
    }
    // 依次平移，绘制所有点
    x += sliceWidth;
  }
  ctx.value.lineTo(oCanvas.value.width, oCanvas.value.height / 2);
  ctx.value.stroke();
}
async function start() {
  // 销毁上一个
  await destroy()
  // 重新开启一个实例
  recorder.value = new Recorder({
    sampleBits: 16,                 // 采样位数，支持 8 或 16，默认是16
    sampleRate: 16000,              // 采样率，支持 11025、16000、22050、24000、44100、48000，根据浏览器默认值，我的chrome是48000
    numChannels: 1,                 // 声道，支持 1 或 2， 默认是1
  });
  recorderInfo.duration = 0
  recorderInfo.fileSize = 0
  // 开始录音
  recorder.value.start().then(() => {
    drawRecord();//开始绘制图片
    // 录音过程监听
    if (recorder.value) {
      recorder.value.onprogress = (params) => {
        // console.log(params);       // 当前获取到到音频数据
        recorderInfo.duration = params.duration
        recorderInfo.fileSize = params.fileSize
        let dataArray = recorder.value.getRecordAnalyseData();
        console.log(dataArray);

      }
    }
  }, (error) => {
    alert(`${error.name} : ${error.message}`)
    // 出错了
    console.log(`${error.name} : ${error.message}`);
  });
}
function pause() {
  recorder.value?.pause()
}
function resume() {
  recorder.value?.resume()
}
function stop() {
  recorder.value?.stop()
}
function destroy() {
  return recorder.value?.destroy().then(() => {
    recorder.value = null;
    drawRecordId.value && cancelAnimationFrame(drawRecordId.value);
    drawRecordId.value = null;
  });
}
function getData() {
  let res = recorder.value?.getPCMBlob();
  console.log(res);

}
function play() {
  recorder.value?.play();
}
function stopPlay() {
  recorder.value?.pausePlay();
}


</script>
<style scoped>
.demo {}
</style>