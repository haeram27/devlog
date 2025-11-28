# HTML5에서 `.ts` (MPEG Transport Stream) 형식의 영상을 플레이

HTML5에서 `.ts` 파일은 주로 **HLS 스트리밍**의 일부로 사용되며, **hls.js 라이브러리**를 사용하는 것이 가장 일반적이고 안정적인 방법

## 참고

- [aws-vod-streaming-service](https://hanamon.kr/aws-vod-streaming-service-%ea%b5%ac%ed%98%84-%ea%b0%80%ec%9d%b4%eb%93%9c-1-%ec%84%a4%ea%b3%84/)

## HLS(HTTP Live Streaming Protocaol)

- HTTP를 이용하여 영상을 스트리밍 하기 위한 프로토콜
- HLS는 m3u8 파일(playlist)와 ts(영상데이터의 chunk)를 구성하는 스트리밍하는 방식

### m3u8

- m3u8 확장자를 가진 파일은 UTF-8로 인코딩된 재생 목록 파일
- m3u8 파일은 스트리밍 오디오 또는 비디오의 URL 경로와 미디어 트랙에 대한 정보를 저장하는 데 사용할 수 있는 일반 텍스트 파일
- m3u8은 영상 재생을 위한 메타 데이터로 대역폭별 m3u8 파일 경로, ts 파일 경로가 담김
- 대역폭마다의 설정이 담겨 있기 때문에 m3u8이 네트워크 상태에 따른 영상 화질 선택이 가능

### ts

- ts 파일은 시간마다 쪼개져 있는 실제 영상 데이터 (이렇게 조개져 있어서 부분 재생이 가능)
- ts 파일은 MPEG transport stream 포맷으로 인코딩된 청크 파일
- ts 파일은 보통 2-10초 정도로 자르고, 비디오 데이터를 포함
- ts 파일은 m3u8 파일을 통해 클라이언트에게 제공, 클라이언트는 이 파일들을 다운로드하고 병합하여 비디오를 재생

## HTTP 스트리밍 방식 두 가지

- HTML5 이전에는 video 태그가 없었기 때문에 웹에서 비디오를 재생하기 위해서는 플래시와 같은 것이 필요했음
- 점차 표준 기술인 HTML5의 video로 전환되어지고 네트워크 환경에 다른 최적의 스트리밍 서비스를 제공하기 위해 여러 시도들이 있었음
- 그 중 하나가 Adaptive HTTP Streaming (HTTP 프로토콜) 이다.
- Adaptive HTTP Streaming (HTTP 프로토콜)의 형태 중 하나가 HLS
- 다른 형태로는 MPEG-DASH (Dynamic Adaptive Streaming over HTTP)가 있음
- Progressive Download (HTTP 프로토콜)
  - 서버에 있는 동영상을 시청자가 다운받는 동시에 재생을 하는 방식을 이용한 스트리밍 방식
  - 동영상을 전부 다운받지 않아도 재생할 수 있다는 점에서 이후에 Adaptive Streaming 방식과 유사하나 동영상을 끝까지 보지 않더라도 영상을 다운받는 점과 각 화질별로 영상을 다 다운받아야 화질 변경을 할 수 있다는 점에서 트래픽을 많이 요구한다는 단점
  - 또한, 동영상이 결국엔 사용자의 PC의 임시 폴더에 저장된다는 점에서 보안에 취약

- Adaptive HTTP Streaming (HTTP 프로토콜)
  - 하나의 동영상을 Chunk라 불리는 작은 단위의 파일로 쪼개어 재생하는 것을 말함
  - 20분짜리 영상을 10초 단위로 나누어 저장하여 총 240개(음성 + 영상)의 chunk 파일로 만듬
  - N초 단위로 나눠 저장했기에 재생하는 시간보다 약간의 시간만 앞서서 데이터를 가져오면 되고, 영상이 나뉘어 있기 때문에 화질을 변경했을 때도 요청한 시간에 해당하는 Chunk를 다운받으면 되어서 리소스 전송에 효율적이라는 장점
  - 사전에 화질별, N초 별로 나누는 트랜스 코딩 과정이 필요하다는 번거로움이 있음


## 1. **브라우저 네이티브 지원 (제한적)**

```html
<!-- 일부 브라우저에서는 직접 재생 가능 (Safari 등) -->
<video controls>
  <source src="video.ts" type="video/mp2t">
  Your browser does not support TS format.
</video>
```

**문제점:**
- 브라우저마다 지원이 다름
- Chrome, Firefox는 네이티브 지원 안 함
- Safari는 HLS 맥락에서 지원

## 2. **HLS (HTTP Live Streaming) 사용 (권장)**

`.ts` 파일은 주로 HLS 스트리밍에서 사용됩니다.

### 방법 A: hls.js 라이브러리 사용 (가장 일반적)

```html
<!DOCTYPE html>
<html>
<head>
  <title>HLS Video Player</title>
</head>
<body>
  <video id="video" controls width="640" height="360"></video>
  
  <!-- hls.js 라이브러리 로드 -->
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  
  <script>
    const video = document.getElementById('video');
    const videoSrc = 'https://example.com/playlist.m3u8';  // HLS 플레이리스트
    
    if (Hls.isSupported()) {
      // hls.js 지원 브라우저 (Chrome, Firefox 등)
      const hls = new Hls();
      hls.loadSource(videoSrc);
      hls.attachMedia(video);
      
      hls.on(Hls.Events.MANIFEST_PARSED, function() {
        video.play();
      });
      
      // 에러 처리
      hls.on(Hls.Events.ERROR, function(event, data) {
        console.error('HLS Error:', data);
      });
      
    } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
      // Safari - 네이티브 HLS 지원
      video.src = videoSrc;
      video.addEventListener('loadedmetadata', function() {
        video.play();
      });
    } else {
      alert('This browser does not support HLS');
    }
  </script>
</body>
</html>
```

### 방법 B: Video.js + videojs-contrib-hls

```html
<!DOCTYPE html>
<html>
<head>
  <link href="https://vjs.zencdn.net/8.6.1/video-js.css" rel="stylesheet" />
</head>
<body>
  <video 
    id="my-video" 
    class="video-js vjs-default-skin" 
    controls 
    preload="auto"
    width="640" 
    height="360">
  </video>

  <script src="https://vjs.zencdn.net/8.6.1/video.min.js"></script>
  
  <script>
    const player = videojs('my-video', {
      sources: [{
        src: 'https://example.com/playlist.m3u8',
        type: 'application/x-mpegURL'
      }]
    });
    
    player.ready(function() {
      console.log('Player is ready');
    });
  </script>
</body>
</html>
```

## 3. **개별 .ts 파일 직접 재생 (변환 필요)**

`.ts` 파일을 직접 재생하려면 MP4로 변환하는 것이 좋습니다.

### 옵션 A: 서버에서 변환 (ffmpeg)

```bash
# .ts를 .mp4로 변환
ffmpeg -i input.ts -c:v copy -c:a copy output.mp4
```

### 옵션 B: MediaSource API 사용 (고급)

```html
<video id="video" controls></video>

<script>
  const video = document.getElementById('video');
  const mediaSource = new MediaSource();
  
  video.src = URL.createObjectURL(mediaSource);
  
  mediaSource.addEventListener('sourceopen', async function() {
    const sourceBuffer = mediaSource.addSourceBuffer('video/mp2t; codecs="avc1.42E01E,mp4a.40.2"');
    
    // .ts 파일 fetch
    const response = await fetch('video.ts');
    const arrayBuffer = await response.arrayBuffer();
    
    sourceBuffer.addEventListener('updateend', function() {
      if (!sourceBuffer.updating && mediaSource.readyState === 'open') {
        mediaSource.endOfStream();
      }
    });
    
    sourceBuffer.appendBuffer(arrayBuffer);
  });
</script>
```

## 4. **HLS 플레이리스트 구조**

```
# playlist.m3u8 (마스터 플레이리스트)
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:0

#EXTINF:10.0,
segment0.ts
#EXTINF:10.0,
segment1.ts
#EXTINF:10.0,
segment2.ts
#EXT-X-ENDLIST
```

## 5. **브라우저 지원 현황**

| 브라우저 | 네이티브 .ts 지원 | HLS 네이티브 지원 | hls.js 필요 |
|----------|------------------|------------------|-------------|
| Chrome | ❌ | ❌ | ✅ |
| Firefox | ❌ | ❌ | ✅ |
| Safari | ✅ (HLS) | ✅ | ❌ |
| Edge | ❌ | ❌ | ✅ |
| Mobile Safari | ✅ (HLS) | ✅ | ❌ |

## 6. **실전 예제: 완전한 HLS 플레이어**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>TS Video Player</title>
  <style>
    #video-container {
      max-width: 800px;
      margin: 20px auto;
    }
    video {
      width: 100%;
      height: auto;
    }
    .controls {
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div id="video-container">
    <video id="video" controls></video>
    <div class="controls">
      <button id="play">Play</button>
      <button id="pause">Pause</button>
      <span id="status"></span>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <script>
    const video = document.getElementById('video');
    const playBtn = document.getElementById('play');
    const pauseBtn = document.getElementById('pause');
    const status = document.getElementById('status');
    
    const videoSrc = 'https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8';
    
    function updateStatus(msg) {
      status.textContent = msg;
      console.log(msg);
    }
    
    if (Hls.isSupported()) {
      const hls = new Hls({
        debug: true,
        enableWorker: true,
        lowLatencyMode: true
      });
      
      hls.loadSource(videoSrc);
      hls.attachMedia(video);
      
      hls.on(Hls.Events.MANIFEST_PARSED, function() {
        updateStatus('Ready to play');
      });
      
      hls.on(Hls.Events.ERROR, function(event, data) {
        if (data.fatal) {
          switch(data.type) {
            case Hls.ErrorTypes.NETWORK_ERROR:
              updateStatus('Network error');
              hls.startLoad();
              break;
            case Hls.ErrorTypes.MEDIA_ERROR:
              updateStatus('Media error');
              hls.recoverMediaError();
              break;
            default:
              updateStatus('Fatal error');
              hls.destroy();
              break;
          }
        }
      });
      
    } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
      video.src = videoSrc;
      updateStatus('Using native HLS');
    } else {
      updateStatus('HLS not supported');
    }
    
    playBtn.addEventListener('click', () => video.play());
    pauseBtn.addEventListener('click', () => video.pause());
  </script>
</body>
</html>
```

## 요약:

| 방법 | 권장도 | 사용 사례 |
|------|--------|----------|
| **hls.js** | 5 | HLS 스트리밍 (권장) |
| **Video.js** | 4 | 풍부한 플레이어 UI 필요시 |
| **네이티브 video** | 2 | Safari만 지원 |
| **MP4 변환** | 3 | 단일 파일, 호환성 중요시 |