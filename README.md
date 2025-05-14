# OpenIPC-G2G

## Overview

The **OpenIPC-G2G** project is designed to measure and optimize the **glass-to-glass latency** in openipc system. Currently only tested with **openipc thinker** + **radxa 3W**.
Glass-to-glass latency refers to the time taken from video acquisition on the sensor side to its display on the screen. This repository provides tools, scripts, and modifications to install it.

---

## Current Status

The project is currently in **alpha** stage. It includes experimental features and may contain bugs or limitations. Use it with caution and report any issues to help improve the project.

---

## Installation

### Ground Station Setup

1. Adapt `gs/etc/wifibroadcast.cfg.g2g` to your need
2. Copy the contents of the `gs` directory to the ground station and activate `g2g` mode:
   ```bash
   cd gs
   scp -r config/ etc/ usr/ root@<gs.ip>:/
   ssh root@<gs.ip> g2g enable
   ```

### Drone Setup
1. Adapt `air/etc/divinus.yaml`, `air/etc/rc.local`, `air/etc/vtxmenu.ini` and `air/etc/wfb.yaml` to your need
2. Copy the contents of the `air` directory to the drone and activate `g2g` mode:
   ```bash
   cd air
   scp -r -O etc/ lib/ usr/ root@<drone.ip>:/
   ssh root@<drone.ip> g2g enable
   ```

### Reverting to the Default Installation
To revert to the default installation on the ground station:
```bash
ssh root@<gs.ip> g2g disable
```

To revert to the default installation on the drone:
```bash
ssh root@<drone.ip> g2g disable
```

---

## Technical Details

### Modifications in OpenIPC-G2G:
1. **Divinus Enhancements**:
   - Added support for FPV-specific features.
   - Integrated a kernel module (`sstarts.ko`) to timestamp key stages in the video acquisition and encoding pipeline.

2. **PixelPilot Enhancements**:
   - Removed GStreamer dependency for RTP packetization.
   - Implemented low-latency RTP packetization logic.
   - Added client/server timestamp logic with SEI payload numbering in the H.265 stream.
   - Additionnal WFB in OSD
   - Support skyzone 04X pro 720p100

3. **Timestamping and Latency Measurement**:
   - Added a thread in PixelPilot to monitor `vsync` for display timing.
   - Displayed latency information on the OSD (On-Screen Display).

---

## Limitations

- **Only H.265 is supported**: H.264 is not currently supported.
- **No configurator support**: Configuration changes cannot be made via a configurator (avalonia or windows one)
- **Tunnel disabled**: The tunnel is disabled by default as it interferes with the video stream. Users can re-enable it by modifying `/etc/wifibroadcast.cfg` on the ground station.
- **Transmission offset**: The transfer time via `wfb-ng` has been measured to have a fixed offset of **3ms** for timestamped packets. This offset is added systematically. While OpenIPC-G2G can dynamically determine the transmission latency, enabling this feature currently disrupts the video stream.
- **No msposd rendering on the drone**: The only available rendering is on the ground station.
- **Pipeline dependency**: The pipeline configuration in Divinus fails if `majestic` has not been executed beforehand. This constraint is handled in the `S95divinus` script.

---

## Usage

The `g2g` script is used to toggle between the `g2g` mode and the default mode. It manages configuration files by renaming them with `.ori` (original) and `.g2g` extensions.

### Commands
- **Enable G2G Mode**:
  Activates the `g2g` configuration:
  ```bash
  g2g enable
  ```
- **Disable G2G Mode**:
  Restores the original configuration:
  ```bash
  g2g disable
  ```

### Note
For optimized latency, it is recommended to add the `--disable-vsync` option to PixelPilot. You can modify the PixelPilot launch configuration by editing `/config/scripts/stream.sh` on the ground station.

### Latency Metrics
The OSD provides detailed latency metrics for each stage of the pipeline:
- **Avg**: The glass-to-glass latency (average and maximum).
- **Snr**: The latency from the sensor.
- **Isp**: The latency from the ISP (Image Signal Processor).
- **Enc**: The latency from the VPE (Video Processing Engine) and VENC (Video Encoder).
- **Tra**: The latency for the transfer stage.
- **Dec**: The latency for decoding.
- **Dis**: The latency for display up to the `vsync`.
- **Size**: The size of the image being processed.

---

## Known Bugs and Limitations

- The project is in alpha, and some features may not work as expected.
- Configuration file conflicts may occur if `.ori` or `.g2g` files are manually modified.
- Limited testing has been performed on `openipc thinker` and `radxa 3w`.

---

## Contributing

Contributions are welcome! If you encounter bugs or have suggestions for improvements, feel free to open an issue or submit a pull request.

---

## License

This repository is licensed under the [MIT License](LICENSE).

---

## Screenshot

![OpenIPC-G2G Screenshot](screenshot.jpg)
