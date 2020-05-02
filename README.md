# esp32wifibase
Basic starting point for an ESP32 running MicroPython with an interactive wifi setup.

Requirements:
* Any ESP32 board (ESP8266 should work as well but not tested)
* Micro USB cable
* Python 3

_Recommended: Linux/MacOS dev environment or Windows 10 running WSL_


## Local dev setup
Ensure that Python 3 is installed (`which python3` `python3 --version`).

Create a new virtualenv for your project (I use [`virtualenvwrapper`](https://virtualenvwrapper.readthedocs.io/en/latest/)).

Then install dependencies:
```
pip install -r requirements.txt
```


## Download MicroPython firmware
Identify your chip here: https://micropython.org/download/
(ESP32 for this specific project)

The ESP32s seem to come in two basic flavors: ESP-WROOM-32 (most common) or ESP-WROVER.

The WROOM versions do not have additional SPIRAM so opt for the "GENERIC" firmware (whereas the WROVER can use the "GENERIC-SPIRAM" firmware).

Download the highest version ESP-IDF v3.x that does not say "-unstable" (e.g. esp32-idf3-20191220-v1.12.bin).


## Flashing the firmware
With the ESP32 plugged into your computer via USB you now how to find its USB-to-UART port. The various tutorials I found assume a Linux OS where the ESP32 shows up as `/dev/ttyUSB0`. See [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html) for specific instructions for your OS.

On my Mac the ESP32 was identified as both `/dev/tty.SLAB_USBtoUART` and `/dev/cu.SLAB_USBtoUART` (no clue what the difference is, if any).

I was able to confirm that either worked by querying some basic info via `esptool.py`:
```
$ esptool.py --port /dev/tty.SLAB_USBtoUART read_mac
esptool.py v2.8
Serial port /dev/tty.SLAB_USBtoUART
Connecting........_____....._____.
Detecting chip type... ESP32
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 24:62:ab:dc:b3:cc
Uploading stub...
Running stub...
Stub running...
MAC: 24:62:ab:dc:b3:cc
Hard resetting via RTS pin...
```

So now we're ready to clear the board. Wipe it clean with `erase_flash`:
```
$ esptool.py --chip esp32 --port /dev/tty.SLAB_USBtoUART erase_flash
esptool.py v2.8
Serial port /dev/tty.SLAB_USBtoUART
Connecting........_____....._____....._____....._____....._____....._____.
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 24:62:ab:dc:b3:cc
Uploading stub...
Running stub...
Stub running...
Erasing flash (this may take a while)...
Chip erase completed successfully in 13.9s
Hard resetting via RTS pin...
```

_(my Mac was inconsistent about connecting to the board. Unplugging/replugging the USB connection worked)_

Now flash the MicroPython firmware to the board with `write_flash`:
```
$ esptool.py --chip esp32 --port /dev/tty.SLAB_USBtoUART --baud 460800 write_flash -z 0x1000 esp32-idf3-20191220-v1.12.bin
esptool.py v2.8
Serial port /dev/tty.SLAB_USBtoUART
Connecting........_
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 24:62:ab:dc:b3:cc
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Compressed 1247280 bytes to 787794...
Wrote 1247280 bytes (787794 compressed) at 0x00001000 in 19.2 seconds (effective 519.9 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

If you run into problems, check [here](https://docs.micropython.org/en/latest/esp32/tutorial/intro.html#deploying-the-firmware).

We can also verify that nothing was corrupted with `verify_flash`:
```
$ esptool.py --chip esp32 --port /dev/tty.SLAB_USBtoUART verify_flash 0x1000 esp32-idf3-20191220-v1.12.bin 
esptool.py v2.8
Serial port /dev/tty.SLAB_USBtoUART
Connecting........_____.....____
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 24:62:ab:dc:b3:cc
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
Auto-detected Flash size: 4MB
Verifying 0x130830 (1247280) bytes @ 0x00001000 in flash against esp32-idf3-20191220-v1.12.bin...
-- verify OK (digest matched)
Hard resetting via RTS pin...
```


## Testing basic interaction with the board via python
We can run python interactively on the board via the python REPL. To do this we need a terminal environment that can communicate over the serial port to the ESP32. Based on (this tutorial)[https://www.youtube.com/watch?v=QopRAwUP5ds] I installed [`picocom`](https://github.com/npat-efault/picocom).

On my Mac I can install it via homebrew:
```
brew install picocom
```

And now we can run it by pointing it to our ESP32 port and a specified baud rate (communication speed with the board):
```
$ picocom -b 115200 /dev/tty.SLAB_USBtoUART
picocom v3.1

port is        : /dev/tty.SLAB_USBtoUART
flowcontrol    : none
baudrate is    : 115200
parity is      : none
databits are   : 8
stopbits are   : 1
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
hangup is      : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv -E
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,
logfile is     : none
initstring     : none
exit_after is  : not set
exit is        : no

Type [C-a] [C-h] to see available commands
Terminal ready

```

** Pro-tip: How the heck to exit picocom?! CTRL(A + X) (while holding CTRL, type A, type X)
