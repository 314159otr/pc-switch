# Installations

1. Install Solaar (Linux device manager for Logitech devices):

   GitHub: https://github.com/pwr-Solaar/Solaar  
   Terminal:
   ```console
   sudo pacman -S solaar
   ```

2. Install hidapitester (Simple command-line program to exercise HIDAPI):

   GitHub: https://github.com/todbot/hidapitester  
   Terminal:
   ```console
   cd ~
   git clone https://github.com/libusb/hidapi
   git clone https://github.com/todbot/hidapitester
   cd hidapitester
   make
   mv hidapitester ~/.local/bin/
   cd ~
   rm -fr hidapi*
   ```
   
4. Install ddcutil (Linux program for managing monitor settings):

   Terminal:
   ```console
   sudo pacman -S ddcutil
   ```
   Once had to do:
   ```console
   sudo modprobe i2c-dev
   ```
# Configuration

## Logitech Devices

1. Find device:

   - Solaar device name (MX Master 3S):

     Terminal:
     ```console
     solaar show
     ```
     Output:
     ```console
     MX Master 3S
     Device path  : /dev/hidraw3
     USB id       : 046d:B034
     Codename     : MX Master 3S
     Kind         : mouse
     Protocol     : HID++ 4.5
     Serial number:
     Model ID:      B03400000000
     ```
   - Hidapitester device vidpid (046D/B034):
     
     Terminal:
     ```console
     hidapitester --list-detail
     ```
     Output:
     ```console
     046D/B034:  - Logitech MX Master 3S
     vendorId:      0x046D
     productId:     0xB034
     usagePage:     0xFF43
     usage:         0x0202
     serial_number: df:32:d4:4f:58:fb 
     interface:     -1 
     path: /dev/hidraw3
     ```
2. Find the HID instruction to switch the channel of the device (11 FF 0A1D 01000000000000000000000000000000):

   Terminal:
   ```console
   solaar -ddd config "MX Master 3S" change-host 2
   ```
   Output:
   ```console
   ...
   2025-02-08 15:34:28,737,737    DEBUG [MainThread] logitech_receiver.settings: change-host: prepare write(2:PIOTRBLASZCZYK2) => b'\x01'
   2025-02-08 15:34:28,737,737    DEBUG [MainThread] logitech_receiver.base: (9) <= w[11 FF 0A1D 01000000000000000000000000000000]
   Setting change-host of MX Master 3S to 2:PIOTRBLASZCZYK2
   ```
3. Compose the command:

   Write the HID in this format:
   ```console
   0x11,0xFF,0x0A,0x1D,0x01,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00
   ```  
   Only the first 5 values are different from 0x00:  
   `0x11`: Always stays 0x11  
   `0xFF`: 0x00 or 0xFF = bluetooth, 0x01,0x02... = The number of the device connected to the Unifying receiver  
   `0x0A`: Device specific value  
   `0x1D`: Device specific value (0x10...0x1F works too...)  
   `0x01`: Channel to switch: 0x00 = channel 1, 0x01 = channel 2, 0x02 = channel 3

   Command looks like:
   ```console
   hidapitester --vidpid 046D/B034 --usage 0x0202 --usagePage 0xFF43 --open --length 20 --send-output 0x11,0xFF,0x0A,0x1D,0x01,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00
   ```

## Monitor

1. Find device (display 1):

   Terminal:
   ```console
   sudo ddcutil detect
   ```
   Output:
   ```console
   Display 1
      I2C bus:  /dev/i2c-2
      DRM connector:           card1-HDMI-A-1
      EDID synopsis:
         Mfg id:               AOC - UNK
         Model:                24B2W1G5
         Product code:         9218  (0x2402)
         Serial number:        XXTNBHA000107
         Binary serial number: 107 (0x0000006b)
         Manufacture year:     2022,  Week: 48
      VCP version:         2.1
   ```
2. Find the input source feature (60):

   Terminal:
   ```console
   sudo ddcutil capabilities --display 1
   ```
   Output:
   ```console
   ...
   Feature: 60 (Input Source)
      Values:
         01: VGA-1
         03: DVI-1
         11: HDMI-1
   ...
   ```
3. Compose the command:

   Terminal:
   ```console
   ddcutil setvcp 60 01 --display 1
   ```
