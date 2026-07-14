 I've been wanting to pull data from my car parked in the garage for some time, and after recently purchasing another car, I found the motivation to put some effort into the process. I've found some projects using the Torque app, but I wanted a solution that allowed for direct communication from the OBD Bluetooth module through an ESP32 to homeassistant.

My repo for my complete config yaml is over here:

[clumsyCoder00/OBD-II-ESPHome-ESP32-Configuration](https://github.com/clumsyCoder00/OBD-II-ESPHome-ESP32-Configuration/blob/main/ESP32%20OBD.yaml)

I'm using an ESP32-S3 N16R8 because it's what I had laying around, but I imagine any ESP32 variant will do.

To connect to the car I purchased the [VEEPEAK OBDCHeck BLE](https://veepeak.com/products/obdcheck-ble) to connect to the vehicle.

I found this custom component [BLE ELM327 Component](https://github.com/eigger/espcomponents/tree/master/components/ble_elm327) to make the connection between the ELM327 and homeassistant.

To establish communication between the ESP32 and the VEEPEAK, I needed to first enable the BLE tracker module in ESPhome in order to find the MAC address of the VEEPEAK. The process is detailed on this page: [ESP32 Bluetooth Low Energy Device](https://esphome.io/components/binary_sensor/ble_presence/#setting-up-devices). The MAC address is entered in the ble_client section of the yaml file.

Next I needed to retrieve the service_uuid, rx_char_uuid & tx_char_uuid values. These are unique to the particular OBD BLE module hardware being used. Hopefully you can find the values in conversations online as I did for the VEEPEAK. If anyone knows of a good place to find these, please chime in!

At this point I was connected to the CAN bus and and ready to make requests for data. There are many modules on the bus, so the particular module of which I am making the request must be specified before making the request. this is done through the ATSH command. The address of each module is called the "ECU header."

I was able to find all of my OBD PID's and associated ECU headers over at https://obdb.community . I used this information to populate the ESPhome sensor values including the arithmetic needed to convert the value to a useful value in home assistant. Many presets for common vehicle PID's are listed on the github page for the custom component. Less common PID's might require some bit shifting or bit masking. I added the formula, device class, unit of measurement and accuracy decimals to complete the parameters of each sensor.

This ESPHome configuration allows me to configure all the data I want to see from my vehicle's OBD interface on a webpage hosted on the ESP32 or in home assistant.

*** There is some risk to leaving the module plugged in all the time. It appears that my module draws about 12mA of current from the battery while in standby and 45mA when active. If the car isn't driven for a week, this may be an issue. I'll keep track of the 12V battery voltage to determine if it is regularly pulled down to an unhealthy value and if I can leave it plugged in all the time. I disable the BLE section of the ESP32 with the software switch as shown in the config yaml, and turn it on every hour or so to grab values then allow the VEEPEAK go back into standby mode.

There is some risk involved with leaving the module plugged in all the time, as the VIN of the vehicle is accessible through the CAN bus and if someone wardriving is looking for an open Bluetooth module, the VIN is accessible. I don't know if this is any more risky than having the VIN visible in the windshield just as every other car does. *** 
