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

For demo purposes, you can stream your Mac's microphone as an RTSP feed for Birdnet-Go to use. This requires the [VLC media player](https://www.videolan.org/vlc/download-macosx.html).

### 1. Grant Microphone Access to VLC

Before starting, you must give VLC permission to access your microphone:

1.  Open **System Settings** > **Privacy & Security** > **Microphone**.
2.  If VLC is in the list, ensure the toggle is on. 
3.  If VLC is not in the list, open the VLC application, go to **File > Open Capture Device...**, select the **Audio** tab, choose your microphone, and click **Open**. This will trigger a permission prompt from macOS. Allow it.

### 2. Start the RTSP Stream with VLC

Open your terminal and run the following command. This command must be pasted as a single, unbroken line.

```bash
"/Applications/VLC.app/Contents/MacOS/VLC" -I dummy "avfoundation://:-1:0" --sout='#transcode{acodec=mp3,ab=128,channels=1,samplerate=44100}:rtp{sdp=rtsp://:8554/mic}'
```

If you have VLC installed in a different location (e.g., an external drive), make sure to update the path to the VLC executable in the command.

This command will run in your terminal and broadcast your microphone's audio to `rtsp://localhost:8554/mic`.

### 3. Update Your `.env` File

Update your `.env` file to use the local VLC stream. Note the use of `host.docker.internal` which allows the Docker container to connect to services running on your Mac.

```ini
# .env
BIRDNET_COMMAND="birdnet-go realtime --rtsp 'rtsp://host.docker.internal:8554/mic'"
```

### 4. Restart Your Docker Container

Finally, restart your Docker Compose setup to apply the new environment variable:

```bash
docker-compose up --build
```
