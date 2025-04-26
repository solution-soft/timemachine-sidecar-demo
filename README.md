# SolutionSoft Time Machine Sidecar Demo

This repository provides a demonstration of SolutionSoft's Time Machine product using a Docker Compose setup with a sidecar pattern.

Time Machine is a software product that provides virtual clocks to applications running in a Linux environment. It works by preloading a shared library into the application's process space, intercepting system calls related to time.

In this demonstration, two containers are managed by Docker Compose:
1.  **`sidecar`**: Runs the Time Machine agent (`tmagent`) which manages the virtual clocks.
2.  **`java`**: Runs a simple Spring Boot application (`tmdemo`) that displays the current time.

The `java` container is configured to depend on the `sidecar` container, ensuring the Time Machine agent is available before the application starts.

## Prerequisites

*   Docker
*   Docker Compose

## Getting Started

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/solution-soft/timemachine-sidecar-demo.git
    cd timemachine-sidecar-demo
    ```

2.  **Configure Licensing:**
    The `sidecar` container requires a valid Time Machine license to run. Licensing is configured via environment variables.
    *   Copy the sample environment file:
        ```bash
        cp sample.env .env
        ```
    *   Edit the newly created `.env` file and replace the placeholder values for `TM_LICHOST`, `TM_LICPORT`, and `TM_LICPASS` with your actual Time Machine license server details.

    ```ini
    # .env file - Replace with your license details
    TM_LICHOST=your_license_server_host
    TM_LICPORT=your_license_server_port
    TM_LICPASS=your_license_password
    ```

3.  **Start the containers:**
    Run the following command in the root directory of the repository:
    ```bash
    docker compose up -d
    ```
    This will build (if necessary) and start both the `sidecar` and `java` containers in detached mode.

## How to Use

Once the containers are up and running, you can interact with the Time Machine agent and the demo application.

1.  **Check the initial time:**
    Access the Spring Boot application running in the `java` container via port 8080.
    ```bash
    curl "http://localhost:8080/tmdemo/gettime"
    ```
    This will show the current system time as seen by the application.

2.  **Set a virtual clock:**
    Interact with the Time Machine agent (`tmagent`) running in the `sidecar` container via port 7800.
    ```bash
    curl "http://localhost:7800/tmapp/addrelclock?procid=-1&year=5"
    ```
    This command sets a relative virtual clock for *all* processes (`procid=-1`) managed by this agent, advancing the perceived year by 5 years.

3.  **Verify the clock change:**
    Check the time from the Spring Boot application again.
    ```bash
    curl "http://localhost:8080/tmdemo/gettime"
    ```
    You should now see the time reported by the application is 5 years in the future compared to the initial time.

## Directory Structure

*   `ssstm/`: Contains files related to Time Machine persistence (e.g., configuration).
*   `app/`: Contains the Java JAR file (`tmdemo-0.0.1-SNAPSHOT.jar`) for the Spring Boot application.
*   `sample.env`: A template file for licensing configuration.
*   `docker-compose.yml`: Defines the services, networks, and volumes for the Docker containers.

## Licensing Note

The `sidecar` container running the Time Machine agent requires a valid license to function. Ensure the `.env` file is correctly configured with your license server details before starting the containers.
