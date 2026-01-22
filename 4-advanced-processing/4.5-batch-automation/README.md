# 4.5 Batch Processing and Automation

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Process multiple files automatically
- Create reusable FFmpeg scripts
- Build automated media pipelines
- Handle errors gracefully in batch operations

---

## 🔄 Batch Processing Overview

```mermaid
flowchart LR
    INPUT[Multiple Files] --> SCRIPT[Batch Script]
    SCRIPT --> PROCESS[FFmpeg Processing]
    PROCESS --> OUTPUT[Processed Files]
```

---

## 💻 Windows PowerShell

### Basic Loop

```powershell
# Convert all MP4 to MKV
Get-ChildItem *.mp4 | ForEach-Object {
    $output = $_.BaseName + ".mkv"
    ffmpeg -i $_.FullName -c copy $output
}
```

### With Progress

```powershell
$files = Get-ChildItem *.mp4
$total = $files.Count
$i = 0

foreach ($file in $files) {
    $i++
    Write-Progress -Activity "Converting" -Status "$i of $total" -PercentComplete (($i/$total)*100)
    
    $output = $file.BaseName + "_converted.mp4"
    ffmpeg -i $file.FullName -c:v libx264 -crf 23 -c:a aac $output -y -loglevel error
}

Write-Progress -Activity "Converting" -Completed
```

### Recursive Processing

```powershell
# Process all MP4 files in subdirectories
Get-ChildItem -Recurse -Filter "*.mp4" | ForEach-Object {
    $output = $_.DirectoryName + "\" + $_.BaseName + "_720p.mp4"
    ffmpeg -i $_.FullName -vf "scale=-2:720" -c:v libx264 -c:a copy $output -y
}
```

### Parallel Processing

```powershell
# Process files in parallel (PowerShell 7+)
Get-ChildItem *.mp4 | ForEach-Object -Parallel {
    $output = $_.BaseName + "_processed.mp4"
    ffmpeg -i $_.FullName -c:v libx264 -crf 23 -c:a aac $output -y -loglevel error
} -ThrottleLimit 4
```

---

## 🐧 Linux/macOS Bash

### Basic Loop

```bash
# Convert all MP4 to MKV
for f in *.mp4; do
    ffmpeg -i "$f" -c copy "${f%.mp4}.mkv"
done
```

### With Error Handling

```bash
#!/bin/bash
for f in *.mp4; do
    output="${f%.mp4}_converted.mp4"
    
    if ffmpeg -i "$f" -c:v libx264 -crf 23 -c:a aac "$output" -y; then
        echo "✓ Converted: $f"
    else
        echo "✗ Failed: $f"
    fi
done
```

### Recursive with Find

```bash
# Process all MP4 files recursively
find . -name "*.mp4" -exec bash -c '
    output="${1%.mp4}_720p.mp4"
    ffmpeg -i "$1" -vf "scale=-2:720" "$output" -y
' _ {} \;
```

### Parallel with GNU Parallel

```bash
# Process 4 files at a time
ls *.mp4 | parallel -j 4 'ffmpeg -i {} -c:v libx264 -crf 23 {.}_out.mp4'
```

---

## 📁 Reusable Scripts

### Conversion Script (PowerShell)

```powershell
# convert-videos.ps1
param(
    [Parameter(Mandatory=$true)]
    [string]$InputPath,
    
    [string]$OutputFormat = "mp4",
    [string]$Resolution = "1080",
    [int]$CRF = 23
)

$files = Get-ChildItem -Path $InputPath -Filter "*.mp4"

foreach ($file in $files) {
    $output = Join-Path $InputPath ("converted_" + $file.BaseName + "." + $OutputFormat)
    
    $scale = "scale=-2:$Resolution"
    
    ffmpeg -i $file.FullName `
        -vf $scale `
        -c:v libx264 -crf $CRF `
        -c:a aac `
        $output -y
    
    Write-Host "Converted: $($file.Name)"
}

# Usage: .\convert-videos.ps1 -InputPath "C:\Videos" -Resolution 720 -CRF 26
```

### Conversion Script (Bash)

```bash
#!/bin/bash
# convert-videos.sh

INPUT_DIR="${1:-.}"
OUTPUT_FORMAT="${2:-mp4}"
RESOLUTION="${3:-1080}"
CRF="${4:-23}"

for f in "$INPUT_DIR"/*.mp4; do
    [ -f "$f" ] || continue
    
    base=$(basename "$f" .mp4)
    output="$INPUT_DIR/converted_${base}.$OUTPUT_FORMAT"
    
    ffmpeg -i "$f" \
        -vf "scale=-2:$RESOLUTION" \
        -c:v libx264 -crf "$CRF" \
        -c:a aac \
        "$output" -y
    
    echo "Converted: $(basename "$f")"
done

# Usage: ./convert-videos.sh /path/to/videos mp4 720 26
```

---

## 📊 Logging and Reporting

### With Log File

```powershell
$logFile = "conversion_log.txt"
$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"

Add-Content $logFile "=== Conversion started: $timestamp ==="

Get-ChildItem *.mp4 | ForEach-Object {
    $output = $_.BaseName + "_out.mp4"
    $start = Get-Date
    
    $result = ffmpeg -i $_.FullName -c:v libx264 -c:a aac $output -y 2>&1
    
    $duration = (Get-Date) - $start
    
    if ($LASTEXITCODE -eq 0) {
        Add-Content $logFile "SUCCESS: $($_.Name) (${duration}s)"
    } else {
        Add-Content $logFile "FAILED: $($_.Name) - $result"
    }
}
```

### Generate Report

```powershell
# Generate CSV report of all media files
$report = @()

Get-ChildItem *.mp4 | ForEach-Object {
    $info = ffprobe -v error -show_entries format=duration,size:stream=width,height,codec_name `
        -of json $_.FullName | ConvertFrom-Json
    
    $report += [PSCustomObject]@{
        Filename = $_.Name
        Duration = [math]::Round($info.format.duration, 2)
        Size_MB = [math]::Round($info.format.size / 1MB, 2)
        Resolution = "$($info.streams[0].width)x$($info.streams[0].height)"
        Codec = $info.streams[0].codec_name
    }
}

$report | Export-Csv "media_report.csv" -NoTypeInformation
```

---

## 🔧 Common Batch Operations

### Create Thumbnails

```powershell
Get-ChildItem *.mp4 | ForEach-Object {
    $thumb = $_.BaseName + "_thumb.jpg"
    ffmpeg -i $_.FullName -ss 00:00:05 -vframes 1 -q:v 2 $thumb -y
}
```

### Extract Audio from All Videos

```powershell
Get-ChildItem *.mp4 | ForEach-Object {
    $audio = $_.BaseName + ".mp3"
    ffmpeg -i $_.FullName -vn -c:a libmp3lame -q:a 2 $audio -y
}
```

### Compress for Web

```powershell
Get-ChildItem *.mp4 | ForEach-Object {
    $web = "web_" + $_.Name
    ffmpeg -i $_.FullName `
        -vf "scale=-2:720" `
        -c:v libx264 -preset fast -crf 28 `
        -c:a aac -b:a 128k `
        -movflags +faststart `
        $web -y
}
```

### Add Watermark to All

```powershell
$watermark = "logo.png"

Get-ChildItem *.mp4 | ForEach-Object {
    $output = "watermarked_" + $_.Name
    ffmpeg -i $_.FullName -i $watermark `
        -filter_complex "[0:v][1:v]overlay=W-w-10:H-h-10" `
        $output -y
}
```

---

## ⚡ Performance Tips

### Use Hardware Acceleration

```powershell
# NVIDIA GPU encoding (if available)
Get-ChildItem *.mp4 | ForEach-Object {
    $output = "hw_" + $_.Name
    ffmpeg -hwaccel cuda -i $_.FullName `
        -c:v h264_nvenc -preset fast -crf 23 `
        $output -y
}
```

### Skip Already Processed

```powershell
Get-ChildItem *.mp4 | ForEach-Object {
    $output = "converted_" + $_.Name
    
    if (Test-Path $output) {
        Write-Host "Skipping (exists): $($_.Name)"
        return
    }
    
    ffmpeg -i $_.FullName -c:v libx264 $output -y
}
```

---

## ✅ Best Practices

> [!TIP]
> **Use -y for Automation**: The `-y` flag auto-overwrites, essential for unattended scripts.

> [!TIP]
> **Add -loglevel error**: Reduces output noise in batch scripts.

> [!IMPORTANT]
> **Test on Sample First**: Always test your batch script on a few files before running on everything.

> [!WARNING]
> **Keep Originals**: Don't overwrite source files; output to different names or folders.

---

## 🏋️ Exercises

### Exercise 1: Basic Batch
Write a script to convert all AVI files to MP4 in a folder.

### Exercise 2: Thumbnails
Generate thumbnails at 10 seconds for all videos.

### Exercise 3: Report
Create a CSV report listing duration and resolution of all videos.

---

## 📝 Summary

| Platform | For Loop |
|----------|----------|
| PowerShell | `Get-ChildItem *.mp4 \| ForEach-Object { ... }` |
| Bash | `for f in *.mp4; do ... done` |
| Parallel | `parallel 'ffmpeg ...' ::: *.mp4` |

---

## 🎉 Module 4 Complete!

You've mastered advanced FFmpeg processing:
- Concatenation techniques
- Subtitle and metadata handling
- Synchronization and timestamps
- Complex filtergraphs
- Batch automation

## ➡️ Next Module

Proceed to [Module 5: Streaming](../../5-streaming/) to learn about media distribution and live streaming.
