# FFmpeg API (FFAPI)

This API allows you to run FFmpeg commands with various customization options, including downloading and uploading files in chunks, managing job concurrency, and automating the deletion of output files after a specified TTL (Time-to-Live).

## Features

- **Single Job Execution**: Multiple requests with the same ID will only run one job. Subsequent requests return the saved result until the TTL expires.
- **Chunked Downloading and Uploading**: Download and upload files in small chunks to avoid throttling (useful for YouTube and large files).
- **Manual Deletion**: Option to manually delete output files before the TTL expires.
- **Customizable Input and Output Options**: Specify chunk size, concurrency, and use a range query for chunk fetching.
- **Constraints**: Set limitations on file sizes for inputs and outputs.

## Endpoint

- **POST** `/api/exec`

## Request Body Schema

```json
{
  "id": "string", // Unique identifier for the job
  "ttl": "string", // Time-to-Live for the job in milliseconds
  "commands": ["string"], // Array of FFmpeg command arguments
  "inputs": [
    // Array of input objects
    {
      "name": "string", // Name of the input file
      "url": "string", // URL of the input file
      "chunkSize": "number", // (Optional) Size of each chunk in bytes (minimum 1KB)
      "queueSize": "number", // (Optional) Number of chunks to download simultaneously
      "chunkWithQuery": "boolean", // (Optional) Fetch chunks using a range query (default: false)
      "useProxy": "boolean", // (Optional) Use proxies to download file (default: true)
      "timeout": "number", // (Optional) Time limit for downloading file in milliseconds (default: 60000; 1 minute)
      "params": {
        // (Optional) Additional HTTP request parameters
        "method": "string", // (Optional) GET, POST, HEAD etc (default: GET)
        "headers": "object", // (Optional) HTTP headers
        "body": "string" // (Optional) HTTP request data
      }
    }
  ],
  "outputs": [
    // Array of output objects
    {
      "name": "string", // Name of the output file
      "chunkSize": "number", // (Optional) Size of each chunk in bytes for upload (minimum 5MB, default 30MB)
      "queueSize": "number", // (Optional) Number of chunks to upload simultaneously (default: 6)
      "CDNCaching": "boolean", // (Optional) Enable caching for output URL (default: true)
      "downloadName": "string" // (Optional) Set download filename for the output URL
    }
  ],
  "constraints": {
    // (Optional) Constraints for file sizes
    "single_input_file_size": "number",
    "single_output_file_size": "number",
    "total_inputs_file_size": "number",
    "total_outputs_file_size": "number"
  }
}
```

### Example Request Body

```json
{
  "id": "sample-job",
  "ttl": "180000", // expire in 3 minutes
  "commands": [
    "-i",
    "video.mp4", // File name from inputs
    "-c",
    "copy",
    "output.mp4"
  ],
  "inputs": [
    {
      "name": "video.mp4",
      "url": "https://example.com/video.mp4",
      "chunkSize": 1048576, // Downloads in 1MB chunks
      "queueSize": 4, // Downloads 4 chunks at the same time
      "chunkWithQuery": false,
      "useProxy": true,
      "params": {
        "method": "POST",
        "headers": {
          "Authorization": "Bearer key"
        },
        "body": "hello world!"
      }
    }
  ],
  "outputs": [
    {
      "name": "output.mp4",
      "chunkSize": 31457280, // Uploads in 30MB chunks
      "queueSize": 6, // Uploads 6 chunks at the same time
      "CDNCaching": true,
      "downloadName": "processed_video.mp4"
    }
  ],
  "constraints": {
    "single_output_file_size": 1073741824 // 1GB limit for single output file
  }
}
```

## Response Body Schema

```json
{
  "success": "boolean", // Indicates if the operation was successful
  "msg": "string", // Response message
  "result": {
    "processingTime": "number", // Time taken to process the command (in milliseconds)
    "ffmpegLogs": "string", // Detailed FFmpeg logs
    "ttl": "string", // Expiration time in ISO format
    "outputs": [
      // Array of output files
      {
        "name": "string", // Name of the output file
        "size": "number", // Size of the output file in bytes
        "url": "string", // URL to access the output file
        "deleteUrl": "string" // URL to delete the output file
      }
    ]
  }
}
```

### Example Response Body

```json
{
  "success": true,
  "msg": "ok",
  "result": {
    "processingTime": 547.05,
    "ffmpegLogs": "ffmpeg version... detailed logs",
    "ttl": "2024-08-25T14:38:33.846Z",
    "outputs": [
      {
        "name": "output.mp4",
        "size": 3129923,
        "url": "https://example.com/storage/output.mp4",
        "deleteUrl": "https://example.com/storage/output.mp4?delete"
      }
    ]
  }
}
```

##

##

##

##

## Example Usages

### 1. Joining Audio and Video (YOUTUBE)

Combine an audio file and a video file, specifically from youtube

```json
{
  "id": "dbHkdVhuK_1080p",
  "ttl": 3600000,
  "inputs": [
    {
      "chunkSize": 8961472,
      "queueSize": 8,
      "chunkWithQuery": false,
      "useProxy": true,
      "name": "video.mp4",
      "url": "https://redirector.googlevideo.com/videoplayback"
    },
    {
      "chunkSize": 8961472,
      "queueSize": 8,
      "chunkWithQuery": false,
      "useProxy": true,
      "name": "audio.m4a",
      "url": "https://redirector.googlevideo.com/videoplayback"
    }
  ],
  "commands": [
    "-i",
    "video.mp4",
    "-i",
    "audio.m4a",
    "-c",
    "copy",
    "output.mp4"
  ],
  "outputs": [
    {
      "name": "output.mp4",
      "downloadName": "combined_video_audio.mp4"
    }
  ]
}
```

### 2. Joining Audio and Video

Combine an audio file and a video file without reencoding (copy streams).

```json
{
  "id": "join-example",
  "ttl": "3600000",
  "commands": [
    "-i",
    "video.mp4",
    "-i",
    "audio.aac",
    "-c:v",
    "copy",
    "-c:a",
    "copy",
    "output.mp4"
  ],
  "inputs": [
    {
      "name": "video.mp4",
      "url": "https://example.com/video.mp4"
    },
    {
      "name": "audio.aac",
      "url": "https://example.com/audio.aac"
    }
  ],
  "outputs": [
    {
      "name": "output.mp4",
      "chunkSize": 52428800,
      "queueSize": 4,
      "CDNCaching": true,
      "downloadName": "combined_video_audio.mp4"
    }
  ]
}
```

### 3. Extract Video Stream

Extract the video stream only, without reencoding it.

```json
{
  "id": "extract-video-example",
  "ttl": "3600000",
  "commands": ["-i", "video.mp4", "-an", "-c:v", "copy", "output.mp4"],
  "inputs": [
    {
      "name": "video.mp4",
      "url": "https://example.com/video.mp4"
    }
  ],
  "outputs": [
    {
      "name": "output.mp4",
      "chunkSize": 20971520,
      "queueSize": 5,
      "CDNCaching": false,
      "downloadName": "video_only.mp4"
    }
  ]
}
```

### 4. Extract Audio Stream

Extract the audio stream only, without reencoding it.

```json
{
  "id": "extract-audio-example",
  "ttl": "3600000",
  "commands": ["-i", "video.mp4", "-vn", "-c:a", "copy", "output.aac"],
  "inputs": [
    {
      "name": "video.mp4",
      "url": "https://example.com/video.mp4"
    }
  ],
  "outputs": [
    {
      "name": "output.aac",
      "chunkSize": 10485760,
      "queueSize": 3,
      "CDNCaching": true,
      "downloadName": "audio_only.aac"
    }
  ]
}
```

### 5. Normal Example

Download a video file, transcode it, and save the output.

```json
{
  "id": "normal-example",
  "ttl": "3600000",
  "commands": [
    "-i",
    "video.mp4",
    "-c:v",
    "libx264",
    "-crf",
    "20",
    "output.mp4"
  ],
  "inputs": [
    {
      "name": "video.mp4",
      "url": "https://example.com/video.mp4"
    }
  ],
  "outputs": [
    {
      "name": "output.mp4",
      "chunkSize": 31457280,
      "queueSize": 6,
      "CDNCaching": true,
      "downloadName": "transcoded_video.mp4"
    }
  ]
}
```

---

This documentation should give you a thorough understanding of how to use your API, covering all essential details, example requests, and common use cases. If there’s anything more specific you’d like to add or adjust, let me know!