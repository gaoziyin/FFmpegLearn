# 7.2 Scripting and CI/CD Integration

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Create production-ready FFmpeg scripts
- Integrate FFmpeg in CI/CD pipelines
- Use Docker for consistent processing
- Implement automated media workflows

---

## 🐳 Docker Integration

### FFmpeg Docker Image

```dockerfile
# Dockerfile
FROM jrottenberg/ffmpeg:4.4-scratch

WORKDIR /data
ENTRYPOINT ["ffmpeg"]
```

### Run FFmpeg in Docker

```bash
# Process file with Docker
docker run -v $(pwd):/data jrottenberg/ffmpeg:4.4-scratch \
  -i /data/input.mp4 -c:v libx264 /data/output.mp4
```

---

## 🔧 GitHub Actions

### Video Processing Workflow

```yaml
# .github/workflows/video-process.yml
name: Process Video

on:
  push:
    paths:
      - 'videos/raw/**'

jobs:
  process:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install FFmpeg
        run: |
          sudo apt-get update
          sudo apt-get install -y ffmpeg
      
      - name: Process Videos
        run: |
          for f in videos/raw/*.mp4; do
            output="videos/processed/$(basename $f)"
            ffmpeg -i "$f" -c:v libx264 -crf 23 "$output"
          done
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: processed-videos
          path: videos/processed/
```

---

## 📜 Production Scripts

### Bash Processing Script

```bash
#!/bin/bash
# process_video.sh

set -e  # Exit on error

INPUT="$1"
OUTPUT_DIR="$2"

if [ -z "$INPUT" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <input> <output_dir>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

BASENAME=$(basename "$INPUT" .mp4)

# Generate thumbnails
ffmpeg -i "$INPUT" -ss 00:00:05 -vframes 1 \
  "$OUTPUT_DIR/${BASENAME}_thumb.jpg" -y

# Create web version
ffmpeg -i "$INPUT" \
  -vf "scale=-2:720" \
  -c:v libx264 -crf 23 \
  -c:a aac -b:a 128k \
  -movflags +faststart \
  "$OUTPUT_DIR/${BASENAME}_720p.mp4" -y

echo "Processing complete!"
```

### PowerShell Processing Script

```powershell
# Process-Video.ps1
param(
    [Parameter(Mandatory=$true)]
    [string]$Input,
    
    [Parameter(Mandatory=$true)]
    [string]$OutputDir
)

$ErrorActionPreference = "Stop"

if (!(Test-Path $OutputDir)) {
    New-Item -ItemType Directory -Path $OutputDir | Out-Null
}

$baseName = [System.IO.Path]::GetFileNameWithoutExtension($Input)

# Generate thumbnail
ffmpeg -i $Input -ss 00:00:05 -vframes 1 `
    "$OutputDir\${baseName}_thumb.jpg" -y

# Create web version
ffmpeg -i $Input `
    -vf "scale=-2:720" `
    -c:v libx264 -crf 23 `
    -c:a aac -b:a 128k `
    -movflags +faststart `
    "$OutputDir\${baseName}_720p.mp4" -y

Write-Host "Processing complete!"
```

---

## ✅ Best Practices

> [!TIP]
> **Use set -e in Bash**: Script stops on first error.

> [!IMPORTANT]
> **Pin FFmpeg Versions**: Use specific versions in CI/CD for reproducibility.

---

## 📝 Summary

| Platform | Integration Method |
|----------|-------------------|
| Docker | `jrottenberg/ffmpeg` image |
| GitHub Actions | `apt-get install ffmpeg` |
| Jenkins | Shell scripts |
| GitLab CI | Docker executor |

---

## ➡️ Next Steps

Proceed to [7.3 Library Integration](../7.3-library-integration/) for programmatic FFmpeg usage.
