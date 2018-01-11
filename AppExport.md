# App Export

## Firmware voor ESP8266, NodeMCU, BlynkBoard, etc.

#### Voorbereiding voor je ontwikkel omgeving
1. Installeer [Arduino IDE](https://www.arduino.cc/en/Main/Software)
2. Installeer [Blynk Library](https://github.com/blynkkk/blynk-library/releases/latest) en start de Arduino IDE opnieuw op
3. Installeer [ESP8266 core for Arduino](https://github.com/esp8266/Arduino#installing-with-boards-manager)
4. Voor Windows / macOS is het mogelijk dat je drivers voor je serieële poort moet installeren, al naar gelang de convertor die gebruikt:
 - СP2102: https://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx
 - FTDI (FT232, etc): http://www.ftdichip.com/Drivers/VCP.htm
 - *TODO: Link to drivers for CH340 and PL2303.*
5. Als je bord een NeoPixel RGB LED aan boord heeft, installeer [Adafruit NeoPixel](https://github.com/adafruit/Adafruit_NeoPixel) library vanuit de Library Manager

#### Bouw je Firmware
1. Open het voorbeeld in de Arduino IDE: ```File -> Examples -> Blynk -> Provisioning -> Blynk_ESP8266```
2. Open ```Settings.h``` tab.
3. Configureer je firmware:
  * ```BOARD_NAME``` - ...
  * ```BOARD_VENDOR``` - ...
  * ```PRODUCT_WIFI_SSID``` - ...

#### Upload firmare
1. Selecteer het type bord: ```Tools -> Board -> [Your Board]```
2. Selecteer de poort: ```Tools -> Port -> [...]```
3. Verify en Upload!

Voor het Blynk Board kun je het volgende type selecteren ```NodeMCU 1.0```.
