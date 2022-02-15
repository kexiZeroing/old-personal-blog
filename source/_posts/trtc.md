# 音视频直播开发学习

## 基本概念
Tencent Real-Time Communication，TRTC 帮助开发者快速搭建低成本、低延时、高品质的音视频互动解决方案，主打低延时互动直播和多人音视频两大解决方案。

场景：视频通话场景（视频聊天、视频会议、在线问诊），语音通话场景（语音聊天、语音会议），视频互动直播（互动课堂、远程培训），语音互动直播（语音直播连麦、语聊房、K 歌房）

### 互动直播 
- 互动直播服务是一种多路音视频实时互动的解决方案。能够实现主播与观众的连麦互动，并且将这个互动的实况直播出去，让更多的用户观看。由于互动直播服务需要超低的延迟才能保证互动的效果，所以互动直播服务的延迟是毫秒级别的，而普通云直播是秒级别的。
- 旁路直播（关键词：云端混流，CDN 直播观看）指的是将低延时连麦房间里的多路推流画面复制出来，在云端将画面混合成一路，并将混流后的画面推流给直播 CDN 进行分发播放。
- TRTC 采用旁路推流的方式为您提供全程的云端录制功能，并将录制下来的文件存储到云点播平台，保证录制过程的可靠性和实时性。

### 导播（云导播台）
直播视频可由摄像机、手机、无人机等信号收集渠道接入，也可以直接导入媒体库的视频文件，传输至云端整合输出，云导播台提供了视频切换，画面组合，台标 logo、人工智能实时字幕、动画效果融合等功能，为网络直播效果提供了更丰富的场景实现。

- 画中画设置：自由配置组合画面，满足多画面同屏方案
- 模拟调音台：自由选择声道输出
- 图片叠加：可配置台标、logo、互动二维码等
- 字幕叠加：可配置文字、比分、字幕条等
- 特效包括：支持 HTML 场景渲染；支持拖拽方式叠加多层字幕、台标、时钟以及滚动字幕；支持在直播信号上使用图片特效、文字特效、时间特效等；提供硬切及软切操作，支持淡入淡出及多种类型滑像；支持整个场景保存、载入，以及场景特效元素的保存载入

### 推拉流
推送指用户将本地的音视频数据上传给 TRTC 服务端的操作，对应“推流”
订阅指用户向 TRTC 服务端请求拉取指定用户音视频数据的操作，对应“拉流”

- 拉流直播即拉取一个流地址，然后转到自己平台的直播流，进行直播和控制
- 推送到第三方直播平台

### 云端混流转码
云端混流先将多路音视频流进行解码，包括视频解码和音频解码 -> 将多路画面混合在一起，并根据来自 SDK 的混流指令实现具体的排版方案，同时需要将解码后的多路音频信号进行混音处理 -> 将混合后的画面和声音进行二次编码，并封装成一路音视频流，交给下游的直播和录制服务。

使用客户端 TRTC SDK 的 `setMixTranscodingConfig` 接口发起混流指令（纯音频模式/预排版模式/屏幕分享模式），预排版模式下，SDK 会自动按照您预先设定各路画面的排版规则将房间里的多路音频流混合成一路。

云端混流的本质是将多路流混合到当前（即发起混流指令的）用户所对应的音视频流上，因此当前用户本身必须有音视频上行才能构成混流的前提条件。

## WebRTC samples
浏览器支持播放视频、播放音频、分享屏幕等的原理，具体看 https://webrtc.github.io/samples/

```js
// Display the video stream
const constraints = {
  audio: false,
  video: true
};
const stream = await navigator.mediaDevices.getUserMedia(constraints);
handleSuccess(stream);

function handleSuccess(stream) {
  const video = document.querySelector('video');
  const videoTracks = stream.getVideoTracks();
  console.log(`Using video device: ${videoTracks[0].label}`);
  video.srcObject = stream;
}
```

```js
// Draw a frame from the video onto the canvas (take snapshot)
const video = document.querySelector('video');
const canvas = document.querySelector('canvas');
const button = document.querySelector('button');
button.onclick = function() {
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  canvas.getContext('2d').drawImage(video, 0, 0, canvas.width, canvas.height);
};
```

```js
// Choose camera resolution
let stream;

vgaButton.onclick = () => {
  getMedia(vgaConstraints);
};
hdButton.onclick = () => {
  getMedia(hdConstraints);
};

const vgaConstraints = {
  video: {width: {exact: 640}, height: {exact: 480}}
};
const hdConstraints = {
  video: {width: {exact: 1280}, height: {exact: 720}}
};

function getMedia(constraints) {
  if (stream) {
    stream.getTracks().forEach(track => {
      track.stop();
    });
  }
  navigator.mediaDevices.getUserMedia(constraints).then(gotStream)
}

function gotStream(mediaStream) {
  stream = mediaStream;
  video.srcObject = mediaStream;
}
```

```js
// Render the audio stream 
const audio = document.querySelector('audio');
const constraints = {
  audio: true,
  video: false
};

function handleSuccess(stream) {
  const audioTracks = stream.getAudioTracks();
  console.log('Using audio device: ' + audioTracks[0].label);
  stream.oninactive = function() {
    console.log('Stream ended');
  };
  audio.srcObject = stream;
}

navigator.mediaDevices.getUserMedia(constraints).then(handleSuccess)
```

```js
// Display the screensharing stream
function handleSuccess(stream) {
  startButton.disabled = true;
  const video = document.querySelector('video');
  video.srcObject = stream;

  // detect that the user has stopped sharing the screen
  stream.getVideoTracks()[0].addEventListener('ended', () => {
    console.log('The user has ended sharing the screen');
    startButton.disabled = false;
  });
}

const startButton = document.getElementById('startButton');
startButton.addEventListener('click', () => {
  navigator.mediaDevices.getDisplayMedia({video: true}).then(handleSuccess)
});
```

## TRTC Electron demo
https://github.com/tencentyun/TRTCSDK/tree/master/Electron
https://cloud.tencent.com/document/product/647/38548

- 直播模式 **TRTCAppSceneLIVE** 下的 TRTC，支持单个房间最多 10 万人同时在线，具备小于 300ms 的连麦延迟和小于 1000ms 的观看延迟，以及平滑上下麦切换技术。该模式下有角色的概念，分为主播和观众
- 通话模式 **TRTCAppSceneVideoCall** TRTC 房间中的所有用户都相当于是主播，每个用户随时都可以发言，最高的上行并发限制为 50 路，适合在线会议等场景，单个房间的人数限制为 300 人。该模式下无角色的概念
