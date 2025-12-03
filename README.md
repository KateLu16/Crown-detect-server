# AI Crowd Detection Server

An intelligent crowd detection system powered by YOLOv5 and Flask server.

## Features

- 🤖 **AI Detection**: Real-time person and crowd detection in images
- 🗑️ **Auto Cleanup**: Automatic deletion of original images after processing
- 📊 **Analytics**: Detailed crowd statistics and insights
- 🌐 **REST API**: Comprehensive API interface
- 🍓 **RPi Optimized**: Optimized for Raspberry Pi deployment

## Installation

### 1. Install Dependencies

```bash
pip install flask torch torchvision opencv-python numpy scikit-learn psutil Pillow PyYAML tqdm requests
```

### 2. Download YOLOv5 Model

```bash
# Model will be automatically downloaded on first run
# Or download manually:
wget https://github.com/ultralytics/yolov5/releases/download/v7.0/yolov5s.pt
```

### 3. Run Setup (Optional)

```bash
python setup.py
```

## Usage

### 1. Start the Server

```bash
python main.py
```

Server will be available at: `http://localhost:7860`

### 2. Test the Server

```bash
python test_server.py
```

## API Endpoints

### POST /upload
Upload an image for AI analysis

**Request:**
```bash
curl -X POST -F "file=@image.jpg" http://localhost:7860/upload
```

**Response:**
```json
{
  "status": "success",
  "filename": "20250830_123456.jpg",
  "timestamp": "2025-08-30T12:34:56.789",
  "analysis": {
    "processing_time": "2.45s",
    "inference_time": "0.123s", 
    "total_people": 8,
    "crowd_analysis": {
      "total_crowds": 2,
      "total_people_in_crowds": 6,
      "isolated_people": 2,
      "crowds": [
        {
          "id": 0,
          "people_count": 4,
          "center": [320, 240],
          "bbox": [200, 150, 440, 330],
          "size": [240, 180]
        }
      ]
    }
  },
  "note": "Original image deleted after processing"
}
```

### GET /stats
Retrieve server statistics

**Response:**
```json
{
  "status": "success",
  "stats": {
    "total_images_received": 15,
    "total_analyses_completed": 15,
    "ai_detector_status": "active"
  }
}
```

### GET /latest
Get the most recent analysis result

### GET /analysis/<filename>
Get analysis result for a specific image

### POST /cleanup
Clean up old files

**Request:**
```bash
curl -X POST -H "Content-Type: application/json" \
     -d '{"keep_days": 7}' \
     http://localhost:7860/cleanup
```

### POST /clear_all
Delete all files

## Project Structure

```
project/
├── main.py                 # Main Flask server
├── rpi_crowd_detector.py   # AI crowd detection engine
├── setup.py               # Setup script
├── test_server.py         # Test script
├── requirements_rpi.txt   # Raspberry Pi dependencies
├── yolov5s.pt            # YOLO model
├── received_images/      # Received images (auto-deleted)
├── analysis_results/     # Analysis results (JSON + images)
└── README.md            # This documentation
```

## AI Detection Features

### Person Detection
- Powered by YOLOv5s model
- Confidence threshold: 0.3
- Detects "person" class only

### Crowd Analysis
- **DBSCAN Clustering**: Groups people into crowds
- **Min people per crowd**: 4 people
- **Max distance**: 50 pixels
- **Crowd Metrics**:
  - Number of crowds detected
  - People count in crowds
  - Isolated individuals count
  - Crowd positions and dimensions

### Visualization
- Green bounding boxes: Isolated individuals
- Red bounding boxes: Crowds
- Red dots: Crowd centers
- Statistics overlay on image

## Raspberry Pi Optimizations

- **CPU-only**: No GPU required
- **Memory efficient**: Image size reduced to 320px
- **Auto cleanup**: Deletes original images to save storage
- **Background monitoring**: Tracks memory usage
- **Lightweight libraries**: ARM-optimized versions

## ESP32 Integration

The server is designed to receive images from ESP32 devices:

```cpp
// ESP32 code example
WiFiClient client;
HTTPClient http;
http.begin(client, "http://192.168.1.100:7860/upload");
http.addHeader("Content-Type", "multipart/form-data");
// ... upload image data
```

## Troubleshooting

### 1. Model Download Fails
```bash
# Download manually
wget https://github.com/ultralytics/yolov5/releases/download/v7.0/yolov5s.pt
```

### 2. Memory Issues on Raspberry Pi
```bash
# Reduce image size
# In rpi_crowd_detector.py: set img_size=256 (instead of 320)
```

### 3. Import Errors
```bash
pip install --upgrade torch torchvision
```

### 4. Server Not Accessible from ESP32
- Check firewall settings
- Ensure `host="0.0.0.0"` in main.py
- Verify IP address: `ip addr show`

## Performance

### Raspberry Pi 4 (4GB RAM):
- **Processing time**: 2-5 seconds per image
- **Memory usage**: ~500MB
- **Image size**: 320x320 pixels
- **Accuracy**: 85-90% person detection

### Desktop PC:
- **Processing time**: 0.5-1 second per image
- **Memory usage**: ~200MB
- **Image size**: 640x640 pixels
- **Accuracy**: 90-95% person detection

## License

Open source - Free to use for educational and research purposes.
