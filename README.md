# HLS-Adaptive-Streaming
# 🎥 AWS Lambda Video HLS Transcoder

This project is a **serverless video transcoding pipeline** that runs inside **AWS Lambda**. It downloads videos from **Amazon S3**, processes them with **FFmpeg**, generates **HLS (HTTP Live Streaming) playlists and segments**, and uploads them back to S3.  
The resulting HLS streams can be served efficiently using **Amazon CloudFront** for adaptive video streaming.

---

## ✨ Features

- ✅ Download videos from S3  
- ✅ Extract video metadata (resolution, aspect ratio, rotation) using **ffprobe**  
- ✅ Generate multiple **HLS renditions** (different resolutions & bitrates)  
- ✅ Create both:
  - **Master playlist** (`master.m3u8`)
  - **Low-bitrate master playlist** (`low_master.m3u8`) for constrained devices  
- ✅ Upload processed HLS files back to S3  
- ✅ Output CloudFront-ready URL for playback  
- ✅ Handles video rotation and maintains aspect ratio  
- ✅ Configurable encoding presets and CRF for quality/speed trade-offs  
- ✅ Cleans up temporary files in `/tmp` (Lambda’s writable directory)  

---

## 🛠️ Tech Stack

- **Node.js 18+** (AWS Lambda runtime)  
- **AWS SDK v3** (`@aws-sdk/client-s3`, `@aws-sdk/lib-storage`)  
- **FFmpeg + FFprobe** (provided via [Lambda Layer](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html))  
- **fluent-ffmpeg** for building and running FFmpeg commands  

---

## 📂 Project Structure

