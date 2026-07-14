# <img src="https://github.com/user-attachments/assets/cec4215f-1cdb-478f-8814-6add4b768666" width="100%" alt="RadiaSight Banner" />

<div align="center">

[![Documentation](https://img.shields.io/badge/docs-rimaspec-007EC6?style=for-the-badge&logo=read-the-docs&logoColor=white)](https://docs.rimaspec.upc.edu/books/radiasight)
[![YouTube Playlist](https://img.shields.io/badge/youtube-playlist-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://youtube.com/playlist?list=PLLIsosutcpWc&si=-Xz_bwcLZ8HVY8PK)

[![C++ 20](https://img.shields.io/badge/C++-20-00599C?style=flat-square&logo=c%2B%2B&logoColor=white)](https://cppreference.com/)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9-7F52FF?style=flat-square&logo=kotlin&logoColor=white)](https://kotlinlang.org/)
[![Vulkan](https://img.shields.io/badge/Vulkan-1.3-E6522C?style=flat-square&logo=vulkan&logoColor=white)](https://www.vulkan.org/)
[![Zenoh](https://img.shields.io/badge/Protocol-Zenoh-00A4A6?style=flat-square)](https://zenoh.io/)
[![Android](https://img.shields.io/badge/Platform-Android-3DDC84?style=flat-square&logo=android&logoColor=white)](https://developer.android.com/)

**RadiaSight** is a real-time 3D radiation mapping ecosystem. It synchronizes spatial sensor feeds (LiDAR, SLAM, pose odometry) and nuclear detector readings (counts, dosage, spectrum) over a high-performance publish-subscribe network to reconstruct colorized radiation field point clouds.

</div>

---

## Architecture Flowchart

```mermaid
```mermaid
flowchart LR
    %% Devices
    subgraph JETSON ["Jetson Orin Nano Super"]
        direction TB
        LIDAR["Livox MID360 LiDAR + IMU"]
        RC_DEV["Radiacode 10x (USB)"]

        GLIM["GLIM SLAM Node"]
        RC_NODE["radiacode_node (Python)"]
        BRIDGE["zenoh_bridge_node (C++)"]

        LIDAR -->|Datos crudos| GLIM
        RC_DEV -->|Paquetes USB| RC_NODE

        GLIM -->|/glim_ros/pose_corrected<br/>/glim_ros/aligned_points_corrected| BRIDGE
        RC_NODE -->|/radiacode/spectrum<br/>/radiacode/dose_rate| BRIDGE
    end

    subgraph PI5 ["Raspberry Pi 5 (GCS)"]
        direction TB
        ZROUTER["Zenoh Router"]
        ZSTORAGE["Zenoh Storage<br/>plugin de grabacion"]
        DISK["Almacenamiento local<br/>misiones grabadas"]

        ZROUTER -->|Todos los topics, en vivo| ZSTORAGE
        ZSTORAGE -->|Lectura y escritura| DISK
    end

    subgraph TABLET ["Android Tablet"]
        CLIENT["ZenohClient (Kotlin)"]
        RENDER["Vulkan Renderer"]
        PREFS["SharedPreferences<br/>solo config y preferencias"]

        CLIENT -->|Parsed Message structs| RENDER
        CLIENT -->|URL servidor, calibracion, ajustes UI| PREFS
    end

    subgraph QUEST3 ["Meta Quest 3 (futuro)"]
        QCLIENT["Cliente Zenoh / Visor VR"]
    end

    %% Network Connections
    BRIDGE <==>|Enlace por radio Jetson a Pi5<br/>Zenoh TCP-UDP| ZROUTER
    ZROUTER <==>|Enlace por USB-Tether Pi5 a Tablet<br/>Zenoh TCP-UDP | CLIENT
    ZROUTER <==>|Hotspot Wi-Fi de la Pi5<br/>Zenoh TCP-UDP| QCLIENT
