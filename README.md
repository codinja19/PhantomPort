# PhantomPort 🛡️⚡
> **Autonomous Edge-IoT Hardware Quarantine & Self-Healing Firmware Sentinel**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Hardware-ESP32%20Dual--Core-red.svg)]()
[![Agent Framework](https://img.shields.io/badge/Agent-LangGraph%20%7C%20FastAPI-green.svg)]()
[![Sync](https://img.shields.io/badge/Cloud-Firebase%20Firestore-orange.svg)]()

---

## 📌 Executive Summary
**PhantomPort** is an autonomous sidecar AI security agent designed to protect resource-constrained edge IoT devices from firmware exploits and supply-chain compromises without causing physical operational downtime. 

By offloading anomaly detection to an external watchdog agent via UART/SPI telemetry streaming, PhantomPort continuously inspects microcontroller runtime parameters (heap memory spikes, anomalous GPIO transitions, and unauthorized TCP/DNS requests) with **< 2% onboard overhead**. When a threat or data exfiltration attempt is confirmed, the agent dynamically severs the microcontroller's network interface—air-gapping the device into a "Phantom Port" while keeping physical sensor and actuator loops running uninterrupted. The agent then commands the hardware bootloader to execute an autonomous dual-partition rollback to a cryptographically verified **Golden Firmware (`ota_0`)** image in under 2 seconds.

---

## 🚨 The Edge IoT Security Dilemma
1. **Zero Endpoint Detection (EDR) Capacity:** Edge nodes (ESP32, STM32, PLCs) power critical infrastructure but lack the RAM/CPU overhead to run traditional security software.
2. **Vulnerable OTA Pipelines:** Compromised firmware builds or injected third-party dependencies expose industrial systems to remote hijacking and botnet recruitment.
3. **The "Brick or Breach" Tradeoff:** Current mitigation strategies require remotely cutting power to a compromised node, resulting in catastrophic downtime in smart grids, healthcare monitors, or manufacturing production lines.

---

## ⚙️ System Architecture

```mermaid
graph TD
    subgraph TIER1["Tier 1: Edge Hardware (ESP32)"]
        direction TB
        subgraph CORES["Dual-Core Execution"]
            C1["Core 1: Physical Actuation<br/>(Sensor / Relay Loop)"]
            C0["Core 0: Network & Telemetry<br/>(Wi-Fi / Sockets / JSON Stream)"]
        end
        subgraph FLASH["Dual-Bank Flash Memory"]
            OTA0[("ota_0: Golden Image<br/>(Verified Base)")]
            OTA1[("ota_1: Active Runtime<br/>(Current Execution)")]
        end
    end

    subgraph TIER2["Tier 2: PhantomPort Sentinel Agent (Sidecar)"]
        direction TB
        PARSE["1. Dynamic Telemetry Ingestion<br/>(Heap / Sockets / GPIO)"]
        ISO["2. Isolation Forest Filter<br/>(Statistical Anomaly Scoring)"]
        LG["3. LangGraph Reasoning Engine<br/>(Contextual Exploit Verification)"]
        TRIGGER{"Autonomous Policy Trigger"}
        
        PARSE --> ISO --> LG --> TRIGGER
        TRIGGER -->|Sever Network| CMD1["esp_wifi_stop()<br/>(Air-Gaps External Port)"]
        TRIGGER -->|Rollback Bootloader| CMD2["esp_ota_mark_app_invalid...<br/>(Revert Boot Pointer to ota_0)"]
    end

    subgraph TIER3["Tier 3: Cloud SOC & Forensics"]
        direction TB
        FIREBASE[("Firebase Firestore<br/>(Real-Time State Sync)")]
        DASH["React + Tailwind Dashboard<br/>(Live Operator Command Center)"]
        REPORT["Automated Incident Exporter<br/>(Forensic Log & CVE Diffs)"]
        FIREBASE --> DASH
        FIREBASE --> REPORT
    end

    %% Data Flow Connections
    C0 -->|UART Stream (200ms)| PARSE
    CMD1 -.->|Hardware Interrupt| C0
    CMD2 -.->|Flash Pointer Revert| OTA0
    OTA1 -.->|Active Image Invalidated| CMD2
    LG -->|Real-Time Telemetry & State| FIREBASE

    %% Styling
    classDef default fill:#1e293b,stroke:#475569,stroke-width:1px,color:#f8fafc;
    classDef highlight fill:#0f172a,stroke:#06b6d4,stroke-width:2px,color:#38bdf8;
    classDef alert fill:#450a0a,stroke:#ef4444,stroke-width:1.5px,color:#fca5a5;
    classDef success fill:#064e3b,stroke:#10b981,stroke-width:1.5px,color:#6ee7b7;

    class TIER1,TIER2,TIER3 highlight;
    class CMD1,CMD2 alert;
    class OTA0 success;
```

## 🔄 4-Stage Operational Lifecycle

1. **Continuous Telemetry Streaming (Perception):** The microcontroller broadcasts a compact JSON telemetry frame over UART every 200ms:
   ```json
   {
     "free_heap": 182400,
     "active_sockets": 1,
     "outbound_dst_ip": "192.168.1.10",
     "cpu_load": 14.2,
     "gpio_state": 4
   }

  Dual-Stage Behavioral Reasoning (Cognition):

Fast Filter: A lightweight Isolation Forest flags sudden heap fragmentation drops and socket anomalies.

Agent Evaluation: LangGraph cross-references device identity and active network destinations. If an anomalous shell spawn or outbound exfiltration pattern is identified, it initiates the quarantine workflow.

Dynamic Air-Gapping (Quarantine): The agent signals an immediate network isolation interrupt (esp_wifi_stop()). The device cuts all inbound/outbound wireless communication while the local sensor/actuator tasks on Core 1 continue running uninterrupted.

Autonomous Dual-Partition Rollback (Self-Healing): The agent dispatches a hardware bootloader directive. Using esp_ota_mark_app_invalid_rollback_and_reboot(), the ESP32 invalidates the compromised partition (ota_1), reverts boot pointers to the verified Golden Image (ota_0), and boots into clean state in ~1.8 seconds.

📂 Repository Structure

PhantomPort/
├── firmware/
│   ├── partitions.csv               # Custom A/B partition table (otadata, ota_0, ota_1)
│   ├── golden_firmware_v1.0/        # Stable, verified reference image
│   └── poisoned_firmware_v1.1/      # Simulated supply-chain exploit build for demo
├── agent/
│   ├── main.py                      # FastAPI entry point & serial listener
│   ├── langgraph_sentinel.py        # Autonomous decision state-machine
│   ├── anomaly_detector.py          # Isolation Forest model on heap/socket logs
│   └── serial_bridge.py             # Bi-directional UART communication interface
├── dashboard/
│   ├── src/                         # React + Vite SOC frontend
│   └── firebase_config.js           # Real-time Firestore configuration
├── docs/
│   ├── system_architecture.png      # High-resolution block diagrams
│   └── hackathon_pitch_deck.pdf     # Project presentation deck
├── LICENSE
└── README.md
