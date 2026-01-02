# Troubleshooting Guide

## Common Issues and Solutions

### Hardware Connection Issues

#### Problem: LoRa HAT not recognized

**Symptoms:**
- No /dev/ttyS0 found
- GPIO pins not responding
- Serial port errors

**Solutions:**
1. Check physical connection:
   ```bash
   # Verify HAT is fully seated
   ls -l /sys/class/gpio/
   ```

2. Enable hardware UART:
   ```bash
   sudo raspi-config
   # Interfacing -> Serial -> Enable
   sudo reboot
   ```

3. Test serial port:
   ```bash
   stty -F /dev/ttyS0 9600
   echo "test" > /dev/ttyS0
   ```

#### Problem: GPIO pins not accessible

**Solutions:**
```bash
# Add user to gpio group
sudo usermod -a -G gpio $USER
sudo usermod -a -G dialout $USER

# Apply changes
newgrp gpio

# Or use sudo
sudo python3 main.py
```

### Serial Communication Issues

#### Problem: Permission denied on /dev/ttyS0

**Solutions:**
```bash
# Option 1: Add user to dialout group
sudo usermod -a -G dialout $USER
logout  # Log back in

# Option 2: Change permissions
sudo chmod 666 /dev/ttyS0

# Option 3: Use sudo
sudo python3 main.py
```

#### Problem: No serial port found

**Check available ports:**
```bash
ls -l /dev/tty*
ls -l /dev/serial*

# For USB-to-Serial:
ls -l /dev/ttyUSB*

# List with descriptions:
cat /proc/tty/drivers | grep serial
```

**Try alternative ports in code:**
```python
serial_ports = [
    "/dev/ttyS0",    # Hardware UART
    "/dev/ttyAMA0",  # Alternative hardware
    "/dev/ttyUSB0",  # USB adapter
]

for port in serial_ports:
    try:
        node = sx126x.sx126x(serial_num=port, ...)
        print(f"Connected to {port}")
        break
    except:
        pass
```

#### Problem: Baud rate mismatch

**Check serial configuration:**
```bash
# Set baud rate to 9600 (standard for Waveshare)
stty -F /dev/ttyS0 9600

# Verify
stty -F /dev/ttyS0 -a
```

**In code:**
```python
node = sx126x.sx126x(
    serial_num="/dev/ttyS0",
    # Baud rate is hardcoded at 9600 in module
)
```

### LoRa Module Issues

#### Problem: Module not responding

**Symptoms:**
- "No response from module"
- Timeout errors
- No data transmitted/received

**Checks:**
1. Power supply:
   ```bash
   # Verify 5V supply to HAT
   # Use multimeter: measure between 5V and GND pins
   # Should read ~5.0V
   ```

2. Antenna connection:
   - Check SMA or IPEX connector
   - Try wiggling antenna gently
   - Try alternative antenna

3. GPIO pin configuration:
   ```bash
   # Check M0/M1 status
   gpio read 22  # M0
   gpio read 27  # M1
   # Should show 0 (LOW) or 1 (HIGH)
   ```

4. Reset module:
   ```python
   # Add to code
   import RPi.GPIO as GPIO
   
   GPIO.setmode(GPIO.BCM)
   GPIO.setup(22, GPIO.OUT)  # M0
   GPIO.setup(27, GPIO.OUT)  # M1
   
   # Reset to mode 0 (transmission)
   GPIO.output(22, GPIO.LOW)
   GPIO.output(27, GPIO.HIGH)
   time.sleep(0.5)
   GPIO.output(27, GPIO.LOW)
   time.sleep(0.5)
   ```

#### Problem: No data reception

**Checklist:**
1. Same frequency on both devices
2. Same network ID
3. Both in transmission mode (M0=LOW, M1=HIGH)
4. Antenna properly attached
5. Not too far apart (start <1m)
6. No obstacles between devices

**Debug code:**
```python
print(f"Frequency: {node.freq} MHz")
print(f"Address: {node.addr}")
print(f"Power: {node.power} dBm")
print(f"Air Speed: {node.air_speed} bps")
print(f"Network ID: {node.net_id}")
```

#### Problem: Poor signal strength / Short range

**Optimization steps:**
1. Increase power:
   ```python
   node = sx126x.sx126x(..., power=22)  # Max power
   ```

2. Lower air speed (slower = better range):
   ```python
   node = sx126x.sx126x(..., air_speed=300)  # Min air speed
   ```

3. Improve antenna:
   - Use external SMA antenna
   - Elevate antenna height
   - Avoid obstacles
   - Clear line of sight

4. Check RSSI (signal strength):
   ```python
   # Enable RSSI in code
   node = sx126x.sx126x(..., rssi=True)
   
   # Will show signal strength in dBm
   # Typical: -120dBm (weak) to -70dBm (strong)
   ```

### Python/Software Issues

#### Problem: Module import error

**Error:** `ModuleNotFoundError: No module named 'sx126x'`

**Solution:**
```bash
# Ensure correct path
cd ~/Documents/SX126X_LoRa_HAT_Code/raspberrypi/python

# Or add to sys.path in code:
import sys
sys.path.insert(0, '/home/pi/Documents/SX126X_LoRa_HAT_Code/raspberrypi/python')
import sx126x
```

#### Problem: GPIO module not found

**Error:** `ModuleNotFoundError: No module named 'RPi'`

**Solution:**
```bash
# Install GPIO library
sudo apt install -y python3-rpi.gpio

# Or via pip
pip3 install RPi.GPIO
```

#### Problem: Serial module not found

**Error:** `ModuleNotFoundError: No module named 'serial'`

**Solution:**
```bash
# Install pyserial
pip3 install pyserial

# Verify
python3 -c "import serial; print(serial.__version__)"
```

#### Problem: Timeout or blocking issues

**Symptoms:**
- Program hangs
- "Timeout waiting for data"
- Cannot interrupt (Ctrl+C doesn't work)

**Solution:**
```python
# Add timeouts
self.ser.timeout = 1  # 1 second timeout
self.ser.write_timeout = 1

# Or in main loop
import signal

def timeout_handler(signum, frame):
    print("Timeout!")
    raise TimeoutError()

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(10)  # 10 second alarm
```

### MeshCore Issues

#### Problem: Web Flasher not working

**Symptoms:**
- "No devices found"
- Device drops connection
- Flashing fails

**Solutions:**
1. Use Chrome or Edge browser
   - Firefox/Safari don't support Web Serial API

2. Check USB connection:
   ```bash
   lsusb  # List USB devices
   ```

3. Install CH340 driver (if needed):
   ```bash
   # For Chinese USB adapters
   sudo apt install -y ch340 ch341
   ```

4. Reset device and retry

#### Problem: MeshCore app can't connect

**Solutions:**
1. Verify device flashed correctly
2. Check connection method (BLE/USB/WiFi)
3. Restart app
4. Restart device
5. Check firmware version

### Network Issues

#### Problem: Mesh network not forming

**Checklist:**
1. All devices powered on
2. All devices can see each other
3. Same frequency band
4. Network ID consistent
5. Repeater nodes configured correctly

**Debug:**
```bash
# Check device addresses
for i in 1 2 3; do
    echo "Device $i:"
    ssh pi@raspberrypi${i}.local "python3 test_address.py"
done
```

#### Problem: Packet loss

**Causes:**
- RF interference
- Too high air speed
- Power too low
- Obstacles blocking signal
- Too many hops

**Solutions:**
1. Increase power (up to 22dBm)
2. Lower air speed
3. Reduce distance/obstacles
4. Enable LBT (Listen Before Talk):
   ```python
   node = sx126x.sx126x(..., lbt=True)
   ```

## Performance Optimization

### For Low Power (Battery Operation)

```python
node = sx126x.sx126x(
    freq=868,
    addr=1,
    power=10,           # Lower power
    air_speed=300,      # Lower speed = better range, less power in RX
    wor=True,           # Wake-on-Radio mode
    buffer_size=64      # Smaller buffer
)
```

Expected battery life improvements:
- 2-3x longer with WOR mode
- 10x+ longer in sleep mode
- Trade-off: reduced range

### For Maximum Range

```python
node = sx126x.sx126x(
    freq=868,
    addr=1,
    power=22,          # Maximum power
    air_speed=300,     # Slowest speed
    relay=True,        # Relay mode to extend range
    lbt=True           # Reduce packet loss
)
```

## Testing Tools

### Basic Connectivity Test

```bash
#!/bin/bash
# test_lora.sh

echo "Testing LoRa HAT..."
echo "1. Checking serial port"
ls -l /dev/ttyS0

echo "2. Checking GPIO"
ls -l /sys/class/gpio/gpio22/
ls -l /sys/class/gpio/gpio27/

echo "3. Running demo"
cd ~/Documents/SX126X_LoRa_HAT_Code/raspberrypi/python
sudo python3 main.py
```

### RF Analyzer

```python
# analyze_signal.py
import sx126x
import time

node = sx126x.sx126x(serial_num="/dev/ttyS0", freq=868, rssi=True)

for i in range(100):
    if node.ser.inWaiting():
        data = node.ser.read(node.ser.inWaiting())
        # Extract and print RSSI
        print(f"Signal: {data}")
    time.sleep(0.1)
```

## Log Files & Debugging

### Enable Debug Output

```bash
# Redirect output to log file
sudo python3 main.py > lora_debug.log 2>&1 &

# Monitor in real-time
tail -f lora_debug.log
```

### System Logs

```bash
# Check system messages
sudo journalctl -u rc-service
dmesg | tail -20

# Check GPIO events
sudo cat /sys/kernel/debug/gpio
```

## Getting Help

1. **Check Documentation**:
   - README.md (overview)
   - MESHCORE_SETUP.md (MeshCore specific)
   - WAVESHARE_SETUP.md (hardware specific)

2. **Community Resources**:
   - [MeshCore Discord](https://discord.gg/BMwCtwHj5V)
   - [Waveshare Support](https://service.waveshare.com/)
   - [Raspberry Pi Forums](https://forums.raspberrypi.com/)

3. **Online Research**:
   - GitHub Issues
   - Stack Overflow (tag: raspberry-pi, lora)
   - Electronics forums

---

**Last Updated**: January 2, 2026
**Common Issues Covered**: 20+
**Solutions Provided**: 50+
