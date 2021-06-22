# ESPhome_codes

## Contents

1. General
2. Basic example
3. TODO
4. References / Further reading

## 1. General

Repository for experimental ESPHome codes (yaml files).

ESPHome is a system for generating firmware for ESP32 / ESP8266 boards and uploading this wirelessly to WIFI capable boards. For more information on ESPHome, see reference 1.

Normally the firmware (a binary file) is obtained writing C++ code or C++ - like code (Arduino .ino)  and compiling this. ESPHome uses a yaml file containing a description of the firmware functionality. The system converts this into the C++ code and subsequently compiles it. The generated C++ code and the firmware file (.bin)  


## 2. Basic example
**Assumption:**  you have an ESPHome server up and running and have access to the dashboard.


I have a server running under Docker on my Synology NAS. I can access the ESPHome dashboard in my browser with http://<NAS-IP>:6052, which uses the default port number. The docker container uses an external bound volume, which means that contents of the /config folder of the application is directly accessible for me on the storage volume of the NAS.

In this example I will create the firmware for a very basic application on a SB-NodeMCU-ESP32 board from Joy-It (https://joy-it.net/en/products/SBC-NodeMCU-ESP32).     
Documentation for this board can be found here: https://joy-it.net/files/files/Produkte/SBC-NodeMCU-ESP32/SBC-NodeMCU-ESP32-Manual-20200320.pdf 

![alt text](https://github.com/goofy2k/ESPhome_codes/blob/main/media/SBC-NodeMCU-ESP32-02_cropped.png "SBC-NodeMCU-ESP32 (Joy-it)")

This is a very affordable ESP32 board. It has one huge disadvantage if you want to use it for experimenting with connection of other hardware to it's pins:  the board does NOT fit on normal breadboards. The distance between the two pin rows does not fit with the breadboard spacing. There are solutions available for this on the web, but it takes some cutting...
         
![alt text](https://github.com/goofy2k/ESPhome_codes/blob/main/media/separated_breadboard_cropped.jpg "separated breadboard")

ESPHome is capable of doing over-the-air (OTA) firmware updates. It should be evident that before it is possible to do OTA updates, the firmware must be capable to receive those. This requires a WIFI connection and a routine to receive updates over WIFI. The very first firmware with this minimum capabilities must be loaded to the board over a wired (USB) connection. All subsequent updates can be done over-the-air. Unless something is wrong with the latest firmware uploaded to the board or something is wrong with the WIFI connection.

If the root cause of failure is in the firmware, a repaired firmware version must be uploaded via a USB connection. For cases where the previously used WIFI network is not available a fallback solution can be built into the firmware. After a predetermined number of connection attempts, the board can go into access point (AP) mode and serve a webpage where the user can set the credentials for an available wireless network. The board can then reboot and start connecting to the set network. Note, that after a subsequent firmware update the edited WIFI settings are lost and the user must set them again (see TODO 2).  

After completion of the basic firmware example, the board will be able to:
         
- connect to an available WIFI network
- receive OTA firmware updates
- go to AP mode and accept edited WIFI creedentials in case connection fails
         
Next steps:         

1. create yaml file with basic functionality         
2. compile / create firmware.bin
3. download firmware.bin to PC
4. upload firmware via USB (requires a upload application)
5. check logs (via USB, via ESP log over WIFI)
6. disconnect from USB
7. reboot and check logs (ESP log over WIFI)         
8. make some firmware changes in yaml file
9. compile and upload (OTA)
10. inspect logs  
         
1. create yaml file with basic functionality
         
Open ESPHome dashboard in browser
Create new node (+ button) 
Follow the steps in the wizzard       
Enter node name: joyit_nodemcu_esp32_ex
Select device type: Generic ESP32 (WROVER Module)
WiFi & Updates: WiFi credentials, OTA password
Submit
         
After these steps a file **joyit_nodemcu_esp32_ex.yaml** has been created in the /config folder with the following contents:
    code block
esphome:
  name: joyit_nodemcu_esp32_ex
  platform: ESP32
  board: esp-wrover-kit

wifi:
  ssid: "<myssid>"
  password: "<mypass>"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Joyit Nodemcu Esp32 Ex"
    password: "PPzHIfjNV92T"

captive_portal:

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: "OTAmacho_99"

ota:
  password: "OTAmacho_99"      

         
The file can be viewed and edited under the Edit button    maybe you have to note the auto-generated password for the AP mode?????     
The credentials for the fallback hotspot (AP) have been automatically generated.          
With validate the contents of the yaml file can be validated. As this one is auto-generated by the wizzard, the result is obviously positive :-)
It makes no sense to use the Upload or Logs buttons as the board does not have OTA Upload capabilities and is not yet connected with the ESPHome API server. You can try Logs and see the messages....

The firmware must be created without trying to upload it:
         Choose "Compile" from the three dot menu of the board's box in the dashboard
         
3. I have to download the file:  "DOWNLOAD BINARY" and "CLOSE" the window and 
        
         filename ..... joyit_nodemcu_esp32_ex.bin in your downloads folder 
         it is also available in the output in the /config folder    .....   as firmware.bin .....
         
4. flash the file over USB with the ESPHome-flasher, see:
 joyit_nodemcu_esp32_ex.bin in your download folder, but can also be found in the /config/joyit_nodemcu_esp32_ex  folder ....
         https://esphome.io/guides/faq.html#i-can-t-get-flashing-over-usb-to-work
         https://github.com/esphome/esphome-flasher/releases
         
       start ESPHome-Flasher-1.3.0-Windows-x64.exe 
         choose COM part
         select file 
         Flash   and see messages appear
         
         flasher_output.log 
         
    5. Check logs  (board_logs.log)
         
         cannot connect, board is on wifi

WARNING Error resolving IP address of joyit_nodemcu_esp32_ex.local. Is it connected to WiFi?
WARNING (If this error persists, please set a static IP address: https://esphome.io/components/wifi.html#manual-ips)
WARNING Initial connection failed. The ESP might not be connected to WiFi yet (Error resolving IP address: Error resolving address with mDNS: Did not respond. Maybe the device is offline., [Errno -2] Name or service not known). Re-Trying in 1 seconds
         
Implement...  >> new yaml
         compile
         download binary
         flash (over usb)
         
         ESP_36BF6D
         
         
         test fallback hotspot:
         flash with wrong WIFI credentials
         
         static IP adress
         https://esphome.io/components/wifi.html#manual-ips
         
         OK
         Now upload (OTA) firmware with wrong wifi credentials...
         
         A AP appears: ESP_36BF6D  but with a different name than the configured one.  After some time (minutes) the configured one appears. Connection to it is not possibel
         
         captive portal
         https://esphome.io/components/captive_portal.html
         http://192.168.4.1/
### Remarks:         

The source codes (in this case yaml files!) are saved in the /config folder and have the device/application name (appname.yaml). The output (C++ code, compiled firmware(?)) are stored in a separate folder per app:  /config/appname.  Other resources must be stored in .....          
         

## 3. TODO

1. Put secrets (credentials, host IP's etc) in a separate file and do not publish those. Test Secrets Editor.
2. Find a solution for keeping WIFI credentials after a firmware update
3. Investigate "Update All"
4. Investigate "Clean MQTT" 
5. Investigate "Clean Build"
6. Discuss (high level) integration in Home Assistant         
         

## 4. References / Further reading

1. https://esphome.io/
2. https://www.home-assistant.io/
