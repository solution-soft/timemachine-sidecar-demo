# SolutionSoft Time Machine Sidecar Demo

This repository provides a demonstration of SolutionSoft's Time Machine product using a Docker Compose setup with a sidecar pattern.

**Time Machine for containers** is a software product that provides virtual clocks to applications running in a Linux environment. It works by preloading a shared library into the application's process space, intercepting system calls related to time.

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

1.  **Check Time Machine status:**
    Access the Time Machine agent (`tmagent`) running in the `sidecar` container via port 7800.
    ```bash
    curl "http://localhost:7800/tmapp/getstatus"
    ```
    Expected Output (may vary slightly):
    ```json
    [{"Licensed":"yes"},{"Floating":"yes"},{"Suspended":"no"},{"Version":"18.03"},{"Release":"76"},{"Status":"Running"},{"TMVersion":"18.03R76"},{"Timezone":"Etc/UTC"},{"System":"linux"},{"Hostname":"5d3c9f5a3ad1"},{"CPUCount":1},{"SystemID":""}]
    ```

2.  **Check current timestamp from java container:**
    Access the Spring Boot application running in the `java` container via port 8080.
    ```bash
    curl http://localhost:8080/tmdemo/gettime
    ```
    Expected Output (timestamp will vary):
    ```json
    {"seq":1,"pid":1,"username":"www-data","hostname":"37d0de6bcab8","timestamp":"2025-04-26 00:40:38 UTC"}
    ```
    This shows the current system time as seen by the application *before* any virtual clock is applied.

3.  **Set a virtual clock:**
    Use the Time Machine agent API to set a relative virtual clock.
    ```bash
    curl "http://localhost:7800/tmapp/addrelclock?procid=-1&year=5"
    ```
    This command sets a relative virtual clock for *all* processes (`procid=-1`) managed by this agent, advancing the perceived year by 5 years.
    Expected Output (values will vary, especially timestamp):
    ```json
    {"Floating":"yes","Licensed":"yes","Suspended":"no","cmds":[],"day":0,"gids":[],"hour":0,"minute":0,"origin":1745628068,"persist":"False","platform":"linux","procids":["@all"],"relative":1,"speed":1,"timestamp":"20300426004108","uids":[],"year":5}
    ```

4.  **Verify the clock change:**
    Check the time from the Spring Boot application again.
    ```bash
    curl http://localhost:8080/tmdemo/gettime
    ```
    Expected Output (timestamp will be approximately 5 years ahead of the previous output):
    ```json
    {"seq":2,"pid":1,"username":"www-data","hostname":"37d0de6bcab8","timestamp":"2030-04-26 00:41:48 UTC"}
    ```
    You should now see the time reported by the application is 5 years in the future compared to the initial time.

## Directory Structure

*   `ssstm/`: Contains files related to Time Machine persistence (e.g., configuration).
*   `app/`: Contains the Java JAR file (`tmdemo-0.0.1-SNAPSHOT.jar`) for the Spring Boot application.
*   `sample.env`: A template file for licensing configuration.
*   `docker-compose.yml`: Defines the services, networks, and volumes for the Docker containers.

## Licensing Note

The `sidecar` container running the Time Machine agent requires a valid license to function. Ensure the `.env` file is correctly configured with your license server details before starting the containers.
