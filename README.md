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
