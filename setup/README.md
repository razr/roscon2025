# 🛠 Demo Setup Instructions

This document describes the **setup required** to run the ⋮⋮⋮ROS 2 <img src="https://www.windriver.com/sites/default/files/2023-08/vxworks.png" width="16"/>VxWorks demo on an Ubuntu 22.04 host machine.

---

## 🖥 Host Requirements

- **OS:** <img src="https://assets.ubuntu.com/v1/29985a98-ubuntu-logo32.png" width="16"/> Ubuntu 22.04 LTS
- **Tools:** 🐳Docker, Minikube, <img src="https://images.icon-icons.com/2699/PNG/512/qemu_logo_icon_169821.png" width="16"/> QEMU

---

## 1️⃣ Install Docker

Install Docker on your Ubuntu host:

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

Verify installation:

```bash
docker --version
```

## 2️⃣ Install Minikube

Follow these steps to install Minikube:

```bash
# Download latest Minikube binary
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify
minikube version
```

Start Minikube with a bridge network:

```bash
minikube start
```

## 3️⃣ Install QEMU

Install QEMU to run VxWorks:

```bash
sudo apt update
sudo apt install -y qemu-system-x86
```

Verify installation:

```bash
qemu-system-x86_64 --version
```

## 4️⃣ Network Setup

Two tap interfaces are required for VxWorks to communicate with the host and Minikube network.

```bash
Create tap0 and tap1
sudo ip tuntap add dev tap0 mode tap
sudo ip link set tap0 up

sudo ip tuntap add dev tap1 mode tap
sudo ip link set tap1 up
```

⚡ Ensure these networks do not conflict with existing host networks.

## ✅ Summary

After completing this setup:
- 🐳Docker is installed and ready for containerized ⋮⋮⋮ROS 2 nodes.
- Minikube is running for ☸Kubernetes orchestration.
- QEMU is installed to run VxWorks.
- `tap` interfaces are configured for VxWorks VM networking.

You can now deploy Linux ⋮⋮⋮ROS 2 pods and VxWorks ⋮⋮⋮ROS 2 nodes as described in the demo.
