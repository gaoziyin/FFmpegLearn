# 6.4 Troubleshooting Guide

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Diagnose common FFmpeg errors
- Fix encoding and decoding issues
- Resolve sync and quality problems
- Debug complex filter graphs

---

## 🔴 Common Errors

### "No such file or directory"

```bash
# Problem: File path issue
# Solution: Use quotes and check path
ffmpeg -i "path/to/file with spaces.mp4" output.mp4
```

### "Unknown encoder"

```bash
# Check available encoders
ffmpeg -encoders | grep x264

# Solution: Install or use different encoder
ffmpeg -i input.mp4 -c:v libx264 output.mp4
```

### "Invalid data found"

```bash
# Force format detection
ffmpeg -f mp4 -i corrupted.mp4 -c copy output.mp4

# Or try repair
ffmpeg -err_detect ignore_err -i broken.mp4 -c copy fixed.mp4
```

### "Height/width not divisible by 2"

```bash
# Use scale filter with -2
ffmpeg -i input.mp4 -vf "scale=-2:720" output.mp4
```

---

## 🔧 Debug Mode

```bash
# Verbose output
ffmpeg -v verbose -i input.mp4 output.mp4

# Debug level
ffmpeg -v debug -i input.mp4 output.mp4

# Show info only
ffmpeg -v info -stats -i input.mp4 output.mp4
```

---

## 🔄 Sync Issues

### Audio ahead of video

```bash
ffmpeg -i video.mp4 -itsoffset 0.5 -i video.mp4 \
  -map 0:v -map 1:a -c copy output.mp4
```

### Video ahead of audio

```bash
ffmpeg -i video.mp4 -itsoffset 0.5 -i video.mp4 \
  -map 1:v -map 0:a -c copy output.mp4
```

---

## 📋 Quick Solutions

| Error | Solution |
|-------|----------|
| File not found | Check quotes, paths |
| Unknown encoder | Check build, use -encoders |
| Odd dimensions | Use scale with -2 |
| Negative timestamps | Use -avoid_negative_ts |
| Out of memory | Reduce threads, smaller buffer |

---

## ✅ Best Practices

> [!TIP]
> **Use -v error for Scripts**: Shows only errors, cleaner output.

> [!IMPORTANT]
> **Test on Samples**: Debug on short clips before processing full files.

---

## 📝 Summary

| Debug Level | Usage |
|-------------|-------|
| `-v error` | Errors only |
| `-v warning` | Warnings + errors |
| `-v info` | Default |
| `-v verbose` | Detailed |
| `-v debug` | Everything |

---

## 🎉 Module 6 Complete!

You've learned:
- Hardware acceleration
- Codec optimization
- Performance tuning
- Troubleshooting techniques

## ➡️ Next Module

Proceed to [Module 7: Professional](../../7-professional/) for production workflows.
