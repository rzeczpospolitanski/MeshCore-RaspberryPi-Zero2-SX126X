# MeshCore Installation and Setup Guide

## Official Resources

For the most up-to-date information, visit:
- **MeshCore GitHub**: https://github.com/meshcore-dev/MeshCore
- **MeshCore FAQ**: https://github.com/meshcore-dev/MeshCore/blob/main/docs/faq.md
- **Web Flasher**: https://flasher.meshcore.co.uk/
- **Config Tool**: https://config.meshcore.dev/
- **Discord Community**: https://discord.gg/BMwCtwHj5V

## Prerequisites

- Raspberry Pi Zero 2 W with Raspberry Pi OS Lite installed
- Waveshare SX126X LoRa HAT connected to GPIO pins
- USB power supply (5V/2.5A minimum)
- LoRa antenna attached to HAT
- Serial port enabled on Raspberry Pi

## MeshCore Features

**MeshCore** is a lightweight, portable C++ library that enables:

- Multi-hop packet routing for embedded projects
- Decentralized mesh networks without internet
- Low power consumption (ideal for battery-powered devices)
- Off-grid communication for remote areas
- Support for different device roles (Companion, Repeater, Room Server)

## Installation Method: Web Flasher

The easiest way to install MeshCore is using the Web Flasher:

### Step 1: Prepare Your Device

1. Ensure Raspberry Pi is powered on and has serial connection enabled
2. Antenna must be properly attached to the LoRa HAT
3. Use a USB-to-Serial adapter if connecting from PC

### Step 2: Open Web Flasher

1. Visit https://flasher.meshcore.co.uk/
2. Use Chrome or Edge browser (required for Web Serial API)
3. Click "Select Device"

### Step 3: Choose Your Device & Firmware

**Device Selection:**
- Note: Raspberry Pi devices may require custom setup
- For Pi Zero 2: Consider using compatible ESP32-based devices as alternative
- Alternative: Build firmware manually from source

**Firmware Types:**

| Type | Purpose | Use Case |
|------|---------|----------|
| Companion | Connects to external apps | User devices, mesh nodes |
| Repeater | Relays messages | Network range extension |
| Room Server | Shared message board | Group communication hub |

### Step 4: Flash Firmware

1. Connect serial interface to PC/laptop
2. Click "Flash" button in Web Flasher
3. Select appropriate USB serial port
4. Wait for flashing to complete (2-5 minutes)
5. Device will reboot automatically

## Alternative: Manual Installation from Source

For advanced users wanting to customize MeshCore:

```bash
# Install dependencies
sudo apt install -y git build-essential cmake

# Clone MeshCore repository
git clone https://github.com/meshcore-dev/MeshCore.git
cd MeshCore

# Install PlatformIO (for building)
pip3 install platformio

# Build firmware
pio run -e your_target_board

# Flash to device
pio run -e your_target_board -t upload
```

## Post-Installation Configuration

### Using Web Config Tool

For Repeater and Room Server firmware:

1. Visit https://config.meshcore.dev/
2. Connect device via USB serial
3. Configure:
   - Device name
   - Network settings
   - Repeater address ranges (for Repeater mode)
   - Security settings

### Using Mobile Apps

For Companion Firmware:

**Mobile Apps:**
- **Android**: [MeshCore on Google Play](https://play.google.com/store/apps/details?id=com.liamcottle.meshcore.android)
- **iOS**: [MeshCore on App Store](https://apps.apple.com/us/app/meshcore/id6742354151)
- **Web**: [MeshCore Web App](https://app.meshcore.nz/)

**Connection Methods:**
- Bluetooth Low Energy (BLE)
- USB Serial
- WiFi (if device supports it)

## Network Configuration

### Setting Device Address

Each node needs a unique address (0-255):

```bash
# Via serial console
ADDRESS=1  # Change to unique number
NET_ID=0   # Network ID (optional)
```

### Frequency Selection

| Region | Recommended Frequency | Variant |
|--------|----------------------|----------|
| Europe | 868 MHz | SX1262 868M |
| Americas | 915 MHz | SX1262 915M |
| China/Asia | 433/470 MHz | SX1262 433/470 |

## Testing Your Installation

### Test 1: Check Serial Connection

```bash
ls -l /dev/ttyS0  # Raspberry Pi hardware UART
```

### Test 2: Verify LoRa Module

Using Waveshare demo code:

```bash
cd ~/Documents/SX126X_LoRa_HAT_Code/raspberrypi/python
sudo python3 main.py
```

Press 'i' and send test message

### Test 3: Two-Device Mesh

1. Flash two devices with Companion firmware
2. Connect both to MeshCore app
3. Send message from Device 1 → Device 2
4. Verify successful delivery

## Power Consumption Reference

| State | Current |
|-------|----------|
| Transmitting | ~100 mA |
| Receiving | ~11 mA |
| Sleep | ~2 µA |

## Troubleshooting

### Web Flasher Not Working

1. Ensure browser is Chrome or Edge (Chromium-based)
2. Check USB serial port is correctly selected
3. Try different USB cable
4. Check device manager for COM port conflicts

### Serial Connection Failed

1. Verify `/dev/ttyS0` permissions:
   ```bash
   sudo usermod -a -G dialout $USER
   ```
2. Check serial port is enabled in raspi-config
3. Test with USB-to-Serial adapter

### No Network Connectivity

1. Verify M0/M1 mode pins are set correctly
2. Check antenna is properly attached
3. Ensure both devices on same frequency
4. Test with nearby device first

## Advanced Configuration

### Custom Build Parameters

Edit configuration in MeshCore source:

```cpp
// Maximum hop count
#define MAX_HOPS 3

// Buffer size for messages
#define MESSAGE_BUFFER_SIZE 240

// Transmission power (dBm)
#define TX_POWER 22
```

### Network Topology

**Star Network:**
- Central repeater
- Multiple companion nodes
- Good for one-way communication

**Mesh Network:**
- All nodes equal
- Multi-path routing
- Better resilience

## Resources

- [MeshCore GitHub](https://github.com/meshcore-dev/MeshCore)
- [MeshCore Discord](https://discord.gg/BMwCtwHj5V)
- [Semtech LoRa Specifications](https://www.semtech.com/)
- [Raspberry Pi GPIO Reference](https://pinout.xyz/)

## Additional Notes

- MeshCore is **NOT** LoRaWAN compatible (uses proprietary protocol)
- Uses ISM frequencies (license-free in most countries)
- Range depends on antenna, power, and environment
- Repeater mode extends coverage significantly

## Support & Community

If you need help:

1. Check MeshCore FAQ: https://github.com/meshcore-dev/MeshCore/blob/main/docs/faq.md
2. Join Discord: https://discord.gg/BMwCtwHj5V
3. Check GitHub Issues: https://github.com/meshcore-dev/MeshCore/issues
4. Search existing documentation

---

**Last Updated**: January 2, 2026
**MeshCore Version**: Latest (check GitHub releases)
**Status**: Community-maintained guide
