# Installations

1. Install Solaar (Linux device manager for Logitech devices):

   GitHub: https://github.com/pwr-Solaar/Solaar  
   Terminal:
   ```console
   sudo pacman -S solaar
   ```

2. Install hidapitester (Simple command-line program to exercise HIDAPI):

   GitHub: https://github.com/todbot/hidapitester  
   TODO: how to compile and where to put it
3. Install ddcutil (Linux program for managing monitor settings):

   Terminal:
   ```console
   sudo pacman -S ddcutil
   ```
# Configuration

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
   hidapitester --vidpid 046D/B034 --open --length 20 --send-output 0x11,0xFF,0x0A,0x1D,0x01,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00
   ```
   
