# emlogic-zephyr-addons-test
test repo for our custom zephyr addons

## install dependencies
### Ubuntu 2023.04
#### update the system
```
sudo apt update
sudo apt dist-upgrade
sudo apt autoremove
```
#### install basic dev tools
```
sudo apt install \
  build-essential gdb gdb-multiarch \
  git vim pkg-config cmake \
  ninja-build \
  ccache dfu-util device-tree-compiler wget file xz-utils \
  gperf \
  python3-dev python3-setuptools python3-tk python3-wheel
```
#### install OpenOCD
In this example, we use the Nordic nRF52833-DK STK,
which has an embedded jlink debugger.
##### to install from source:
```
git clone https://github.com/openocd-org/openocd
cd openocd
sudo apt install libtool
./bootstrap
sudo apt install libjaylink0 libjaylink-dev # deps for jlink
./configure --enable-jlink
make
sudo make install
```
##### to install via the package manager:
```
sudo apt install openocd
```
##### add udev rules
```
sudo vim /etc/udev/rules.d/99-jlink.rules
```
Add the following, replacing the product ID
with the one for your board:
```
ACTION!="add", SUBSYSTEM!="usb_device", GOTO="jlink_rules_end"
ATTR{idVendor}=="1366", ATTR{idProduct}=="1015", MODE="666"

##
# Make sure that J-Links are not captured by modem manager service
# as this service would try detect J-Link as a modem and send AT commands
# via the VCOM component which might not be liked by the target...
#
ATTR{idVendor}=="1366", ATTR{idProduct}=="1015", ENV{ID_MM_DEVICE_IGNORE}="1"

##
# Handle known CMSIS-DAP probes (taken from mbed website and OpenOCD):
#   VID 0x1366 (SEGGER)
#     PID 0x1008-100f, 0x1018-101f, 0x1028-102f, 0x1058-105f, 0x1068-106f (SEGGER J-Link)
#     We cover all of them via idProduct=10* and idVendor=1366
#
#   VID 0xC251 (Keil)
#     PID 0xF001: (LPC-Link-II CMSIS_DAP)
#     PID 0xF002: (OpenSDA CMSIS_DAP Freedom Board)
#     PID 0x2722: (Keil ULINK2 CMSIS-DAP)
#   VID 0x0D28 (mbed)
#     PID 0x0204: MBED CMSIS-DAP
#
KERNEL=="hidraw*", ATTRS{idVendor}=="1366", ATTRS{idProduct}=="1015", MODE="666"

##
# Make sure that serial consoles of J-Links can be opened with user permissions
#
SUBSYSTEM=="tty", ATTRS{idVendor}=="1366", MODE="0666", GROUP="dialout"

##
# End of list
#
LABEL="jlink_rules_end"
```

and reload the udev rules:
```
sudo udevadm control --reload
```

If your JTAG connector is plugged in, then unplugg it,
wait a few seconds, and reconnect it.

## setup
```
sudo apt update && sudo apt install python3 python3.11-venv
python3 -m venv ~/.venv.zephyr
source ~/.venv.zephyr/bin/activate
python3 -m pip install west
python3 -m pip install pyelftools
mkdir -p ~/zephyr-stuff # may be anywhere you like
cd ~/zephyr-stuff && west init
cd ~/zephyr-stuff && west update
python3 -m pip install tox
# test that we can build the tests
tox
tox -- tests/test_project.py
```

```
source ~/.venv.zephyr/bin/activate
cd ~/zephyr-stuff
git clone git@github.com:flemming-richter/emlogic-zephyr-addons-test.git
west init
```

### ../west/.config
```
[manifest]
path = emlogic-zephyr-addons-test
file = west.yml

[zephyr]
base = zephyr

[emlogic-zephyr-addons-test]
base = emlogic-zephyr-addons-test
```

### update west
```
cd ~/zephyr-stuff && west update
```

### install the toolchain
Install the toolchain

Install the latest version of the Zephyr SDK.
At the time of writing, this is version 0.16.4.

Create a dir where the SDK should be
If installing for the current user only, then
create a directory somewhere under ${HOME}/.

```
mkdir -p ~/zephyr-stuff/sdks/ # this can be any path you like
cd ~/zephyr-stuff/sdks/

wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.4/zephyr-sdk-0.16.4_linux-x86_64.tar.xz
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.4/sha256.sum | shasum --check --ignore-missing

tar xapf zephyr-sdk-*.tar.xz

cd zephyr-sdk-*/
./setup.sh # I was not sure, so I answered yes to both questions
```

## build

### building an official Zephyr package
```
source ~/.venv.zephyr/bin/activate
cd ~/zephyr-stuff/zephyr/
west build -b <BOARD> <RELATIVE_PATH_TO_PKG> [-p]
west build -t menuconfig
```

### building one of our custom Zephyr packages
```
source ~/.venv.zephyr/bin/activate
cd ~/zephyr-stuff/emlogic-zephyr-addons-test/
west build -b <BOARD> <RELATIVE_PATH_TO_PKG> [-p]
west build -t menuconfig
```
