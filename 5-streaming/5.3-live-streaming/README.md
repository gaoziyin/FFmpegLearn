# 5.3 Live Streaming

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Capture and stream live sources
- Configure low-latency encoding
- Stream to popular platforms
- Handle live streaming challenges

---

## 📹 Capturing Live Sources

### Webcam (Windows)

```bash
# List available devices
ffmpeg -list_devices true -f dshow -i dummy

# Capture webcam + microphone
ffmpeg -f dshow -i video="Webcam Name":audio="Microphone Name" \
  -c:v libx264 -preset veryfast -b:v 2500k \
  -c:a aac -b:a 128k \
  output.mp4
```

### Screen Capture (Windows)

```bash
# Capture screen
ffmpeg -f gdigrab -framerate 30 -i desktop \
  -c:v libx264 -preset ultrafast \
  output.mp4

# Capture specific window
ffmpeg -f gdigrab -framerate 30 -i title="Window Title" output.mp4
```

### Screen Capture (macOS)

```bash
# Capture screen
ffmpeg -f avfoundation -framerate 30 -i "1:0" output.mp4
```

---

## 🌐 Streaming to Platforms

### YouTube Live

```bash
ffmpeg -f dshow -i video="Webcam":audio="Mic" \
  -c:v libx264 -preset veryfast -b:v 4500k -maxrate 4500k -bufsize 9000k \
  -g 60 -keyint_min 60 \
  -c:a aac -b:a 128k -ar 44100 \
  -f flv rtmp://a.rtmp.youtube.com/live2/YOUR_STREAM_KEY
```

### Twitch

```bash
ffmpeg -f dshow -i video="Webcam":audio="Mic" \
  -c:v libx264 -preset veryfast -b:v 6000k -maxrate 6000k -bufsize 12000k \
  -g 60 -keyint_min 60 \
  -c:a aac -b:a 160k -ar 48000 \
  -f flv rtmp://live.twitch.tv/app/YOUR_STREAM_KEY
```

### Facebook Live

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 -preset veryfast -b:v 4000k \
  -c:a aac -b:a 128k \
  -f flv rtmps://live-api-s.facebook.com:443/rtmp/YOUR_STREAM_KEY
```

---

## ⚡ Low-Latency Settings

```bash
# Ultra-low latency streaming
ffmpeg -f dshow -i video="Webcam":audio="Mic" \
  -c:v libx264 -preset ultrafast -tune zerolatency \
  -b:v 3000k -maxrate 3000k -bufsize 1500k \
  -g 30 -keyint_min 30 \
  -c:a aac -b:a 128k \
  -f flv rtmp://server/live/stream
```

### Key Low-Latency Options

| Option | Purpose |
|--------|---------|
| `-preset ultrafast` | Fastest encoding |
| `-tune zerolatency` | Disable latency-adding features |
| `-g 30` | Short GOP (1 second at 30fps) |
| `-bufsize` | Small buffer = lower latency |

---

## 🔄 Re-streaming

### Stream to Multiple Platforms

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 -b:v 4500k -c:a aac -b:a 128k \
  -f tee "[f=flv]rtmp://youtube/key|[f=flv]rtmp://twitch/key"
```

### Record While Streaming

```bash
ffmpeg -f dshow -i video="Webcam":audio="Mic" \
  -c:v libx264 -preset fast -b:v 4500k \
  -c:a aac -b:a 128k \
  -f tee "[f=flv]rtmp://server/live/stream|[f=mp4]recording.mp4"
```

---

## ✅ Best Practices

> [!TIP]
> **Use -tune zerolatency**: Essential for real-time streaming.

> [!IMPORTANT]
> **Set Keyframe Interval**: Match platform requirements (usually 2 seconds).

> [!WARNING]
> **Monitor Bandwidth**: Ensure upload speed exceeds bitrate by 50%+.

---

## 📝 Summary

| Platform | Server |
|----------|--------|
| YouTube | `rtmp://a.rtmp.youtube.com/live2/KEY` |
| Twitch | `rtmp://live.twitch.tv/app/KEY` |
| Facebook | `rtmps://live-api-s.facebook.com:443/rtmp/KEY` |

---

## ➡️ Next Steps

Proceed to [5.4 Progressive Download](../5.4-progressive-download/) for HTTP delivery optimization.
