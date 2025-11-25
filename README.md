# realsense-pyrealsense2-rpi-base

A Raspberry Pi (ARM64) **base Docker image** that includes:

- Intel RealSense SDK (`librealsense`) **v2.55.1**  
- Python bindings (`pyrealsense2`) built from source and installed into site-packages  
- Python **3.11** on Debian **slim-trixie**
- All required system libraries for USB / UDEV / basic OpenGL

This image includes a full source build of librealsense for the following reasons:

- Intel does **not** provide precompiled ARM64 packages for Raspberry Pi  
- `pyrealsense2` wheels are **not available** for Pi via pip  
- Raspberry Pi kernels require custom patches and USB backend flags  
- Building from source ensures compatibility with  
  - Raspberry Pi OS  
  - Python 3.11 ABI  
  - ARM64 architecture  
  - USB/RSUSB backend required for Pi  
- It guarantees that downstream projects can import RealSense Python bindings
  **without recompiling**.

This base image allows any Python project on a Raspberry Pi to interact with
RealSense cameras **without rebuilding librealsense or pyrealsense2**.

**Tested on:** Raspberry Pi 4 Model B (64-bit Raspberry Pi OS)

---


## Recommended: Add swap on Raspberry Pi for building

Building `librealsense` on a Pi (especially 4 GB models) can be memory-hungry.
Adding a temporary **2 GB swapfile** makes builds much more reliable.

### 1. Check if you already have swap

```bash
swapon --show
free -h
```

If `swapon --show` prints nothing, you have **no swap enabled**.

### 2. Create a 2 GB swapfile

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Verify:

```bash
swapon --show
free -h
```

You should see `/swapfile` listed and **Swap total ~2.0G**.

### 3. Make it persistent across reboots

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Quick test:

```bash
sudo swapoff /swapfile
sudo swapon /swapfile
swapon --show
```

If `/swapfile` shows up again, your swap is working and persistent.

> 💡 You only need to do this once per Pi. It affects the host OS, not the container.

---

## Building locally on the Pi


### Option 1: Direct `docker build`

From the repository root:

```bash
cd src
docker build -f Dockerfile -t realsense-pyrealsense2-rpi-base:2.55.1 .
```

### Option 2: Using `docker compose` (for testing with hardware too)

From the repository root:

```bash
docker compose -f src/docker-compose.yml build
```

This will build the image defined in `src/docker-compose.yml` and tag it as
`realsense-pyrealsense2-rpi-base:2.55.1`

---

## Running & testing on a Pi (with camera attached)

The `src/docker-compose.yml` is configured to:

- run the container as `privileged`
- use `network_mode: host`
- mount:
  - `/dev/bus/usb` (for RealSense)
  - `/dev` (for serial devices, if needed)
  - `/run/udev` (for udev hotplug events)

> These mounts and `privileged` are **only required when running with real hardware**.
> They are **not needed** just to build the image.

From the repository root:

```bash
docker compose -f src/docker-compose.yml up --build -d
```

Then test inside the running container:

```bash
docker exec -it realsense_base python -c "import pyrealsense2 as rs; print(rs.context())"
```

You should see context info instead of an import error.

To stop the container:

```bash
docker compose -f src/docker-compose.yml down
```


## Versioning

- **Image name:** `realsense-pyrealsense2-rpi-base`
- **Base Python image:** `python:3.11-slim-trixie`
- **Target platform:** `linux/arm64` (64-bit Raspberry Pi OS)
- **Librealsense version:** currently `v2.55.1` (tagged as `2.55.1`)



## License

This repo only contains Docker-related configuration. `librealsense` and
`pyrealsense2` are licensed under their respective upstream licenses. Check the
Intel RealSense SDK repository for details.
