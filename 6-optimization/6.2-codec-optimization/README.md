# 6.2 Advanced Codec Optimization

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Optimize H.264/H.265 for specific content
- Configure AV1 and VP9 encoders
- Balance quality, speed, and file size
- Choose optimal settings for your use case

---

## 🎬 H.264 Optimization

### Common Tune Options

| Tune | Content Type |
|------|--------------|
| `film` | High quality movies |
| `animation` | Animated content |
| `grain` | Preserve film grain |
| `stillimage` | Slideshows |
| `fastdecode` | Low-power devices |
| `zerolatency` | Real-time |

```bash
# Film content
ffmpeg -i input.mp4 -c:v libx264 -tune film -crf 20 output.mp4

# Animation
ffmpeg -i anime.mp4 -c:v libx264 -tune animation -crf 20 output.mp4
```

### Advanced x264 Settings

```bash
ffmpeg -i input.mp4 -c:v libx264 \
  -preset slow -crf 18 \
  -x264-params "ref=6:bframes=3:b-adapt=2:direct=auto:me=umh:subme=10:trellis=2" \
  output.mp4
```

---

## 🎥 H.265/HEVC Optimization

### Basic H.265

```bash
# High quality H.265
ffmpeg -i input.mp4 -c:v libx265 -preset slow -crf 24 output.mp4
```

### x265 Parameters

```bash
ffmpeg -i input.mp4 -c:v libx265 \
  -preset medium -crf 26 \
  -x265-params "pools=4:frame-threads=4:wpp=1" \
  output.mp4
```

### 10-bit Encoding

```bash
ffmpeg -i input.mp4 -c:v libx265 \
  -pix_fmt yuv420p10le \
  -crf 24 output.mp4
```

---

## 📊 AV1 Encoding

### libaom-av1 (Reference)

```bash
# AV1 with libaom (slow but efficient)
ffmpeg -i input.mp4 -c:v libaom-av1 \
  -crf 30 -cpu-used 4 \
  -row-mt 1 \
  output.webm
```

### SVT-AV1 (Faster)

```bash
# SVT-AV1 (faster alternative)
ffmpeg -i input.mp4 -c:v libsvtav1 \
  -crf 30 -preset 6 \
  output.mp4
```

---

## 📈 Quality Recommendations

| Codec | Visually Lossless | High | Medium | Low |
|-------|-------------------|------|--------|-----|
| H.264 | CRF 17-18 | 19-23 | 24-27 | 28+ |
| H.265 | CRF 22-24 | 25-28 | 29-32 | 33+ |
| AV1 | CRF 25-28 | 30-35 | 36-40 | 41+ |

---

## ✅ Best Practices

> [!TIP]
> **Use Tune Options**: They can significantly improve quality for specific content types.

> [!IMPORTANT]
> **Test on Samples**: Always test codec settings on short clips before full encodes.

---

## 📝 Summary

| Codec | Command |
|-------|---------|
| H.264 | `-c:v libx264 -crf 23` |
| H.265 | `-c:v libx265 -crf 28` |
| AV1 | `-c:v libsvtav1 -crf 30` |

---

## ➡️ Next Steps

Proceed to [6.3 Performance Tuning](../6.3-performance-tuning/) for system optimization.
