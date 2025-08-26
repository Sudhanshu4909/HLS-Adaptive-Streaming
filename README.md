# HLS Video Processing Lambda

A serverless AWS Lambda function that converts video files into HLS (HTTP Live Streaming) format with multiple quality levels for adaptive bitrate streaming.

## Overview

This Lambda function automatically processes video files stored in S3, generating HLS streams with multiple resolutions and bitrates. It creates adaptive streaming playlists that allow video players to automatically adjust quality based on network conditions.

## Features

- **Multi-resolution HLS streaming**: Generates 3 quality levels (super_low, lower, low)
- **Adaptive bitrate calculation**: Automatically calculates optimal bitrates based on video dimensions
- **Portrait/landscape optimization**: Handles both orientations with proper aspect ratio preservation
- **Video rotation support**: Automatically handles rotated videos based on metadata
- **Master playlist generation**: Creates both full and low-bandwidth master playlists
- **S3 integration**: Downloads from and uploads to S3 buckets
- **CloudFront ready**: Generates URLs ready for CDN distribution

## Architecture

```
S3 Input Video → Lambda Processing → HLS Segments + Playlists → S3 Output → CloudFront CDN
```

## Quality Levels

The function generates three quality levels based on the original video resolution:

| Quality Level | Scale Factor | Target Use Case |
|---------------|--------------|-----------------|
| `super_low` | 0.7-0.8 | Very slow connections |
| `lower` | 0.8-1.0 | Mobile/moderate connections |
| `low` | 1.0 | Standard quality |

## Prerequisites

### AWS Resources
- AWS Lambda function with appropriate execution role
- S3 bucket for input and output files
- CloudFront distribution (optional but recommended)
- Lambda Layer with FFmpeg binaries

### Environment Variables
```bash
MY_AWS_REGION=us-east-1
MY_S3_BUCKET=your-video-bucket-name
```

### Lambda Layer Requirements
The function requires a Lambda Layer containing:
- FFmpeg binary at `/opt/bin/ffmpeg`
- FFprobe binary at `/opt/bin/ffprobe`

## Installation

1. **Clone or download the project files**

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Create deployment package**:
   ```bash
   zip -r hls-lambda.zip index.js node_modules/ package.json
   ```

4. **Deploy to AWS Lambda**:
   - Upload the zip file to your Lambda function
   - Set the handler to `index.handler`
   - Configure environment variables
   - Attach the FFmpeg layer
   - Set timeout to at least 15 minutes
   - Allocate sufficient memory (recommend 3008 MB)

## Usage

### Event Structure
The Lambda function expects an event with the following structure:

```json
{
  "s3Key": "path/to/your/video.mp4"
}
```

### Example Invocation
```javascript
const event = {
  s3Key: "videos/InteractiveVideos/uuid123_mergedVideo_001.mp4"
};
```

### Response Structure
```json
{
  "statusCode": 200,
  "body": {
    "message": "Video processing completed successfully",
    "masterPlaylistUrl": "https://d198g8637lsfvs.cloudfront.net/path/to/hls/master.m3u8"
  }
}
```

## Output Structure

After processing, the function creates the following file structure in S3:

```
s3://bucket/path/to/video.mp4/hls/
├── master.m3u8           # Main adaptive playlist
├── low_master.m3u8       # Low-bandwidth playlist
├── super_low/
│   ├── index.m3u8
│   ├── segment_001.m4s
│   ├── segment_002.m4s
│   └── ...
├── lower/
│   ├── index.m3u8
│   ├── segment_001.m4s
│   └── ...
└── low/
    ├── index.m3u8
    ├── segment_001.m4s
    └── ...
```

## Configuration

### Video Encoding Settings

The function uses optimized H.264 encoding settings:

- **Codec**: H.264 (libx264)
- **Pixel Format**: YUV420P
- **Segment Duration**: 4 seconds
- **Container**: MP4 fragments (.m4s)
- **Audio**: AAC, 44.1kHz, Stereo

### Quality-specific Settings

| Quality | CRF | Preset | Audio Bitrate | Frame Rate |
|---------|-----|---------|---------------|------------|
| low | 18 | medium | 96k | Original |
| lower | 23 | veryfast | 64k | 24fps |
| super_low | 25 | veryfast | 48k | 24fps |

## Error Handling

The function includes comprehensive error handling:

- File download validation
- Video metadata verification
- FFmpeg processing errors
- S3 upload/delete operations
- Automatic cleanup of temporary files

## Performance Considerations

- **Memory**: Recommended 3008 MB for processing large videos
- **Timeout**: Set to 15 minutes for large files
- **Concurrent Executions**: Configure based on expected load
- **Cold Starts**: Consider provisioned concurrency for time-sensitive applications

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| @aws-sdk/client-s3 | ^3.613.0 | S3 operations |
| @aws-sdk/lib-storage | ^3.613.0 | Multipart uploads |
| fluent-ffmpeg | ^2.1.3 | Video processing |
| sharp | ^0.33.4 | Image processing (if needed) |

## Monitoring and Logging

The function provides detailed logging for:
- Download progress
- Video metadata analysis
- Processing progress
- Upload status
- Error details with stack traces

Monitor using CloudWatch Logs and set up alarms for:
- Function duration
- Memory usage
- Error rates
- Dead letter queue messages

## Troubleshooting

### Common Issues

1. **FFmpeg not found**
   - Ensure Lambda layer contains FFmpeg binaries at correct paths
   - Verify layer is attached to the function

2. **Out of memory errors**
   - Increase Lambda memory allocation
   - Consider processing smaller video chunks

3. **Timeout errors**
   - Increase function timeout
   - Optimize encoding presets for faster processing

4. **S3 permission errors**
   - Verify Lambda execution role has S3 read/write permissions
   - Check bucket policies and CORS settings

### Debug Mode
Enable detailed logging by modifying console.log statements or implementing structured logging with correlation IDs.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the ISC License.

## Support

For issues and questions:
- Check CloudWatch Logs for detailed error messages
- Verify all prerequisites are met
- Ensure proper IAM permissions are configured
