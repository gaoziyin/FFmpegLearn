# 6.3 Performance Tuning

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Optimize FFmpeg for multi-core CPUs
- Configure memory and buffer settings
- Monitor and improve encoding performance
- Balance speed and resource usage

---

## 🔄 Threading Optimization

### Thread Settings

```bash
# Use all CPU cores
ffmpeg -threads 0 -i input.mp4 -c:v libx264 output.mp4

# Specific thread count
ffmpeg -threads 8 -i input.mp4 -c:v libx264 output.mp4
```

### x264 Thread Options

```bash
ffmpeg -i input.mp4 -c:v libx264 \
  -x264-params "threads=auto:lookahead_threads=2" \
  output.mp4
```

---

## 💾 Buffer Settings

### Input/Output Buffers

```bash
# Larger input buffer for network sources
ffmpeg -fflags +genpts -probesize 100M -analyzeduration 100M \
  -i input.mp4 output.mp4
```

### Streaming Buffers

```bash
ffmpeg -i input.mp4 \
  -bufsize 6M -maxrate 3M \
  output.mp4
```

---

## ⚡ Speed Optimizations

### Preset Selection

| Preset | Speed Multiplier |
|--------|------------------|
| ultrafast | 10x |
| veryfast | 5x |
| fast | 2x |
| medium | 1x |
| slow | 0.5x |

### Quick Encode Settings

```bash
# Maximum speed
ffmpeg -i input.mp4 \
  -c:v libx264 -preset ultrafast -crf 23 \
  output.mp4

# Balanced
ffmpeg -i input.mp4 \
  -c:v libx264 -preset fast -crf 23 \
  output.mp4
```

---

## 📊 Monitoring Performance

### Show Progress

```bash
ffmpeg -i input.mp4 -c:v libx264 -stats output.mp4
```

### Benchmark Mode

```bash
ffmpeg -benchmark -i input.mp4 -c:v libx264 -f null -
```

---

## ✅ Best Practices

> [!TIP]
> **Use -threads 0**: Let FFmpeg auto-detect optimal thread count.

> [!IMPORTANT]
> **Monitor Memory**: Large files may require increased buffer sizes.

---

## 📝 Summary

| Setting | Purpose |
|---------|---------|
| `-threads 0` | Auto thread count |
| `-preset fast` | Speed/quality balance |
| `-bufsize` | Output buffer |
| `-benchmark` | Performance testing |

---

## ➡️ Next Steps

Proceed to [6.4 Troubleshooting](../6.4-troubleshooting/) for error resolution.
