# LicheeRV-Nano-Build (0xjba Fork)

Fork of [Sipeed's LicheeRV-Nano-Build](https://github.com/sipeed/LicheeRV-Nano-Build) with additions for 1920x1440 camera support, MJPEG streaming, and AI vision assistant using the GC4653 4MP sensor.

## What This Fork Adds

- **Camera binaries baked into firmware** — `CSICapture`, `CSIStream`, `CSIHiResStream`, `VisionAssistant`
- **Middleware blobs** (`libs_patch`) included in rootfs overlay
- **LD_LIBRARY_PATH** pre-configured in `etc/profile.d/middleware.sh`
- **ION memory fix** — `memmap.py` patched to free bootlogo memory for larger VB pool

For camera source code, build instructions, and usage see:
👉 **[licheerv-nano-camera](https://github.com/0xjba/licheerv-nano-camera)**

---

## Changes Over Upstream

| File | Change |
|---|---|
| `build/boards/sg200x/sg2002_licheervnano_sd/memmap.py` | Freed bootlogo ION memory — required for 1920x1440 VB pool |
| `buildroot/board/cvitek/SG200X/overlay/etc/profile.d/middleware.sh` | Added `libs_patch` paths to `LD_LIBRARY_PATH` |
| `buildroot/board/cvitek/SG200X/overlay/root/` | Camera binaries and `libs_patch` middleware blobs |
| `ramdisk/rootfs/overlay/musl_riscv64/system/auto.sh` | Added `libs_patch` paths to `LD_LIBRARY_PATH` |

---

## Build Instructions

### Prerequisites

- Ubuntu 20.04 / 22.04 host
- LicheeRV Nano W with GC4653 sensor

### 1. Clone and set up

```bash
git clone https://github.com/0xjba/LicheeRV-Nano-Build.git
cd LicheeRV-Nano-Build
```

### 2. Build camera binaries first

Clone and build [licheerv-nano-camera](https://github.com/0xjba/licheerv-nano-camera), then copy binaries into overlay:

```bash
OVERLAY=buildroot/board/cvitek/SG200X/overlay
cp /path/to/licheerv-nano-camera/src/build/{CSICapture,CSIStream,CSIHiResStream,VisionAssistant} $OVERLAY/root/
cp -r /path/to/licheerv-nano-camera/libs_patch $OVERLAY/root/
```

### 3. Enable required packages

```bash
source build/cvisetup.sh
defconfig sg2002_licheervnano_sd
menuconfig
```

Enable under **Target packages**:
- `Libraries > Networking > libcurl`
- `Libraries > Crypto > openssl`
- `Libraries > Compression > zlib`
- `Libraries > Compression > libbrotli`

### 4. Build firmware

```bash
build_all
```

### 5. Flash

Flash the output image to your SD card and boot.

---

## After Code Changes

Whenever camera binaries are rebuilt, update the overlay and rebuild firmware:

```bash
OVERLAY=buildroot/board/cvitek/SG200X/overlay
cp /path/to/licheerv-nano-camera/src/build/{CSICapture,CSIStream,CSIHiResStream,VisionAssistant} $OVERLAY/root/
source build/cvisetup.sh
defconfig sg2002_licheervnano_sd
build_all
```
