# FFmpeg API (FFAPI)

This API allows you to run FFmpeg commands with various customization options, including downloading files in chunks, managing job concurrency, and automating the deletion of output files after a specified TTL (Time-to-Live).

## Features
- **Single Job Execution**: Multiple requests with the same ID will only run one job. Subsequent requests return the saved result until the TTL expires.
- **Chunked Downloading**: Download files in small chunks to avoid throttling (useful for YouTube).
- **Manual Deletion**: Option to manually delete output files before the TTL expires.
- **Customizable Input Options**: Specify chunk size, concurrency, and use a range query for chunk fetching.

## Endpoint
- **POST** `/api/exec`

## Request Body Schema
```json
{
  "id": "string",             // Unique identifier for the job
  "ttl": "number",            // Time-to-Live for the job in milliseconds
  "inputs": [                 // Array of input objects
    {
      "url": "string",        // URL of the input file
      "name": "string",       // Name of the input file
      "chunkSize": "number",  // (Optional) Size of each chunk in bytes
      "concurrent": "number", // (Optional) Number of chunks to download simultaneously
      "params": {             // (Optional) Additional parameters for the fetch request
        "method": "string",   // HTTP method (GET, POST, etc.)
        "headers": {},        // Custom headers as key-value pairs
        "body": "string",     // Request body
        "credentials": "string", // Include credentials (e.g., 'same-origin')
        "cache": "string",    // Cache mode (e.g., 'default', 'no-cache')
        "mode": "string",     // Request mode (e.g., 'cors', 'no-cors')
        "redirect": "string", // Redirect mode (e.g., 'follow', 'manual')
        "referrerPolicy": "string" // Referrer policy (e.g., 'no-referrer')
      },
      "chunkWithQuery": "boolean" // (Optional) Fetch chunks using a range query (default: 'false')
    }
  ],
  "command": ["string"],      // Array of FFmpeg command arguments
  "outputs": ["string"]       // Array of output file names
}
```

### Example Request Body
```json
{
  "id": "sample-job", 
  "ttl": 180000,             // expire in 3 minutes
  "inputs": [
    {
      "url": "https://example.com/video.mp4", 
      "name": "video.mp4",
      "chunkSize": 1048576, // Downloads in 1MB chunks
      "concurrent": 4,      // Downloads 4 chunks at thesame time
      "params": {
        "method": "GET", 
      },
      "chunkWithQuery": false
    }
  ], 
  "command": [
    "-i", 
    "video.mp4",            // File name from inputs
    "-c", 
    "copy", 
    "output.mp4"
  ], 
  "outputs": [
    "output.mp4"            // If your ffmpeg command generates multiple outputs, add it here, so it will be uploaded
  ]
}
```

## Response Body Schema
```json
{
  "success": "boolean",          // Indicates if the operation was successful
  "msg": "string",               // Response message
  "result": {
    "processingTime": "number",  // Time taken to process the command (in milliseconds)
    "ffmpegLogs": "string",      // Detailed FFmpeg logs
    "ttl": "string",             // Expiration time in ISO format
    "outputs": [                 // Array of output files
      {
        "name": "string",        // Name of the output file
        "size": "number",        // Size of the output file in bytes
        "url": "string",         // URL to access the output file
        "deleteUrl": "string"    // URL to delete the output file
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
## Example Usages

### 1. Normal Example
Download a video file, transcode it, and save the output.
```json
{
  "id": "normal-example",
  "ttl": 3600000, 
  "inputs": [
    {
      "url": "https://example.com/video.mp4",
      "name": "video.mp4"
    }
  ], 
  "command": [
    "-i",
    "video.mp4",
    "-c:v",
    "libx264",
    "-crf",
    "20",
    "output.mp4"
  ], 
  "outputs": [
    "output.mp4"
  ]
}
```

### 2. Joining Audio and Video 
Combine an audio file and a video file without reencoding (copy streams).
```json
{
  "id": "join-example",
  "ttl": 3600000,
  "inputs": [
    {
      "url": "https://example.com/video.mp4",
      "name": "video.mp4"
    },
    {
      "url": "https://example.com/audio.aac",
      "name": "audio.aac"
    }
  ], 
  "command": [
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
  "outputs": [
    "output.mp4"
  ]
}
```

### 3. Extract Video Stream 
Extract the video stream only, without reencoding it.
```json
{
  "id": "extract-video-example",
  "ttl": 3600000,
  "inputs": [
    {
      "url": "https://example.com/video.mp4",
      "name": "video.mp4"
    }
  ],
  "command": [
    "-i",
    "video.mp4",
    "-an",          // Disable audio
    "-c:v",
    "copy",
    "output.mp4"
  ],
  "outputs": [
    "output.mp4"
  ]
}
```

### 4. Extract Audio Stream 
Extract the audio stream only, without reencoding it.
```json
{
  "id": "extract-audio-example",
  "ttl": 3600000,
  "inputs": [
    {
      "url": "https://example.com/video.mp4",
      "name": "video.mp4"
    }
  ],
  "command": [
    "-i",
    "video.mp4",
    "-vn",          // Disable video
    "-c:a",
    "copy",
    "output.aac"
  ],
  "outputs": [
    "output.aac"
  ]
}
```

---

This documentation should give you a thorough understanding of how to use your API, covering all essential details, example requests, and common use cases. If there’s anything more specific you’d like to add or adjust, let me know!
