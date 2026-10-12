# JTAGprobe
A high-performance CMSIS-DAP v2 JTAG and SWD debug probe firmware with an integrated USB CDC-ACM UART serial bridge for the **Raspberry Pi Pico (RP2040)** and **Raspberry Pi Pico 2 (RP2350)**.

---

## Features

- **JTAG & SWD Debugging:** Full CMSIS-DAP v2 bulk transport support for OpenOCD, pyOCD, probe-rs, and Keil MDK.
- **Hardware USB CDC-ACM UART Bridge:** Virtual COM port (`/dev/ttyACM0` on Linux, `COMx` on Windows) bridged directly to target UART on GPIO 12/13.
- **High Performance:**
  - Zero blocking debug logging overhead on DAP transfers.
  - Single-cycle RP2040 SIO atomic register operations for high-speed bit-banging.
  - Multi-packet ring buffer (4 slots) preventing USB NAK back-offs.
- **Dynamic Baud Rate:** Automatically adjusts hardware UART baud rate, parity, stop bits, and data bits when changed by the host terminal software.
- **Supported Hardware:**
  - Raspberry Pi Pico (RP2040)
  - Raspberry Pi Pico 2 (RP2350)

---

## Pinout Mapping

Connect the probe to your target board as follows:

| Signal | RP2040 GPIO | Pico Header Pin | Direction / Role | Description |
| :--- | :---: | :---: | :--- | :--- |
| **SWCLK / TCK** | **GPIO 19** | **Pin 25** | Output (Probe $\rightarrow$ Target) | SWD Clock / JTAG Test Clock |
| **SWDIO / TMS** | **GPIO 14** | **Pin 19** | Bi-directional | SWD Data I/O / JTAG Test Mode Select |
| **TDI** | **GPIO 18** | **Pin 24** | Output (Probe $\rightarrow$ Target) | JTAG Test Data In |
| **TDO** | **GPIO 21** | **Pin 27** | Input (Probe $\leftarrow$ Target) | JTAG Test Data Out |
| **nTRST** | **GPIO 15** | **Pin 20** | Output (Probe $\rightarrow$ Target) | JTAG Test Reset (optional) |
| **nRESET** | **GPIO 16** | **Pin 21** | Open-drain (Target Reset) | System Reset |
| **UART_TXD** | **GPIO 12** | **Pin 16** | Output (Probe $\rightarrow$ Target) | Target UART RX (from Pico `uart0`) |
| **UART_RXD** | **GPIO 13** | **Pin 17** | Input (Probe $\leftarrow$ Target) | Target UART TX (to Pico `uart0`) |
| **GND** | **GND** | **Pin 3, 8, 13, 18, 23, 28, 38** | Ground | Common Ground Reference |

---

## USB Interfaces & Serial Port

When plugged into a host PC, JTAGprobe enumerates as a composite USB device:

1. **Interface 0 (Vendor Specific Bulk):**
   - CMSIS-DAP v2 interface used by OpenOCD, pyOCD, and debuggers via `libusb`.
   - *Note:* CMSIS-DAP is **not** a serial/tty device and will not appear under `/dev/tty*`.
2. **Interfaces 1 & 2 (CDC-ACM UART Bridge):**
   - Creates **`/dev/ttyACM0`** on Linux (or `COMx` on Windows).
   - In system logs (`dmesg`, `/dev/serial/by-id/`), `/dev/ttyACM0` inherits the USB product string `JTAG probe on Pico (CMSIS-DAP)`. **`/dev/ttyACM0` is the serial console port connected to GPIO 12 & 13.**

### Opening the UART Console

Use any serial terminal application (e.g. `picocom`, `minicom`, `screen`, PuTTY):

```bash
# Using picocom at 115200 baud
picocom -b 115200 /dev/ttyACM0

# Using minicom
minicom -b 115200 -o -D /dev/ttyACM0
```

---

# Images of debugging PICO with another PICO and Bluepill

![Bluepill Debug](images/bluepilldebug.jpg)

![Pico Debug](images/picodebug.jpg)



# Hacking

For the purpose of making changes or studying of the code, you may want to compile the code yourself.

First, clone the repository:
```
git clone https://github.com/dst202/JTAGprobe
cd JTAGprobe
```
Initialize and update the submodules:
```
git submodule update --init --recursive
```

## Prerequisites

Before building, ensure you have the following installed:

- **Raspberry Pi Pico SDK** - Set `PICO_SDK_PATH` environment variable
  ```bash
  export PICO_SDK_PATH=/path/to/pico-sdk
  ```
- **CMake** (3.12 or higher)
- **GCC ARM Embedded Toolchain**

## Building

### Option 1: Build for Raspberry Pi Pico (RP2040)

```bash
mkdir -p build/pico && cd build/pico
cmake ../.. -DPICO_BOARD=pico
make
```

The output will be `build/pico/JTAGprobe.uf2`

### Option 2: Build for Raspberry Pi Pico 2 (RP2350)

```bash
mkdir -p build/pico2 && cd build/pico2
cmake ../.. -DPICO_BOARD=pico2
make
```

The output will be `build/pico2/JTAGprobe.uf2`

## Flashing the Firmware

1. **Hold the BOOTSEL button** on your Pico/Pico2
2. **Connect to USB** while holding BOOTSEL (the device will appear as RPI-RP2 drive)
3. **Copy the appropriate `.uf2` file** to the drive:
   - For Pico (RP2040): `build/pico/JTAGprobe.uf2`
   - For Pico 2 (RP2350): `build/pico2/JTAGprobe.uf2`
4. The device will automatically reboot and run the firmware
