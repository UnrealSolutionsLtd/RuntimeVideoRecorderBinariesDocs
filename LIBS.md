### Building Third-Party Libraries

This repository includes pre-compiled libraries (x264, lsmash, mp4v2, libmpv) for various platforms.

#### Building libmpv for Linux

The repository includes a GitHub Actions workflow to build libmpv for both Linux x64 and ARM64 architectures using the [mpv-build](https://github.com/mpv-player/mpv-build) repository.

##### Using GitHub Actions (Recommended)
1. Go to the **Actions** tab in your GitHub repository
2. Select the **Build libmpv for Linux** workflow
3. Click **Run workflow**
4. (Optional) Specify MPV and FFmpeg versions:
   - `master` - Latest development version (default)
   - `release` - Latest stable release
   - Custom tag/branch/commit (e.g., `v0.38.0`)
5. Download the artifacts once the build completes

The workflow will produce:
- `libmpv-linux-x64` - x64 libraries and headers
- `libmpv-linux-arm64` - ARM64 libraries and headers  
- `libmpv-linux-all-architectures` - Combined package with build info

##### Manual Build for Linux

**For x64:**
```bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y \
  build-essential git yasm nasm cmake ninja-build \
  autoconf automake libtool pkg-config \
  libx11-dev libxext-dev libxrandr-dev libxinerama-dev \
  libxcursor-dev libxi-dev libxss-dev libxv-dev \
  libvdpau-dev libva-dev libgl1-mesa-dev \
  libasound2-dev libpulse-dev \
  libfribidi-dev libfreetype6-dev libfontconfig1-dev \
  libharfbuzz-dev libjpeg-dev libssl-dev \
  python3 python3-pip

# Upgrade meson (Ubuntu 22.04 has 0.61.2, but libplacebo requires >= 0.63)
sudo pip3 install --upgrade meson

# Clone and build
git clone https://github.com/mpv-player/mpv-build.git
cd mpv-build

# Configure versions (optional)
./use-mpv-master    # or ./use-mpv-release
./use-ffmpeg-master # or ./use-ffmpeg-release

# Configure options (hardware acceleration only)
printf "%s\n" -Dlibmpv=true >> mpv_options
printf "%s\n" --enable-vdpau >> ffmpeg_options
printf "%s\n" --enable-vaapi >> ffmpeg_options

# Build
./rebuild -j$(nproc)

# Libraries will be in mpv-build/mpv/build/
```

**For ARM64 (using Docker with QEMU):**
```bash
# Set up QEMU for ARM64 emulation
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Build in ARM64 container
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  -w /build \
  arm64v8/ubuntu:22.04 \
  bash -c "
    apt-get update && \
    apt-get install -y build-essential git yasm nasm cmake \
      ninja-build autoconf automake libtool pkg-config \
      libx11-dev libxext-dev libfribidi-dev libfreetype6-dev \
      libfontconfig1-dev libharfbuzz-dev libjpeg-dev libssl-dev \
      python3 python3-pip && \
    pip3 install --upgrade meson && \
    git clone https://github.com/mpv-player/mpv-build.git && \
    cd mpv-build && \
    ./use-mpv-master && \
    printf '%s\n' -Dlibmpv=true >> mpv_options && \
    printf '%s\n' --enable-vdpau >> ffmpeg_options && \
    printf '%s\n' --enable-vaapi >> ffmpeg_options && \
    ./rebuild -j\$(nproc) && \
    mkdir -p /workspace/Linux/arm64/lib && \
    find . -name 'libmpv.so*' -exec cp -P {} /workspace/Linux/arm64/lib/ \;
  "
```

#### Building MP4v2 for Linux ARM64

##### Using Docker with QEMU for cross-compilation
```bash
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  -w /build \
  arm64v8/ubuntu:22.04 \
  bash -c "
    apt-get update && \
    apt-get install -y build-essential cmake git autoconf automake libtool && \
    git clone https://github.com/enzo1982/mp4v2.git && \
    cd mp4v2 && \
    mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF && \
    make -j\$(nproc) && \
    cp libmp4v2.a /workspace/Linux/arm64/
  "
```

The GitHub workflows support:
- **libmpv** - Linux x64 and ARM64 with hardware acceleration (VDPAU, VAAPI) for video playback
- **MP4v2** - Linux ARM64 using QEMU-based cross-compilation

---
