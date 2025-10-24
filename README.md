# Real-Time Meets Cloud: ROS 2 on RTOS and Linux with Kubernetes

## Theme: Edge Robotics

## Title: Real-Time Meets Cloud: [ROS 2](https://www.ros.org/) on [VxWorks RTOS](https://www.windriver.com/products/vxworks) and Linux with [Kubernetes](https://www.windriver.com/solutions/learning/what-is-a-kubelet)
## Author: Andrei Kholodnyi, Principal Technologist, Wind River
## Abstract
As robotics systems evolve, the need to integrate real-time capabilities with scalable, cloud-native infrastructure grows significantly. This talk explores how ROS 2 containers can be deployed across both VxWorks RTOS and Linux environments using Kubernetes. We’ll demonstrate a hybrid orchestration approach that leverages a unified container model, showing how OCI-compatible containers can be built for both VxWorks and Linux. This enables seamless management of ROS 2 nodes across heterogeneous platforms. A recorded demo will highlight distributed, containerized robotics applications that span from real-time systems to edge servers, bridging the embedded and cloud-native worlds in modern ROS 2 deployments.

My name is Andrei Kholodnyi, I'm a Principal Technologist at the Office of the CTO, Wind River. I’ve ported [ROS 2 to VxWorks](https://github.com/Wind-River/vxworks7-ros2-build) and a co-chair the [ROS 2 Real-Time Working Group](https://github.com/ros-realtime). At [ROSCon 2025](https://roscon.ros.org/2025/), I’d like to talk about ROS 2, [real-time containers](https://www.windriver.com/containers), and Kubernetes.


## Demo

```mermaid
flowchart TB
%% Host machine
Host["Host Machine\n(Linux OS / Docker)"]

%% Minikube bridge
Bridge["br-f32aaea20b3<br>Minikube bridge<br>192.168.49.1"]

%% Minikube VM (Kubernetes master)
MinikubeVM["Minikube VM<br>IP: 192.168.49.2<br>K8s Master<br>Pods: VxWorks + Linux"]

%% ROS2 Docker containers
ROS2Pub["ROS2 Docker\nLinux Publisher<br>connected to minikube net"]
ROS2Sub["ROS2 Docker\nLinux Subscriber<br>connected to minikube net"]

%% QEMU VXWorks Publisher node (stacked compartments)
subgraph QEMU_Pub_Group["ROS 2 VXWorks Publisher"]
    direction TB
    QEMU_Pub_Core["QEMU VXWorks"]
    QEMU_Pub_Kubelet["vxkubelet"]
    QEMU_Pub_ROS2["ROS2 Container Publisher<br>IP: 192.168.49.4"]

    style QEMU_Pub_Core fill:#8ef,stroke:#333,stroke-width:1px
    style QEMU_Pub_Kubelet fill:#8ef,stroke:#333,stroke-width:1px
    style QEMU_Pub_ROS2 fill:#8ef,stroke:#333,stroke-width:1px
end

%% QEMU VXWorks Subscriber node (stacked compartments)
subgraph QEMU_Sub_Group["ROS 2 VXWorks Subscriber"]
    direction TB
    QEMU_Sub_Core["QEMU VXWorks"]
    QEMU_Sub_Kubelet["vxkubelet"]
    QEMU_Sub_ROS2["ROS2 Container Subscriber<br>IP: 192.168.49.5"]

    style QEMU_Sub_Core fill:#8ef,stroke:#333,stroke-width:1px
    style QEMU_Sub_Kubelet fill:#8ef,stroke:#333,stroke-width:1px
    style QEMU_Sub_ROS2 fill:#8ef,stroke:#333,stroke-width:1px
end

%% Connections
Host --> Bridge
Bridge --> MinikubeVM
Bridge --> ROS2Pub
Bridge --> ROS2Sub
Bridge --> QEMU_Pub_Core
Bridge --> QEMU_Sub_Core

%% Styles
classDef master fill:#f9f,stroke:#333,stroke-width:2px,color:#000
class MinikubeVM master
```
