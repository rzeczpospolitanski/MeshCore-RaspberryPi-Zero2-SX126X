# MeshCore on Raspberry Pi Zero 2 with Waveshare SX126X LoRa HAT

Complete setup guide and resources for installing MeshCore on Raspberry Pi Zero 2 with Waveshare SX126X LoRa HAT. This repository contains all necessary documentation, configuration files, and example code for building an off-grid mesh networking system.

## Project Overview

This project provides a comprehensive setup guide for:
- **MeshCore**: A lightweight, portable C++ library for multi-hop packet routing with LoRa
- **Raspberry Pi Zero 2**: A compact single-board computer ideal for embedded projects
- **Waveshare SX126X LoRa HAT**: A LoRa expansion board supporting 433/470/868/915MHz frequency bands

## Features

- Multi-hop packet routing for extended communication range
- Off-grid communication capability
- Low power consumption suitable for battery-powered devices
- Decentralized and resilient network architecture
- Support for repeater, companion, and room server modes

## Hardware Requirements

- Raspberry Pi Zero 2 W
- Waveshare SX1262 or SX1268 LoRa HAT (868M version recommended for Europe)
- Micro SD card (minimum 16GB recommended)
- USB power supply (5V/2.5A recommended)
- LoRa antenna (SMA or IPEX connector)
- GPIO headers (optional, for breadboard prototyping)

## Hardware Specifications

### Raspberry Pi Zero 2
- Processor: Dual-core ARM Cortex-A53 @ 1GHz
- RAM: 512MB LPDDR2
- GPIO: 40-pin header
- Connectivity: WiFi & Bluetooth (Zero 2 W variant)

### Waveshare SX126X LoRa HAT
- Chip: SX1262 or SX1268
- Frequency Bands: 433/470/868/915MHz (ISM license-free)
- Max Transmit Power: 22dBm
- Max Range: ~5km (open area, sunny conditions)
- Receive Sensitivity: -147dBm @0.3Kbps
- Interface: UART (via GPIO or USB)
- Working Voltage: 5V, Logic Voltage: 3.3V

## Quick Start Guide

### Step 1: Prepare Raspberry Pi

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Flash Raspberry Pi OS Lite (32-bit recommended for Pi Zero 2)
3. Boot the device and connect via SSH:
   ```bash
   ssh pi@raspberrypi.local
   ```

### Step 2: Connect LoRa HAT Hardware

1. Power off the Raspberry Pi
2. Align the SX126X LoRa HAT with GPIO pins
3. Press down firmly to connect all pins
4. Attach the LoRa antenna to the SMA or IPEX connector
5. Power on the device

### Step 3: Enable Serial Port

```bash
sudo raspi-config
# Navigate to: Interfacing Options -> Serial
# Select: No (disable login shell) -> Yes (enable hardware serial)
```

### Step 4: Install Dependencies

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3 python3-pip git
sudo apt install -y python-serial  # For Python 2 (if needed)
pip3 install pyserial
```

### Step 5: Download MeshCore Flasher

Visit [MeshCore Web Flasher](https://flasher.meshcore.co.uk/) to:
1. Select your device (if Raspberry Pi is not listed, use an ESP32-based device)
2. Choose firmware type: Companion, Repeater, or Room Server
3. Flash the firmware using Web Serial API (requires Chrome/Edge)

### Step 6: Download Waveshare Demo Code

```bash
cd ~/Documents
wget https://files.waveshare.com/upload/1/18/SX126X_LoRa_HAT_CODE.zip
unzip SX126X_LoRa_HAT_CODE.zip
cd SX126X_LoRa_HAT_Code/raspberrypi/python
```

### Step 7: Run Example

```bash
sudo python3 main.py
```

Follow the on-screen prompts:
- Press 'i' to send data
- Press 's' to send CPU temperature every 10 seconds
- Press 'c' to stop temperature broadcast
- Press 'Esc' to exit

## Repository Structure

```
MeshCore-RaspberryPi-Zero2-SX126X/
├── README.md                           # This file
├── LICENSE                             # MIT License
├── docs/
│   ├── MESHCORE_SETUP.md              # MeshCore installation and configuration
│   ├── WAVESHARE_SETUP.md             # Waveshare SX126X setup guide
│   ├── RASPBERRY_PI_SETUP.md          # Raspberry Pi Zero 2 setup guide
│   ├── HARDWARE_WIRING.md             # GPIO wiring diagram and connections
│   └── TROUBLESHOOTING.md             # Common issues and solutions
├── config/
│   ├── meshcore_config.yaml           # MeshCore configuration template
│   └── lora_settings.conf             # LoRa module settings
├── examples/
│   ├── basic_receiver.py              # Simple LoRa receiver example
│   ├── basic_transmitter.py           # Simple LoRa transmitter example
│   ├── mesh_node.py                   # Full mesh network node example
│   └── repeater_setup.py              # Repeater mode configuration
└── scripts/
    ├── install.sh                      # Automated installation script
    ├── setup_serial.sh                # Serial port configuration script
    └── test_connection.sh             # Connection test script
```

## Configuration

### LoRa Module Settings

Key parameters to configure in `config/lora_settings.conf`:

```
FREQUENCY=868         # MHz (868 for Europe, 915 for Americas)
ADDRESS=1             # Node address (0-255)
POWER=22              # dBm (10, 13, 17, 22)
AIR_SPEED=2400        # bps (300-62500)
BUFFER_SIZE=240       # bytes
NET_ID=0              # Network ID (0-255)
```

### Serial Port Configuration

Default serial port for Pi Zero 2: `/dev/ttyS0` (hardware UART)

Alternative: `/dev/ttyAMA0` or USB serial if using CP2102 converter

## GPIO Pinout

| Pin Name | GPIO | Function |
|----------|------|----------|
| M0       | 22   | Mode control 0 |
| M1       | 27   | Mode control 1 |
| RXD      | N/A  | UART RX |
| TXD      | N/A  | UART TX |
| GND      | GND  | Ground |
| 5V       | 5V   | Power |

### LoRa Operating Modes

Set via M0/M1 GPIO pins:

| M1 | M0 | Mode | Description |
|----|----|------|-------------|
| H  | L  | 0    | Transmission mode (default) |
| H  | H  | 1    | WOR (Wake-on-Radio) mode |
| L  | L  | 2    | Configuration mode |
| L  | H  | 3    | Deep sleep mode |

*H = pulled high, L = pulled low (GND)*

## MeshCore Firmware Options

### Companion Firmware
- Connects to external apps via BLE, USB, or WiFi
- Suitable for user-facing mesh devices
- Apps available: [MeshCore Web](https://app.meshcore.nz/), Android, iOS

### Repeater Firmware
- Relays messages between nodes
- Extends network coverage
- Configuration: [Web Config Tool](https://config.meshcore.dev/)

### Room Server Firmware
- Acts as a shared message board
- Useful for group communication
- Configuration: [Web Config Tool](https://config.meshcore.dev/)

## Testing Communication

### Basic Test (Two Devices)

1. Flash both devices with Companion firmware
2. Connect to MeshCore app on both devices
3. Send test message from Device 1
4. Verify message received on Device 2

### Range Test

1. Set up one device as Repeater
2. Set up two devices as Companions
3. Separate devices and log signal strength (RSSI)
4. Move apart gradually to determine max range

## Important Notes

### Frequency Bands
- **Europe**: 868MHz (SX1262 868M variant)
- **Americas**: 915MHz (SX1262 915M variant)
- **Asia-Pacific**: 433/470MHz variants

### Power Consumption
- Transmit: ~100mA
- Receive: ~11mA
- Sleep: ~2µA

### Distance Factors
- Antenna height and orientation
- Obstacles (trees, buildings, terrain)
- Weather conditions (rain reduces range)
- Frequency selection (lower = greater range)

## Troubleshooting

### Serial Port Issues

Verify correct serial port:
```bash
ls -l /dev/serial*
ls -l /dev/ttyS*
ls -l /dev/ttyUSB*
```

### Connection Problems

1. Check antenna connection
2. Verify GPIO pins are correctly soldered/connected
3. Test with CP2102 USB adapter on separate computer
4. Run `sudo python3 main.py` with proper permissions

### No Reception

1. Verify both devices on same frequency
2. Check M0/M1 mode settings (should be 0 for transmission)
3. Confirm power supply is stable (5V/2.5A minimum)
4. Test antenna with nearby device first

### Poor Range

1. Increase transmit power (22dBm max)
2. Reduce air speed (slower = better range)
3. Add external antenna with higher gain
4. Check for RF interference on frequency band

## Documentation References

### MeshCore
- [MeshCore GitHub Repository](https://github.com/meshcore-dev/MeshCore)
- [MeshCore FAQ](https://github.com/meshcore-dev/MeshCore/blob/main/docs/faq.md)
- [MeshCore Discord Community](https://discord.gg/BMwCtwHj5V)

### Waveshare SX126X
- [Waveshare SX1262 LoRa HAT Wiki](https://www.waveshare.com/wiki/SX1262_868M_LoRa_HAT)
- [SX1268 Datasheet](https://files.waveshare.com/upload/c/c4/SX1268_V1.0.pdf)
- [SX1262 Datasheet](https://files.waveshare.com/upload/6/62/DS_SX1261-2_V1.1.pdf)

### Raspberry Pi
- [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)
- [Raspberry Pi Zero 2 Getting Started](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/)
- [GPIO Pinout Reference](https://pinout.xyz/)

## Community & Support

- **MeshCore Community**: [Discord Server](https://discord.gg/BMwCtwHj5V)
- **Waveshare Support**: [Service Portal](https://service.waveshare.com/)
- **Raspberry Pi Community**: [Forums](https://forums.raspberrypi.com/)

## License

MIT License - See LICENSE file for details

This repository contains examples and documentation based on:
- MeshCore (MIT License)
- Waveshare documentation and demo code
- Raspberry Pi official documentation

## Contributing

Contributions are welcome! Please:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Version History

- **v1.0** (2026-01-02): Initial setup guide and documentation

## Disclaimer

This guide is provided as-is for educational and personal use. Users are responsible for:
- Complying with local radio frequency regulations
- Proper antenna installation and safety
- Security of wireless communication
- Hardware compatibility and modifications

## Additional Resources

### Getting Help

If you encounter issues:

1. Check the TROUBLESHOOTING.md guide in `/docs`
2. Search existing GitHub Issues
3. Join the MeshCore Discord community
4. Consult Waveshare wiki and forums
5. Review Raspberry Pi official documentation

### Similar Projects

- [Meshtastic](https://meshtastic.org/) - Another LoRa mesh networking project
- [Reticulum](https://github.com/markqvist/Reticulum) - Resilient long-range packet radio network stack
- [LoRa Alliance](https://lora-alliance.org/) - LoRaWAN specifications and resources

---

**Last Updated**: January 2, 2026
**Status**: Active Development
