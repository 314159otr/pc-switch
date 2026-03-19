
# Installations
All needed is in the folder
# Configuration

## Logitech Devices

1. Find device:

   Terminal:
   ```console
   hidapitester.exe --list-detail
   ```

2. Find the HID instruction to switch the channel of the device (11 FF 0A1D 01000000000000000000000000000000):

   Terminal:
   ```console
   TODO
   ```
   Output:
   ```console
   TODO
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
   TODO
   I have the monitor connected via an adapter that doesnt let sending commands
