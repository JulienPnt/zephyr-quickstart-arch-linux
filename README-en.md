# Zephyr Quick Start Guide on Arch Linux
> The writing and translation of this guide were assisted by AI (Mistral/ChatGPT).

## Introduction
**Note**: This tutorial is inspired by the [official Zephyr Quick Start Guide](https://zephyrproject.org/zephyr-os-getting-started-on-manjaro-arch-linux/), but aims to go further.

Here, you will find:
1. **Detailed explanations** of the Arch Linux packages to install and their purpose, to help you understand each step of Zephyr's configuration.
2. **Step-by-step guidance** for creating a new project, with an introduction to using the Device Tree.
3. **A concrete illustration** of one of Zephyr's major strengths: its **exceptional portability**, allowing you to transfer a project from one board to another with just a few adjustments.

---

## Prerequisites
You should already know how to install and configure:
- Neovim or LazyVim
- Kitty
- Zsh
- Git

For optimal use of this tutorial, you should have access to:
- [esp32-c3-devkitm-1](https://docs.zephyrproject.org/latest/boards/espressif/esp32c3_devkitm/doc/index.html)
- [stm32-nucleo-f446re](https://www.st.com/en/evaluation-tools/nucleo-f446re.html)

Other evaluation boards can be used, but they will require editing specific Device Tree overlays for the blink project (which is a great exercise!).

---

## Table of Contents
- [[#Dependencies]]
- [[#West]]
- [[#Creating a Workspace]]
- [[#USB Rules (udev)]]
- [[#Example: Blinking an LED]]

---

## Dependencies
Install the packages required for Zephyr:
```bash
sudo pacman -S python-pip python-setuptools python-wheel python-pyserial gperf wget curl xz ninja file cmake bison flex gcc dtc openocd arm-none-eabi-gcc arm-none-eabi-gdb patchelf dfu-util gcovr python-anytree python-breathe python-intelhex python-packaging python-ply python-pyaml python-pyelftools python-pkwalify python-tabulate ccache doxygen python-jsonschema
```
*Installation command*

Below is a description of each installed package:

| Package               | Usage                                                                                     |
|-----------------------|-------------------------------------------------------------------------------------------|
| git                   | Distributed version control system.                                                       |
| python-pip            | Official Python package installer.                                                        |
| python-setuptools     | Python library for building and distributing packages.                                   |
| python-wheel          | Extension of setuptools to use the **wheel** binary format.                               |
| python-pyserial       | Python library for serial communication (UART). Required for flashing firmware and hardware interaction. |
| gperf                 | Perfect hash function generator (GNU).                                                   |
| wget                  | Command-line HTTP/HTTPS/FTP client (GNU).                                                 |
| curl                  | Data transfer client (more flexible than wget, also allows resource modification/creation). |
| xz                    | Lossless compression/decompression tool.                                                  |
| ninja                 | Build system optimized for speed (successor to Make, often used with CMake, Meson, or Gyp).|
| file                  | Determines file type.                                                                     |
| cmake                 | Cross-platform build generation and management tool.                                      |
| bison                 | Parser generator (GNU).                                                                   |
| make                  | Build automation tool.                                                                    |
| flex                  | Lexical analyzer generator (often used with Bison).                                       |
| gcc                   | GNU C/C++ compiler.                                                                       |
| dtc                   | Device Tree compiler.                                                                     |
| openocd               | Hardware debugging tool (Open On-Chip Debugger).                                          |
| arm-none-eabi-gcc     | Cross-compiler for ARM EABI processors.                                                   |
| arm-none-eabi-binutils| Binary utilities for ARM EABI (assembly, linking, etc.).                                  |
| arm-none-eabi-gdb     | GNU debugger for ARM EABI.                                                                |
| patchelf              | Tool to modify ELF executables (e.g., RPATH, dynamic linker).                             |
| dfu-util              | USB Device Firmware Update tool.                                                          |
| gcovr                 | Code coverage report generator (gcov).                                                    |
| python-pytest         | Python testing framework.                                                                 |
| python-anytree        | Data tree manipulation in Python.                                                         |
| python-breathe        | Integration of Doxygen with Sphinx for documentation.                                     |
| python-intelhex       | Intel HEX format handling (firmware files).                                               |
| python-packaging      | Python packaging tools.                                                                   |
| python-ply            | Lexical and syntax analysis in Python.                                                    |
| python-pyaml          | YAML format handling in Python.                                                           |
| python-pyelftools     | ELF file manipulation in Python.                                                          |
| python-pkwalify       | YAML/JSON schema validation in Python.                                                    |
| python-tabulate       | Formatted table generation.                                                               |
| ccache                | Compilation cache (speeds up repeated builds).                                            |
| doxygen               | Documentation generator.                                                                  |
| python-jsonschema     | JSON schema validation.                                                                   |

*Description of required packages for using Zephyr*

---

## West
> [!NOTE]
> **West** is Zephyr's **meta-tool**. It simplifies:
> - Directory and dependency management
> - Compilation (native or cross-compilation)
> - Firmware flashing
> - Debugging
>
> West can be extended via **plugins**.
> - [West GitHub Repository](https://github.com/zephyrproject-rtos/west)
> - [Official Documentation](https://docs.zephyrproject.org/latest/develop/west/index.html)

### Installing West
1. Clone the AUR repository:
   ```bash
   cd /tmp
   git clone https://aur.archlinux.org/python-west.git
   ```
2. Build the package:
   ```bash
   cd python-west
   makepkg -s
   ```
3. Install the package:
   ```bash
   sudo pacman -U python-west*.*
   ```

---

## Creating a Workspace
> [!NOTE]
> The Zephyr workspace centralizes all source files, dependencies, and Zephyr OS revisions.

1. Create a directory in `${HOME}`:
   ```bash
   mkdir ~/zephyr-workspace
   ```
2. Create and activate a Python virtual environment:
   ```bash
   python3 -m venv ~/zephyr-workspace/.venv
   source ~/zephyr-workspace/.venv/bin/activate
   ```
3. Install West and initialize the workspace:
   ```bash
   pip install west
   west init ~/zephyr-workspace
   cd ~/zephyr-workspace
   west update
   ```
   Resulting directory structure:
   ```
   zephyr-workspace
   └── zephyr
   ```
4. Install Zephyr-specific Python dependencies:
   ```bash
   west packages pip --install
   ```
5. Install the Zephyr SDK:
   ```bash
   west sdk install
   ```

---

## USB Rules (udev)
Boards (MCU/FPGA) require USB device access for flashing and debugging. Zephyr provides ready-to-use `udev` rules.

1. Download the udev rules:
   ```bash
   wget https://raw.githubusercontent.com/zephyrproject-rtos/openocd/refs/heads/zephyr-20220611/contrib/60-openocd.rules -O 60-openocd.rules
   ```
2. Move the file:
   ```bash
   sudo mv 60-openocd.rules /etc/udev/rules.d/
   ```
3. Reload the rules:
   ```bash
   sudo udevadm control --reload
   ```
4. If the device is already connected:
   ```bash
   sudo udevadm trigger
   ```

---

## Example: Blinking an LED
We will compile and flash the `blink` example on an **ESP32-C3 DevKitM** or **STM32 Nucleo-F446RE** board.

### 1. Creating the Project
1. Create a project directory:
   ```bash
   mkdir -p ~/zephyr-workspace/apps/blink
   ```
2. Clone the repository:
   ```bash
   git clone <repo_blink> ~/zephyr-workspace/apps/blink
   ```
   The repository contains:
   - `boards/`: **Overlay** files (Device Tree) for each supported board
   - `CMakeLists.txt`: Build configuration
   - `prj.conf`: Project configuration (enabled Zephyr modules)
   - `src/main.c`: Application source code

### 2. Compilation
1. Activate the virtual environment:
   ```bash
   source ~/zephyr-workspace/.venv/bin/activate
   ```
2. Export the necessary variables:
   ```bash
   west zephyr-export
   ```
3. List available boards:
   ```bash
   west boards | grep esp32
   ```
4. Compile for the target board:
   ```bash
   west build -p always -b esp32c3_devkitm blink
   ```

### 3. Preparing the Board and Introduction to Device Tree
To make this tutorial more generic, we use an **external LED connected to GPIO2** instead of the built-in LED.
[Figure 1: ESP32 wiring diagram](./doc/ZephyrEsp32BlinkTutorial.drawio.png)

#### Device Tree Used
The Device Tree is used to:
1. Configure **pin 2** as a GPIO.
2. Define an **alias** to identify this pin in the code.
   ```dtc
   / {
       aliases {
           my-led = &led0;
       };
       leds {
           compatible = "gpio-leds";
           led0: user_led {
               gpios = <&gpio0 2 GPIO_ACTIVE_HIGH>;
           };
       };
   };
   ```
   *Device Tree used*

#### Key Points
- `my-led`: Alias accessible from the code
- `led0`: Local identifier in the Device Tree
- `user_led`: Descriptive label
- `gpio0`: GPIO controller
- `2`: Pin number used
- `GPIO_ACTIVE_HIGH`: LED active at high state

Here is a C code snippet that imports the LED using its alias:
```c
#include <zephyr/drivers/gpio.h>
#include <zephyr/kernel.h>
// Settings
static const int32_t sleep_time_ms = 500;
static const struct gpio_dt_spec led =
    GPIO_DT_SPEC_GET(DT_ALIAS(my_led), gpios);
```
*Example of importing `led0` via the `my-led` alias*

The `led` variable is then manipulated using Zephyr's HAL (Hardware Abstraction Layer) functions:
```c
gpio_is_ready_dt(&led) // Check if GPIO is initialized
gpio_pin_configure_dt(&led, GPIO_OUTPUT); // Set GPIO as output
gpio_pin_set_dt(&led, state); // Set GPIO state
```
*Examples of HAL functions for GPIO control*

### 4. Flashing the Firmware
1. Install `esptool` for ESP32:
   ```bash
   sudo pacman -S esptool
   ```
2. Flash the code:
   ```bash
   west flash
   ```

### 5. Flashing the Firmware on a Nucleo-F446RE
One of Zephyr's greatest strengths is its **unmatched portability** for bare-metal projects. To illustrate this, we will port the _blink_ project to a **Nucleo-F446RE** board, this time using the built-in **LED2 (LD2)**.
[Figure 2: Nucleo-F446RE diagram](./doc/ZephyrNucleoF446re.png)

Porting the _blink_ project to STM32 with Zephyr only requires creating a **Device Tree overlay** specific to the Nucleo-F446RE. Here is its content:
```dtc
/ {
    aliases {
        my-led = &green_led_2;
    };
};
```
*Device Tree overlay for the Nucleo-F446RE*

This overlay file simply defines an alias (`my-led`) pointing to the label `green_led_2`, which is already present in the Nucleo-F446RE's base Device Tree.

**Additional step**: Before flashing the board, ensure you have the necessary tools to program STM32 microcontrollers. On Arch Linux, use the following command:
```bash
sudo pacman -S stm32cubeprogrammer
```

Finally, compile and flash the project with a single command using **West**:
```bash
west build -p always -b nucleo_f446re
west flash
```

