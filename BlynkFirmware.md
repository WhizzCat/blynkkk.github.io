#Blynk Firmware
## Configuratie

### Blynk.begin()

De simpelste manier om Blynk te configureren is ```Blynk.begin()```:

```cpp
Blynk.begin(auth, ...);
```
Je kan verschillende parameters gebruiken, afhankelijk van de hardware en het soort verbinding dat je gebruikt. Gebruik de voorbeelden die bij je bord horen.

```begin()``` voert de volgende stappen uit:

1. Verbind met een netwerk (WiFi, Ethernet, ...)
2. Roept ```Blynk.config(...)``` - configureerd het auth token, server adres
3. Probeert verbinding te maken met de server (timeout van 30s, blokkerende functie)

Als je shield/verbinding niet ondersteund wordt, kan je dat simpel zelf oplossen!
[Hier zijn een paar voorbeelden](https://github.com/blynkkk/blynk-library/tree/master/examples/More/ArduinoClient).

### Blynk.config()

```config()``` kan je gebruiken als je zelf je verbinding wil maken
Configureer je verbinding (WiFi, Ethernet, ...) handmatig en gebruik dan:

```cpp
Blynk.config(auth, server, port);
```
of
```cpp
Blynk.config(auth);
```

**Aantekening** Nadat ``` Blynk.config(...) ``` is aangeroepen, is Blynk nog niet verbonden met de server. Vanaf het aanroepen van ``` Blynk.run() ``` of ``` Blynk.connect() ``` wordt er geprobeerd een verbinding met de server te maken.  
Als je geen verbinding met de server wil maken kan je ``` Blynk.disconnect() ``` aanroepen, direct na je configuratie.

Mocht je WiFi gebruiken, kan je simpel ```connectWiFi``` aanroepen (om het makkelijk te maken):

```cpp
Blynk.connectWiFi(ssid, pass);
```
Als je naar open WiFi netwerken verbindt, gebruik dan een lege string als wachtwoord (```""```).

## Verbindings beheer

Er zijn een aantal verschillende functies om je te helpen met het beheer van de verbinding:

### Blynk.connect()

```cpp
# Deze functie probeert verbinding te maken met de server.
# Geeft true terug als de verbinding opgezet is of false als de de timeout bereikt is
# De standaard timeout is 30 seconden
bool result = Blynk.connect();
bool result = Blynk.connect(timeout);
```

### Blynk.disconnect()

Om de verbinding te verbreken, gebruik je:

```cpp
Blynk.disconnect();
```

### Blynk.connected()
Om de status van de verbinding naar de Blynk Server op te vragen:

```cpp
bool result = Blynk.connected();
```

### Blynk.run()
Deze functie moet regelmatig aangeroepen worden om alle Blynk verbindingen, commando's, binnenkomende data en de huishouding van de verbinding te regelen.
Deze wordt normaal gesproken aangeroepen in ``` void loop() {} ```.

Het is bijvoorbeeld niet aan te raden om ``` Blynk.run() ``` aan te roepen binnen ```BLYNK_READ ``` en/of ``` BLYNK_WRITE ``` op borden met weinig RAM

## Digitale & Analoge pinnen besturen
De library kan out-of-the-box de basis I/O besturingen aan:

    digitalRead
    digitalWrite
    analogRead
    analogWrite (PWM of analoog signaal, afhankelijk van je platform)

Er is dus geen code nodig voor simpele dingen als een LED, relais aan/uit of analoge sensoren uitlezen.

## Virtuele pinnen besturen
Virtuele pinnen zijn ontworpen om allerlei soorten data van je micro controller naar de Blynk App te sturen en vice versa.
Vergelijk een Virtuele pin maar met een data kanaal waarover je van alles kan sturen. Let wel op dat je Virtuele pinnen onderscheid
van fysieke pinnen op je hardware. Virtuele pinnen zijn slechts dat, virtueel en hebben geen fysieke eigenschappen.

Virtuele pinnen zijn bij uitstek geschikt om interfaces te bouwen met allerhande 3e partij libraries (Servo, LCD en vele anderen)
om aangepaste functionaliteit te maken.
Je apparaat kan gebruik maken van ```Blynk.virtualWrite(pin, value)``` om data naar de Blynk App te sturen en van ```BLYNK_WRITE(vPIN)```
om data van de Blynk App te ontvangen.

#### Virtuele pinnen data types
De waardes van de Virtuele pinnen worden verstuurd als strings, dus in principe zijn er geen limieten
aan de data die verstuurd kan worden.
Waar je wel om moet denken, de limieten van je gebruikte hardware platform. Bijvoorbeeld de integer op de
Arduino is 16 bit, die loopt dus van -32768 naar 32768
Je kan binnenkomende data opvangen en interpreteren als Integers, Float, Double en Strings:
```cpp
param.asInt();
param.asFloat();
param.asDouble();
param.asStr();
```

Je kan ook de hele inhoud van de raw buffer opvragen:

```cpp
param.getBuffer()
param.getLength()
```

### Blynk.virtualWrite(vPin, value)

Het is mogelijk om allerlei soorten data naar de Virtuele pinnen te sturen

```cpp
// Stuur string
Blynk.virtualWrite(pin, "abc");

// Stuur integer
Blynk.virtualWrite(pin, 123);

// Stuur float
Blynk.virtualWrite(pin, 12.34);

// Stuur meerdere waardes als Array
Blynk.virtualWrite(pin, "hello", 123, 12.34);

// Stuur RAW data
Blynk.virtualWriteBinary(pin, buffer, length);
```

**Aantekening:** Na het aanroepen van ```virtualWrite``` probeert de Virtuele pin de data direct naar het netwerk te schrijven

### Blynk.setProperty(vPin, "eigenschap", waarde)

Hiermee kan je [widget eigenschappen wijzigen](#blynk-main-operations-change-widget-properties)

### BLYNK_WRITE(vPIN)

```BLYNK_WRITE``` defines a function that is called when device receives an update of Virtual Pin value from the server:

```cpp
BLYNK_WRITE(V0)
{   
  int value = param.asInt(); // Get value as integer

  // The param can contain multiple values, in such case:
  int x = param[0].asInt();
  int y = param[1].asInt();
}
```

### BLYNK_READ(vPIN)

```BLYNK_READ``` defines a function that is called when device is requested to send it's current value of Virtual Pin to the server. Normally, this function should contain some ```Blynk.virtualWrite``` calls.

```cpp
BLYNK_READ(V0)
{
  Blynk.virtualWrite(V0, newValue);
}
```

### BLYNK_WRITE_DEFAULT()

This redefines the handler for all pins that are not covered by custom ```BLYNK_WRITE``` functions.

```cpp
BLYNK_WRITE_DEFAULT()
{
  int pin = request.pin;      // Which exactly pin is handled?
  int value = param.asInt();  // Use param as usual.
}
```

### BLYNK_READ_DEFAULT()

This redefines the handler for all pins that are not covered by custom ```BLYNK_READ``` functions.

```cpp
BLYNK_READ_DEFAULT()
{
  int pin = request.pin;      // Which exactly pin is handled?
  Blynk.virtualWrite(pin, newValue);
}
```

### BLYNK_CONNECTED()

This function is called every time Blynk gets connected to the server. It's convenient to call sync functions here.

```cpp
BLYNK_CONNECTED() {
// Your code here
}
```

### BLYNK_APP_CONNECTED()

This function is called every time the Blynk app gets connected to the server.

```cpp
BLYNK_APP_CONNECTED() {
// Your code here
}
```

[Example](https://github.com/blynkkk/blynk-library/blob/master/examples/More/AppConnectedEvents/AppConnectedEvents.ino)

### BLYNK_APP_DISCONNECTED()

This function is called every time the Blynk app gets connected to the server.

```cpp
BLYNK_APP_DISCONNECTED() {
// Your code here
}
```

[Example](https://github.com/blynkkk/blynk-library/blob/master/examples/More/AppConnectedEvents/AppConnectedEvents.ino)

### Blynk.syncAll()

Request server to send the most recent values for all widgets. In other words, all analog/digital pin states will be restored and every virtual pin will generate BLYNK_WRITE event.

```cpp
BLYNK_CONNECTED() {
    Blynk.syncAll();
}
```

### Blynk.syncVirtual(vPin)

Requests virtual pins value update. The corresponding ```BLYNK_WRITE``` handler is called as the result.

```cpp
Blynk.syncVirtual(V0);
# Requesting multiple pins is also supported:
Blynk.syncVirtual(V0, V1, V6, V9, V16);
```

## BlynkTimer

```BlynkTimer``` enables you to perform periodic actions in the main ```loop()``` context.  
It is the same as widely used SimpleTimer, but fixes several issues.  
```BlynkTimer``` is included in Blynk library by default, so no need to install SimpleTimer separately or include SimpleTimer.h   
**Please note that a single BlynkTimer object allows to schedule up to 16 timers.**

For more information on timer usage, please see: http://playground.arduino.cc/Code/SimpleTimer  
And here is a BlynkTimer [example sketch](https://github.com/blynkkk/blynk-library/blob/master/examples/GettingStarted/PushData/PushData.ino#L30).

## Debugging

### #define BLYNK_PRINT
### #define BLYNK_DEBUG

To enable debug prints on the default Serial, add on the top of your sketch **(should be the first line)**:

```cpp
#define BLYNK_PRINT Serial // Defines the object that is used for printing
#define BLYNK_DEBUG        // Optional, this enables more detailed prints
```

And enable Serial Output in setup():

```cpp
Serial.begin(9600);
```
Open Serial Monitor and you'll see the debug prints.

You can also use spare Hardware serial ports or SoftwareSerial for debug output (you will need an adapter to connect to it with your PC).

<span style="color:#D3435C;">**WARNING:** Enabling ```BLYNK_DEBUG``` will slowdown your hardware processing speed up to 10 times!</span>

### BLYNK_LOG()

When ```BLYNK_PRINT``` is defined, you can use ```BLYNK_LOG``` to print your logs. The usage is similar to ```printf```:

```cpp
BLYNK_LOG("This is my value: %d", 10);
```

On some platforms (like Arduino 101) the ```BLYNK_LOG``` may be unavailable, or may just use too much resources.  
In this case you can use a set of simpler log functions:

```cpp
BLYNK_LOG1("Heeey"); // Print a string
BLYNK_LOG1(10);      // Print a number
BLYNK_LOG2("This is my value: ", 10); // Print 2 values
BLYNK_LOG4("Temperature: ", 24, " Humidity: ", 55); // Print 4 values
...
```

## Minimizing footprint

To minimize the program Flash/RAM, you can disable some of the built-in functionality:

1. Comment-out ```#define BLYNK_PRINT``` to remove prints
2. Put on the top of your sketch:
```
#define BLYNK_NO_BUILTIN   // Disable built-in analog & digital pin operations
#define BLYNK_NO_FLOAT     // Disable float operations
```

Please also remember that a single ```BlynkTimer``` can schedule many timers, so most probably you need only one instance of BlynkTimer in your sketch.

## Porting, hacking

If you want to dive into crafting/hacking/porting Blynk library implementation, please also check [this documemtation](https://github.com/blynkkk/blynk-library/tree/master/extras/docs).
