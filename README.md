# LicheeRV-Nano-Build (0xjba Fork)
Fork of [Sipeed's LicheeRV-Nano-Build](https://github.com/sipeed/LicheeRV-Nano-Build) with additions for 1920x1440 camera support, MJPEG streaming, and AI vision assistant using the GC4653 4MP sensor.

## What This Fork Adds
- **Camera binaries baked into firmware** — `CSICapture`, `CSIStream`, `CSIHiResStream`, `VisionAssistant`
- **Middleware blobs** (`libs_patch`) included in rootfs overlay
- **LD_LIBRARY_PATH** pre-configured in `etc/profile.d/middleware.sh`
- **ION memory fix** — `memmap.py` patched to free bootlogo memory for larger VB pool
- **Audio volume fix** — DAC initialized to max volume (0) on boot via `S99audio` init script

For camera source code, build instructions, and usage see:
👉 **[licheerv-nano-camera](https://github.com/0xjba/licheerv-nano-camera)**

---

## Firmware Changelog

| Date | File | Change | Reason |
|---|---|---|---|
| 2025-03-04 | `build/boards/sg200x/sg2002_licheervnano_sd/memmap.py` | Freed bootlogo ION memory | Bootlogo reserved a chunk of ION memory at startup; reclaiming it was required to allocate a large enough VB pool for 1920x1440 camera frames |
| 2025-03-04 | `buildroot/board/cvitek/SG200X/overlay/etc/profile.d/middleware.sh` | Added `libs_patch` to `LD_LIBRARY_PATH` | Camera binaries depend on SOPHGO middleware `.so` files not present in the default rootfs; paths must be set before any binary runs |
| 2025-03-04 | `buildroot/board/cvitek/SG200X/overlay/root/` | Added camera binaries (`CSICapture`, `CSIStream`, `CSIHiResStream`, `VisionAssistant`) and `libs_patch` middleware blobs | Bake binaries and dependencies into firmware so the camera stack works out of the box on every flash |
| 2025-03-04 | `ramdisk/rootfs/overlay/musl_riscv64/system/auto.sh` | Added `libs_patch` to `LD_LIBRARY_PATH` | Same as above but for the ramdisk environment which has its own separate profile |
| 2025-03-05 | `buildroot/board/cvitek/SG200X/overlay/etc/init.d/S99audio` | Sets DAC volume to 0 (max) on boot | cv182xa DAC uses an attenuation register (0 = max, 32 = silent) that hard-resets to 16 at the silicon level on every power cycle. `alsactl store` alone cannot fix this as the driver reinitializes the register before the store captures state. `S99audio` runs last in boot sequence, sets DAC to 0, and stores state so `alsactl restore` picks up the correct value |

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
cd ~/LicheeRV-Nano-Build
OVERLAY=buildroot/board/cvitek/SG200X/overlay
cp ~/licheerv-nano-camera/src/build/{CSICapture,CSIStream,CSIHiResStream,VisionAssistant} $OVERLAY/root/
cp -r ~/licheerv-nano-camera/libs_patch $OVERLAY/root/
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
- `Libraries > Compression > libz`

> `libbrotli` is pulled in automatically as a libcurl dependency — no manual selection needed.

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
cd ~/LicheeRV-Nano-Build
OVERLAY=buildroot/board/cvitek/SG200X/overlay
cp ~/licheerv-nano-camera/src/build/{CSICapture,CSIStream,CSIHiResStream,VisionAssistant} $OVERLAY/root/
source build/cvisetup.sh
defconfig sg2002_licheervnano_sd
build_all
```
