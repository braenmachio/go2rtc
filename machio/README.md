# README for Frigate & go2rtc Integration

## Overview
This document outlines the integration plan for **go2rtc** with the **shiriki-backend** for managing camera streams.

## Video Reference
For detailed insights, check the video: [Frigate & go2rtc](https://youtu.be/PYMI_jfSizM?si=oD-pQWeo8moR4CnM).

## Integration Plan - go2rtc: shiriki-backend

### 1. Locate Relevant Code/Config
- Look for RTSP configuration, streaming services, or endpoints that can integrate go2rtc.

### 2. Detailed Integration Steps

#### Step 1: Install and Configure go2rtc
- **Download go2rtc:**
  - Visit the go2rtc GitHub page and download the binary for your system.
  - Alternatively, run go2rtc using Docker:
    ```bash
    docker run -d --name go2rtc -p 1984:1984 -p 8554:8554 -p 8555:8555 alexxit/go2rtc
    ```

- **Configuration:**
  - Create a `go2rtc.yaml` file to define your cameras and streams:
    ```yaml
    streams:
      camera1: rtsp://username:password@camera_ip/stream
      camera2: http://camera_ip/mjpeg
    ```
  - Start the service and check the streams at:
    - `http://localhost:1984/`
    - `rtsp://localhost:8554/`

#### Step 2: Modify CameraStream.py to Use go2rtc
- Update `CameraStream.py` to replace direct use of `cv2.VideoCapture` with the RTSP streams from go2rtc:
  ```python
  self.camera_url = f"rtsp://localhost:8554/{camera_name}"  # Replace camera_name with the stream ID
  self.cap = cv2.VideoCapture(self.camera_url)
  ```

#### Step 3: Add go2rtc to Flask Endpoints
- Modify `support_mult_camera_2.py` to expose go2rtc streams via API:
  ```python
  @app.route('/streams', methods=['GET'])
  def get_streams():
      streams = ["camera1", "camera2"]  # Replace with dynamic discovery if needed
      return jsonify({"streams": streams, "rtsp_url_base": "rtsp://localhost:8554/"})
  ```

#### Step 4: Configure Recording and Monitoring
- Utilize go2rtc’s built-in recording features or integrate with the existing `PersistenceManager`.
- Continue using Prometheus for monitoring metrics related to streams.

#### Step 5: Test the Integration
- Verify:
  - Streams are accessible via `http://localhost:1984/` and `rtsp://localhost:8554/`.
  - Existing endpoints interact correctly with go2rtc streams.
  - The application works seamlessly with multiple cameras.

## Key Observations

### CameraStream.py
- **Purpose:** 
  - Manages camera information and stream handling.
  
- **Attributes:**
  - `camera_id`: Camera identifier.
  - `camera_name`: Name of the camera.
  - `camera_url`: URL of the camera stream (likely RTSP or HTTP).
  - `cap`: Uses OpenCV's `cv2.VideoCapture` to process video streams.

- **Concurrency:** 
  - Streams are managed in parallel threads, suitable for multi-camera setups.

### support_mult_camera_2.py
- **Framework:** 
  - Built on Flask for serving the application and handling requests.
  
- **Dependencies:** 
  - OpenCV for video streams, Prometheus for monitoring metrics.

- **Integration with go2rtc:** 
  - Centralizes RTSP management and simplifies the existing stream handling logic.

### go2rtc logs
~~~
12:30:48.096 PM	info	go2rtc version=1.9.8 platform=linux/amd64 revision=mod.2a4a9e2
12:30:48.096 PM	info	config path=/config/go2rtc.yaml
12:30:48.099 PM	info	[rtsp] listen addr=:8554
12:30:48.100 PM	info	[webrtc] listen addr=:8555/tcp
12:30:48.100 PM	info	[api] listen addr=:1984
~~~
