# SDRGateway

<p align="center">
  <img src="images/raspberry-pi-3ap.jpeg" alt="My Raspberry Pi" width="600"><br>
  <em>Figure 1: My Raspberry Pi 3A+ setup running headless.</em>
</p>

## System Architecture

```mermaid
graph TD
    subgraph Sensors [Field Layer]
        WS[Weather Station Sensors<br/>433.92/868 MHz RF]
    end

    subgraph Edge_Gateway [Raspberry Pi 3A+ Gateway Layer]
        SDR[RTL-SDR USB Dongle]
        RTL[rtl_433 Utility<br/>Demodulator]
        NR[Node-RED Flow<br/>Orchestrator]

        WS -- Wireless Broadcast --> SDR
        SDR -- Raw I/Q Stream --> RTL
        RTL -- Structured JSON --> NR
    end

    subgraph Local_Storage [On-Premises Infrastructure]
        NAS[Synology NAS Host]
        subgraph Docker_Engine [Container Space]
            MQ[Eclipse Mosquitto Broker]
        end
        NAS --- Docker_Engine
    end

    subgraph Cloud_Storage [Cloud Infrastructure]
        AZ[Azure IoT Hub]
    end

    %% Data Pipeline Paths
    NR -- "MQTT (Port 1883)" --> MQ
    NR -- "MQTT over TLS (Port 8883)" --> AZ

    %% High-Contrast Styling Realignment
    style WS fill:#ff66cc,stroke:#222,stroke-width:2px,color:#000000
    style SDR fill:#99ccff,stroke:#222,stroke-width:2px,color:#000000
    style RTL fill:#333333,stroke:#ccc,stroke-width:2px,color:#ffffff
    style NR fill:#ff9999,stroke:#222,stroke-width:2px,color:#000000
    style NAS fill:#444444,stroke:#ccc,stroke-width:1px,color:#ffffff
    style MQ fill:#99ff99,stroke:#222,stroke-width:2px,color:#000000
    style AZ fill:#ffff99,stroke:#222,stroke-width:2px,color:#000000

```

### Data Pipeline & Architecture Layers

1. **Physical / RF Layer:** Wireless outdoor weather sensors broadcast raw radio frequencies.
2. **SDR Hardware Ingestion:** An RTL-SDR USB dongle attached to the Raspberry Pi 3A+ intercepts the 433/868MHz signals.
3. **Demodulation Layer:** The `rtl_433` tool runs on Raspberry Pi OS, decoding the radio signals into structured JSON data.
4. **Edge Orchestration:** Node-RED ingests the JSON payload, parses the telemetry, and splits the data stream into two pathways:
   * **Local Storage:** Dispatched to a containerised Eclipse Mosquitto broker on a Synology NAS via MQTT.
   * **Cloud Storage:** Secured and dispatched to Azure IoT Hub for enterprise cloud processing.

