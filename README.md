# Birdnet-Go Docker Compose Setup

This project provides a Docker Compose setup for running the [Birdnet-Go](https://github.com/tphakala/birdnet-go) application, which performs real-time bird sound identification.

## Prerequisites

Before you begin, ensure you have the following installed:

*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose](https://docs.docker.com/compose/install/)

## Setup

1.  **Clone this repository:**

    ```bash
    git clone <this-repository-url>
    cd <repository-directory>
    ```

2.  **Configure Environment Variables:**

    Copy the example environment file and then edit it to set your desired `BIRDNET_COMMAND`.

    ```bash
    cp .env.example .env
    ```

    Edit the `.env` file:

    ```ini
    # .env
    BIRDNET_COMMAND="birdnet-go realtime --rtsp 'rtsp://username:password@192.168.0.13/live'"
    # Or for a local microphone on Linux:
    # BIRDNET_COMMAND="birdnet-go realtime --source \"sysdefault\""
    ```

    Make sure to replace the example RTSP URL with your actual stream details, or uncomment and configure the microphone command if you're using a local microphone.

## Usage

To build and run the Birdnet-Go service, navigate to the project root directory and execute:

```bash
docker-compose up --build
```

This command will:

*   Build the `birdnet-go` Docker image by pulling the latest code from the official GitHub repository.
*   Start the `birdnet-go` container.
*   Expose the web interface on port `8080`.

Once the container is running, you can access the Birdnet-Go web interface at `http://localhost:8080`.

## Stopping the Service

To stop the service and remove the containers, networks, and volumes created by `docker-compose up`:

```bash
docker-compose down
```

## Advanced: Using a Local Microphone (macOS)

For demo purposes, you can stream your Mac's microphone as an RTSP feed for Birdnet-Go to use. This requires FFmpeg and uses MediaMTX as an RTSP server (included in docker-compose.yml).

### 1. Install FFmpeg

If you don't have FFmpeg installed, install it using Homebrew:

```bash
brew install ffmpeg
```

If you don't have Homebrew, you can download a pre-built FFmpeg binary from [evermeet.cx/ffmpeg](https://evermeet.cx/ffmpeg/).

### 2. Grant Microphone Access to Terminal

Before starting, you must give your Terminal application permission to access your microphone:

1.  Open **System Settings** > **Privacy & Security** > **Microphone**.
2.  Ensure your terminal application (Terminal.app or iTerm2) has microphone access enabled.

### 3. Start the Docker Services

Start both MediaMTX (RTSP server) and BirdNet-Go:

```bash
docker-compose up --build
```

This will start:
- **MediaMTX** - RTSP server on port 8554
- **BirdNet-Go** - Bird sound analyzer on port 8080

### 4. Stream Your Microphone to MediaMTX

In a separate terminal window, run this FFmpeg command to capture your microphone and stream it to MediaMTX:

```bash
ffmpeg -f avfoundation -i ":0" -acodec aac -b:a 128k -ar 44100 -ac 1 -f rtsp -rtsp_transport tcp rtsp://localhost:8554/mic
```

**What this command does:**
- `-f avfoundation` - Use macOS's AVFoundation framework to capture audio
- `-i ":0"` - Input from audio device index 0 (usually built-in microphone)
- `-acodec aac` - Encode audio using AAC codec
- `-b:a 128k` - Audio bitrate of 128 kbps
- `-ar 44100` - Sample rate of 44.1 kHz
- `-ac 1` - Mono audio (1 channel)
- `-f rtsp -rtsp_transport tcp` - Stream via RTSP using TCP transport
- `rtsp://localhost:8554/mic` - Stream to MediaMTX at path `/mic`

**Using an External Microphone:**

If you want to use an external USB microphone or audio interface instead of your built-in mic, first list your available audio devices:

```bash
ffmpeg -f avfoundation -list_devices true -i ""
```

This will show output like:
```
[AVFoundation indev @ 0x...] AVFoundation audio devices:
[AVFoundation indev @ 0x...] [0] MacBook Pro Microphone
[AVFoundation indev @ 0x...] [1] External USB Microphone
[AVFoundation indev @ 0x...] [2] Blue Yeti
```

Then use either the device index or name in your FFmpeg command:

**Option 1: Using device index**
```bash
ffmpeg -f avfoundation -i ":1" -acodec aac -b:a 128k -ar 44100 -ac 1 -f rtsp -rtsp_transport tcp rtsp://localhost:8554/mic
```

**Option 2: Using device name (recommended - more reliable)**
```bash
ffmpeg -f avfoundation -i ":External USB Microphone" -acodec aac -b:a 128k -ar 44100 -ac 1 -f rtsp -rtsp_transport tcp rtsp://localhost:8554/mic
```

Using the device name is recommended because it won't change if you plug/unplug other devices.

### 5. Configure Your `.env` File

Your `.env` file should be configured to use the MediaMTX RTSP stream:

```ini
# .env
BIRDNET_COMMAND="birdnet-go realtime --rtsp 'rtsp://host.docker.internal:8554/mic'"
```

The `host.docker.internal` hostname allows the Docker container to connect to MediaMTX running on your Mac.

### 6. Access the Web Interface

Once everything is running, you can access the BirdNet-Go web interface at `http://localhost:8080` to see bird detections in real-time!