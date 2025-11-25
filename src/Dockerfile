# RealSense Python base image for Raspberry Pi (ARM64)
# Contains librealsense + pyrealsense2 installed into site-packages.

FROM python:3.11-slim-trixie

ENV PIP_BREAK_SYSTEM_PACKAGES=1
ENV PIP_EXTRA_INDEX_URL=https://www.piwheels.org/simple
ENV PIP_PREFER_BINARY=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    git cmake build-essential pkg-config \
    libssl-dev libusb-1.0-0-dev \
    libgl1-mesa-dev libglu1-mesa-dev \
    libgtk-3-dev libglfw3-dev \
    libtbb-dev libopenblas-dev \
    python3-dev python3-pip \
    udev curl \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

ARG LIBREALSENSE_TAG=v2.55.1
RUN git clone --branch ${LIBREALSENSE_TAG} --depth 1 https://github.com/IntelRealSense/librealsense.git

# --------------------------------------------------------------------
# Build librealsense with python bindings enabled
# --------------------------------------------------------------------
WORKDIR /opt/librealsense
RUN rm -rf build && mkdir -p build && cd build && \
    cmake .. \
      -DFORCE_RSUSB_BACKEND=ON \
      -DBUILD_PYTHON_BINDINGS=ON \
      -DBUILD_PYTHON=ON \
      -DBUILD_PYTHON3=ON \
      -DPYTHON_EXECUTABLE=/usr/local/bin/python3 \
      -DPYTHON_INCLUDE_DIR=/usr/local/include/python3.11 \
      -DPYTHON_LIBRARY=/usr/local/lib/libpython3.11.so \
      -DBUILD_EXAMPLES=OFF \
      -DBUILD_GRAPHICAL_EXAMPLES=OFF \
      -DBUILD_TOOLS=OFF \
      -DCMAKE_BUILD_TYPE=Release && \
    cmake --build . -j"$(nproc)" && \
    cmake --build . --target install -j"$(nproc)" && \
    ldconfig

# udev rules (optional)
RUN cp /opt/librealsense/config/99-realsense-libusb.rules /etc/udev/rules.d/ || true

# --------------------------------------------------------------------
# Locate and install the pyrealsense2 Python extension (.so)
#
# Why dynamic search?
# - The output path and filename of pyrealsense2*.so are NOT stable.
# - They vary across:
#     * Python versions (3.11 ABI tag, CPython version)
#     * CPU arch (armv7l vs aarch64)
#     * Pi OS base image (Debian Bookworm/Trixie)
#     * CMake generator (Debug/Release folder structure)
#     * Librealsense version (different output directories)
# - Sometimes the .so stays under /opt/librealsense/build/,
#   other times it gets installed directly into site-packages/.
#
# This block:
# 1. Searches the *entire* librealsense tree for pyrealsense2*.so.
# 2. If not found, searches probable Python installation paths.
# 3. Copies the Python wrapper folder + .so into site-packages.
#
# This approach is intentionally defensive and avoids hardcoding
# a fragile path that would break whenever Python, Pi OS, arch,
# or librealsense is updated.
# --------------------------------------------------------------------
RUN set -eux; \
    SO_FILE="$(find /opt/librealsense/build -name 'pyrealsense2*.so' -print -quit)"; \
    echo "Found pyrealsense2 SO: ${SO_FILE}"; \
    test -n "${SO_FILE}"; \
    PY_SITE="$(python -c 'import site; print(site.getsitepackages()[0])')"; \
    echo "Python site-packages: ${PY_SITE}"; \
    rm -rf "${PY_SITE}/pyrealsense2"; \
    cp -r /opt/librealsense/wrappers/python/pyrealsense2 "${PY_SITE}/"; \
    cp "${SO_FILE}" "${PY_SITE}/pyrealsense2/";

# Default to python shell
WORKDIR /root
CMD ["python"]
