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



# Building from Source

To compile the firmware yourself (including the CMSIS-DAP speed optimizations and the integrated UART CDC bridge):

### 1. Clone the Repository (Branch: `jtagprobe-new`)

Clone the **`jtagprobe-new`** branch directly and initialize submodules:

```bash
git clone -b jtagprobe-new https://github.com/dst202/JTAGprobe.git
cd JTAGprobe
git submodule update --init --recursive
```

*(If you already have a local clone, switch to the branch: `git checkout jtagprobe-new`)*

### 2. Prerequisites

Ensure you have the following installed and configured:

- **Raspberry Pi Pico SDK** &mdash; Set the `PICO_SDK_PATH` environment variable:
  ```bash
  export PICO_SDK_PATH=/path/to/pico-sdk
  ```
- **CMake** (3.12 or higher)
- **GCC ARM Embedded Toolchain** (`arm-none-eabi-gcc`)
- **Build tool** (`make` or `ninja`)

### 3. Building the Firmware

#### Option 1: Build for Raspberry Pi Pico (RP2040)

```bash
mkdir -p build/pico && cd build/pico
cmake ../.. -DPICO_BOARD=pico
make -j$(nproc)
```
The output binary is: **`build/pico/JTAGprobe.uf2`**

#### Option 2: Build for Raspberry Pi Pico 2 (RP2350)

```bash
mkdir -p build/pico2 && cd build/pico2
cmake ../.. -DPICO_BOARD=pico2
make -j$(nproc)
```
The output binary is: **`build/pico2/JTAGprobe.uf2`**

---

## Flashing the Firmware

1. **Hold the BOOTSEL button** on your Pico / Pico 2 board.
2. **Plug the board into your computer's USB port** while holding BOOTSEL (the device mounts as a mass-storage drive named `RPI-RP2`).
3. **Copy the appropriate `.uf2` file** to the drive:
   - For Pico (RP2040): `build/pico/JTAGprobe.uf2`
   - For Pico 2 (RP2350): `build/pico2/JTAGprobe.uf2`
4. The Pico will automatically reboot and start running JTAGprobe with CMSIS-DAP v2 and USB CDC UART on GPIO 12/13.

