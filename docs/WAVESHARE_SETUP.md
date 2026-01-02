# Waveshare SX126X LoRa HAT Setup Guide

## Product Overview

The Waveshare SX126X LoRa HAT is a Raspberry Pi expansion board based on the SX1262/SX1268 chip, providing wireless serial port functionality with LoRa modulation.

## Key Specifications

### Hardware
- **Chip**: SX1262 or SX1268
- **Frequency Bands**: 433/470/868/915MHz (license-free ISM bands)
- **Max Transmit Power**: 22dBm (configurable: 10, 13, 17, 22dBm)
- **Max Range**: ~5km (ideal conditions: open area, sunny weather, antenna height 2.5m)
- **Receive Sensitivity**: -147dBm @0.3Kbps
- **Interface**: UART (GPIO or USB via CP2102)
- **Working Voltage**: 5V, Logic Voltage: 3.3V
- **Buffer Size**: 1000 bytes
- **Transmit Length**: 240 bytes (configurable: 32, 64, 128, 240 bytes)
- **Air Speed**: 0.3K~62.5Kbps (software selectable)

### Power Consumption
- **Transmit**: ~100mA (transient current)
- **Receive**: ~11mA
- **Sleep**: ~2µA (deep sleep mode)
- **Working Temperature**: -40°C to +85°C

## Hardware Components

1. **SX1262/SX1268 LoRa Module** - RF transceiver chip
2. **74HC125V** - Voltage level translator (5V to 3.3V)
3. **CP2102** - USB-to-UART converter
4. **Raspberry Pi GPIO Connector** - 40-pin header
5. **USB Port** - For serial debugging and alternative connection
6. **UART Header** - For MCU connections (Arduino/STM32)
7. **SMA Antenna Connector** - For external antenna
8. **IPEX Antenna Connector** - Alternative internal antenna
9. **Indicators**: RXD/TXD (UART), AUX (auxiliary), PWR (power)
10. **Jumpers**: UART selection (A/B/C) and LoRa mode (M0/M1)

## Hardware Connection

### GPIO Connections

| Function | Pin | GPIO |
|----------|-----|------|
| M0 | GPIO22 | Mode Control 0 |
| M1 | GPIO27 | Mode Control 1 |
| RXD | GPIO15 | UART RX (BCM) |
| TXD | GPIO14 | UART TX (BCM) |
| GND | GND | Ground |
| 5V | 5V | Power |
| 3.3V | 3.3V | Logic Power |

### LoRa Module Operating Modes

Set via M0 and M1 GPIO pins:

| M1 | M0 | Mode | Description | Use Case |
|----|----|----|------------|----------|
| H  | L  | 0  | Transmission | Normal communication, sending/receiving data |
| H  | H  | 1  | WOR (Wake-on-Radio) | Ultra-low power, wakes on signal |
| L  | L  | 2  | Configuration | Setting parameters via UART |
| L  | H  | 3  | Deep Sleep | Minimal power consumption |

*H = HIGH (not connected, pulled up internally), L = LOW (connected to GND)*

### UART Selection Jumpers

| Jumper | Position | Function |
|--------|----------|----------|
| A      | Short    | Control LoRa module via USB-to-UART (CP2102) |
| B      | Short    | Control LoRa module via Raspberry Pi GPIO |
| C      | Short    | Access Raspberry Pi terminal via USB-to-UART |

For Raspberry Pi use: Position B (GPIO control)

## Installation Steps

### Step 1: Physical Installation

1. **Power Off** - Turn off Raspberry Pi completely
2. **Align HAT** - Align the 40-pin header with Raspberry Pi GPIO
3. **Insert Firmly** - Press down steadily until fully seated
4. **Verify Contacts** - Check all pins are properly connected
5. **Antenna Connection** - Attach LoRa antenna to SMA or IPEX connector
6. **Power On** - Apply power to Raspberry Pi

### Step 2: Enable Serial Port

```bash
sudo raspi-config
# Navigate to: Interfacing Options -> Serial
# Select: "No" for disable login shell
# Select: "Yes" for enable hardware serial port
# Reboot when prompted
```

### Step 3: Install Python Dependencies

```bash
# Update package list
sudo apt update
sudo apt upgrade -y

# Install Python and pip
sudo apt install -y python3 python3-pip

# Install pyserial (required for serial communication)
pip3 install pyserial

# Install GPIO library (if needed)
pip3 install RPi.GPIO
```

### Step 4: Download Demo Code

```bash
cd ~/Documents
wget https://files.waveshare.com/upload/1/18/SX126X_LoRa_HAT_CODE.zip
unzip SX126X_LoRa_HAT_CODE.zip
ls -la SX126X_LoRa_HAT_Code/
```

### Step 5: Configuration

Edit the configuration in the Python demo code:

```bash
cd ~/Documents/SX126X_LoRa_HAT_Code/raspberrypi/python
nano main.py
```

Key parameters to configure:

```python
# Create LoRa node
node = sx126x.sx126x(
    serial_num="/dev/ttyS0",  # Serial port
    freq=868,                   # Frequency in MHz
    addr=1,                     # Node address (0-255)
    power=22,                   # Transmit power in dBm
    rssi=True,                  # Enable RSSI signal strength
    air_speed=2400,             # Baud rate (bps)
    net_id=0,                   # Network ID
    buffer_size=240,            # Payload buffer size
    relay=False,                # Relay mode (False for normal)
    lbt=True,                   # Listen Before Talk
    wor=False                   # Wake-on-Radio
)
```

## Testing the Setup

### Test 1: Check Serial Connection

```bash
# Verify serial port is available
ls -l /dev/ttyS0

# Try to read from serial (Ctrl+C to exit)
stty -F /dev/ttyS0 9600
cat < /dev/ttyS0
```

### Test 2: Run Demo Code

```bash
cd ~/Documents/SX126X_LoRa_HAT_Code/raspberrypi/python
sudo python3 main.py
```

The program will:
- Initialize the LoRa module
- Display received messages
- Accept keyboard input for sending

Controls:
- **i** - Send message (interactive)
- **s** - Send CPU temperature every 10 seconds
- **c** - Stop temperature broadcast
- **Esc** - Exit program

### Test 3: Two-Device Communication

1. Set up two Raspberry Pi units with LoRa HATs
2. Set different addresses:
   - Device 1: `addr=1`
   - Device 2: `addr=2`
3. Use same frequency on both (e.g., 868MHz)
4. Run demo on both devices
5. Send message from Device 1, receive on Device 2

## Relay Communication Setup

For extended range using relay nodes:

### Configuration

Edit main.py relay section:

```python
# Node A - Direct endpoint
node = sx126x.sx126x(serial_num="/dev/ttyS0", freq=868, addr=0, relay=False)

# Node B - Relay node (requires Windows config tool)
# Set via RF Setting software: addr=1, relay mode enabled

# Node C - Remote endpoint
# Set via RF Setting software: addr=2, relay mode enabled
```

### Steps

1. Connect Node A to Raspberry Pi
2. Connect Node B and C to separate devices/PC
3. Use RF Setting tool to configure Nodes B & C
4. Set Node A in relay mode
5. Messages from Node C relayed through Node B to Node A

## Troubleshooting

### No Serial Port Found

**Problem**: `/dev/ttyS0` doesn't exist

**Solution**:
```bash
# Check which port is available
ls -l /dev/tty*

# If using USB adapter instead:
ls -l /dev/ttyUSB*

# Enable hardware UART again
sudo raspi-config
# Interfacing Options -> Serial -> Enable
```

### "Permission Denied" Error

**Problem**: Cannot access `/dev/ttyS0`

**Solution**:
```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Apply group changes (logout and login, or use newgrp)
newgrp dialout

# Or use sudo
sudo python3 main.py
```

### Module Not Responding

**Problem**: "No response from LoRa module"

**Checks**:
1. Verify antenna is properly attached
2. Check 5V power supply is connected
3. Confirm GPIO pins are fully inserted
4. Test M0/M1 jumper positions
5. Try different serial port settings (baud rate 9600)

### Poor Reception / No Communication

**Problem**: Messages not received between devices

**Checks**:
1. **Same Frequency**: Verify both nodes use identical frequency
2. **Same Network ID**: Set `net_id` to same value
3. **Mode Settings**: Ensure M0/M1 in transmission mode (0)
4. **Power Supply**: Check stable 5V/2.5A minimum
5. **Antenna**: Use quality antenna, check orientation
6. **Distance**: Start with devices close together (<1m)
7. **Obstacles**: Move away from metal/water (RF reflectors)

### Incorrect Frequency Reading

**Problem**: Frequency shows wrong value in output

**Solution**:
- Verify `freq` parameter in code matches hardware (868 for 868M variant)
- Check datasheet for correct frequency mapping
- Some variants use different base frequencies

## Distance & Performance Factors

### Ideal Conditions (5km typical range)
- Open field/outdoor area
- Sunny weather (clear sky)
- High antenna placement (>2.5m)
- External antenna with good gain
- Max power (22dBm) and low air speed (300-2400 bps)

### Reduced Range Factors
- **Obstacles**: Buildings, trees, terrain (-10-30dB)
- **Urban Area**: Metal structures (-10-15dB)
- **Rain/Humidity**: Weather attenuation (-2-5dB)
- **Low Power**: Reduced dBm setting (-6dB per 3dB reduction)
- **High Air Speed**: Faster speed = shorter range
- **Poor Antenna**: Internal antenna vs external (-5-10dB)

## Power Consumption Optimization

For battery-powered operation:

```python
# Use WOR (Wake-on-Radio) mode
node = sx126x.sx126x(
    serial_num="/dev/ttyS0",
    freq=868,
    addr=1,
    wor=True,        # Enable WOR mode
    power=10,        # Reduce power to 10dBm
    air_speed=300    # Reduce speed for better range
)
```

Expected battery life:
- **Continuous RX**: 2-3 days (with 2000mAh battery)
- **WOR Mode**: 1-2 weeks (with 2000mAh battery)
- **Sleep Mode**: Several months possible

## Hardware Variants

### Frequency Variants

- **SX1262 433M**: China, Asia-Pacific
- **SX1262 868M**: Europe, recommended for EU projects
- **SX1262 915M**: Americas, USA/Canada
- **SX1268**: Alternative chip variant (similar specs)

### Connector Options

- **SMA**: External antenna (recommended)
- **IPEX**: Smaller internal antenna option
- **Both**: Use one, other remains unused

## Resources

- [Waveshare Wiki](https://www.waveshare.com/wiki/SX1262_868M_LoRa_HAT)
- [SX1262 Datasheet](https://files.waveshare.com/upload/6/62/DS_SX1261-2_V1.1.pdf)
- [SX1268 Datasheet](https://files.waveshare.com/upload/c/c4/SX1268_V1.0.pdf)
- [Demo Code](https://files.waveshare.com/upload/1/18/SX126X_LoRa_HAT_CODE.zip)
- [RF Setting Tool](https://files.waveshare.com/upload/8/8d/RF_Setting%28E22-E90%28SL%29%29_V1.0.zip)
- [Support Portal](https://service.waveshare.com/)

---

**Last Updated**: January 2, 2026
**Product**: Waveshare SX1262/SX1268 LoRa HAT
**Compatible With**: All Raspberry Pi models with 40-pin GPIO
