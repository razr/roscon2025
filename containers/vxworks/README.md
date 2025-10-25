# 🦾 ROS 2 VxWorks OCI Containers

This directory provides two Dockerfiles that describe ROS 2 example applications — a `minimal publisher` and a `minimal subscriber` — packaged as OCI-compliant containers for VxWorks.
They can be pulled directly from Docker Hub or built with `buildah`. Both containers are ready-to-execute VxWorks RTP (Real-Time Process) images configured for ROS 2 communication.

## 🚀 Quick Start — Use Pre-Built Images

Pre-built images are available under:
👉 https://hub.docker.com/u/razr

To try the publisher example right away:

```bash
# Pull the pre-built ROS 2 VxWorks publisher example
buildah pull docker.io/razr/vxworks:publisher_lambda
```

To run the subscriber example:

```bash
buildah pull docker.io/razr/vxworks:subscriber_lambda
```

## 🧱 Build Images Locally with Buildah

To build the containers from source, first retrieve the ROS 2 environment artifact and mount it:

1. Download the **HDD image qemu humble.zip** artifact from:  
   👉 [https://github.com/Wind-River/vxworks7-ros2-build/actions](https://github.com/Wind-River/vxworks7-ros2-build/actions)

2. Unzip the archive under `~/Downloads`:

```bash
cd ~/Downloads
unzip "HDD image qemu humble.zip"

```bash
git clone https://github.com/razr/roscon2025.git
cd roscon2025/containers/vxworks
```

3. Mount the extracted ros2.img as a loop device:

```bash
sudo mkdir -p ~/Downloads/rootfs
sudo mount -o loop ros2.img ~/Downloads/rootfs
```

4. Build the ROS 2 VxWorks example containers with Buildah, using the mounted filesystem as the build context:

```bash
# Build the publisher example
buildah bud -t publisher_lambda -f Dockerfile.publisher_lambda ~/Downloads/rootfs

# Build the subscriber example
buildah bud -t subscriber_lambda -f Dockerfile.subscriber_lambda ~/Downloads/rootfs
```

5. When done, unmount the image:

```bash
sudo umount ~/Downloads/rootfs
```

## ✅ Verify the Build

List local images

```bash
$ buildah images
REPOSITORY                         TAG      IMAGE ID       CREATED          SIZE
localhost/publisher_lambda         latest   8c24961396dc   11 minutes ago   41.4 MB
localhost/subscriber_lambda        latest   8c24961396dc   11 minutes ago   41.4 MB
```

## 📦 Export as OCI Archive

To export a container image as a portable OCI archive:

```bash
buildah push publisher_lambda oci-archive:publisher_lambda.oci
buildah push publisher_lambda oci-archive:subscriber_lambda.oci
```


## 🏃 Running `subscriber_lambda` from OCI Archive

1. **Copy the OCI archive into an OCI layout directory**  

Use `skopeo` to convert the OCI archive into a directory layout that VxWorks can use:

```bash
skopeo copy oci-archive:subscriber_lambda.oci oci:./subscriber_lambda:latest
```

After this, `./subscriber_lambda/` will contain:

```bash
blobs/
index.json
oci-layout
```

2. Run VxWorks in QEMU

Launch your VxWorks image with QEMU and mount the current directory as a USB storage device:

```bash
sudo qemu-system-x86_64 \
  -machine q35 -cpu Nehalem -smp 8 -m 2G \
  -kernel itl_generic_vip/default/vxWorks \
  -net nic -net tap,ifname=tap0,script=no \
  -display none -serial mon:stdio \
  -append "bootline:gei(0,0)host:vxWorks h=192.168.49.1 e=192.168.49.4 g=192.168.49.1 u=vxworks pw=vxTarget" \
  -usb -device usb-ehci,id=ehci \
  -device usb-storage,drive=fat32 \
  -drive file=fat:rw:`pwd`,id=fat32,format=raw,if=none
```
The current directory (`pwd`) will appear in VxWorks as `/bd0a/`

You should see:

```bash
-> cmd
[vxWorks *]# ls /bd0a/
subscriber_lambda.oci
Dockerfile.publisher_lambda
Dockerfile.subscriber_lambda
subscriber_lambda
publisher_lambda.oci
```

3. Unpack the OCI image inside VxWorks

```bash
[vxWorks *]# vxc unpack --image /bd0a/subscriber_lambda --rootfs flattened /ram0/bundle
```

4. Run the container

```bash
[vxWorks *]# vxc run --detach --bundle /ram0/bundle subscriber_lambda
```

After this, the `subscriber_lambda` container is running on VxWorks.

## 🗂️ Files

| File                        | Description                                     |
|-----------------------------|-------------------------------------------------|
| Dockerfile.publisher_lambda | ROS 2 minimal publisher example for VxWorks     |
| Dockerfile.subscriber_lambda| ROS 2 minimal subscriber example for VxWorks    |
| README.md                   | This document                                   |

## 🧠 Notes

These containers are OCI-compliant and fully compatible with `buildah` and `Podman`.
Each container bundles all required VxWorks runtime and ROS 2 shared libraries.
The labels inside each Dockerfile define VxWorks RTP launch parameters (stack size, priority, etc.).

Mounted volumes (`/ram0`, `/dev/random`, `/dev/urandom`, `/dev/shm`) are configured to mount VxWorks I/O devices and filesystems to the containers.
