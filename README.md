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
## Changes Over Upstream
| File | Change |
|---|---|
| `build/boards/sg200x/sg2002_licheervnano_sd/memmap.py` | Freed bootlogo ION memory — required for 1920x1440 VB pool |
| `buildroot/board/cvitek/SG200X/overlay/etc/profile.d/middleware.sh` | Added `libs_patch` paths to `LD_LIBRARY_PATH` |
| `buildroot/board/cvitek/SG200X/overlay/root/` | Camera binaries and `libs_patch` middleware blobs |
| `buildroot/board/cvitek/SG200X/overlay/etc/init.d/S99audio` | Sets DAC volume to 0 (max) on boot — cv182xa DAC register defaults to 16 (50% attenuation) at power-on |
| `ramdisk/rootfs/overlay/musl_riscv64/system/auto.sh` | Added `libs_patch` paths to `LD_LIBRARY_PATH` |
---
## Firmware Changelog

### 2025-03-05 — Audio: DAC volume init fix
**File:** `buildroot/board/cvitek/SG200X/overlay/etc/init.d/S99audio`
**Reason:** cv182xa DAC uses an attenuation register (0 = max, 32 = silent) that hard-resets to 16 (50% attenuation) on every power cycle at the silicon level. `alsactl store` cannot override this as the driver reinitializes the register before the store captures the state.
**Fix:** Added `S99audio` init script that runs last in boot sequence, sets DAC to 0, and stores the state so subsequent `alsactl restore` calls pick up the correct value.

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
