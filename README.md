# ALUP

The ALUP (Arduino LED USB Protocol, name may change) is a Protocol which handles connection-based transmission of RGB data.

## Overview

It's goal is to make it possible for almost any device to control an LED strip, even if this device has no digital Pins like an
Arduino or Raspberry Pi, by using a device with such digital pins as a proxy (middle-man).

[Fig. 1_en] (two devices connected to each other over a wired connection along with connected leds to the slave device) (TODO: add img)

The original use was to control individually addressable LEDs like ws2812b with
any kind of computer over a USB connection using an Arduino as a proxy.

If you want to control LEDs as just described, you can use an implementation of this protocol in your project. For more, see [Implementations] [TODO: add link]


## Features and Limitations
### Features:
- It's connection based, meaning it first establishes a connection between two devices using some
    initial parameters (as described in [Detailed documentation] (TODO: add link)) and then sends RGB Data continuously

- It's a transport layer protocol, therefore independent of the type of physical connection.
    For the requirements it has for the physical connection and underlying protocols, see [Requirements](TODO: add link)

- Support for custom configuration values (See: [Configuration format](TODO: add link))
- Support for subprograms (See: [Subprograms](TODO: add link))


### Properties:
- Its use is to transmit RGB data
- It is master-slave based which means one of the two connected devices takes the role of the master (aka the data sender)
    and the other one the role of the slave (the data receiver)

- Data frames are only sent from a master device to the slave device (unidirectional)


### Limitations:
- It does not offer Congestion/Flow control to minimize latency
- It does not offer control for lost/corrupted packages

- It only supports RGB (RGBW Support may be added in the future)



## <a name="setup_link"></a>Setup

This section will give an overview about how to set up devices running the ALUP v. 0.1.


Step 1: Select an implementation for the master device (See [Implementations](#Implementations_link)) fitting best for your use case and connection type.
        The listed Implementations differ in things like compatible Hardware, Operating System, etc.

Step 2: Install the implementation for the master device according to it's documentation. This implementation may have to be configured when using it or executing a program
        using it, or before installation. To configure it, see its documentation.


Step 3: Select an implementation for the slave device (See [Implementations](#Implementations_link)) fitting best for your slave device and connection type.
        Those listed Implementations also may differ in things like compatible Hardware, Operating System, etc., but are generally compatible with all master device
        implementations which use the same connection type unless stated differently in the documentation of one of the implementations.

Step 4: Configure the implementation of the slave device according to it's documentation. Most of the time, this has to be done before installing it onto the device itself, as the slave device
        probably won't interface with you directly after this point. For more information and how to configure it, see its documentation.

Step 5: Connect the devices over the previously chosen connection type (e.g. Wi-Fi, USB, ...).
        Note that you may also have to connect additional hardware like a Power Supply depending on your chosen hardware. This documentation is only about the protocol and therefore assumes
        that you have done this already yourself and will not explain how to do it or when additional connections are needed.


The devices are now ready to use. For additional help, see [Troubleshooting], [FAQ] and the documentations of the chosen implementations.




## <a name="Implementations_link"></a>Implementations
### Master Device:
- [Java-ALUP](https://github.com/Skyfighter64/Java-ALUP "Java Implementation of the ALUP")

### Slave Device:
- [Arduino-ALUP](https://github.com/Skyfighter64/Arduino-ALUP "Arduino Implementation of the ALUP")
- [ESP8266-Wifi-ALUP](https://github.com/Skyfighter64/ESP8266-WiFi-ALUP "ESP8266 Implementation of the ALUP")


### <a name="Creating_An_Implementation_link"></a>Creating an implementation:

I encourage anyone to write their own implementation of the protocol using this documentation. I tried to write it as easy to understand as possible.

If there are any parts of it that are not described well enough or are misleading or confusing, please open an issue.

If you wrote an implementation and want to add it to the list of implementations, feel free open an issue/pull request.



## <a name="Projects_link"></a>Projects
Here are some cool projects using this protocol:
- (Projects will be added here)


## <a name="Contributing_link"></a>Contributing

This section is about the guidelines for contributing to this protocol. If you want to contribute by creating your own implementation
of the protocol, see [Implementations](#Implementations_link)


Currently work in progress; there may be new guides added to this section in the future.

Documentation Style:
- When writing documentation, make sure that it is easy to understand. If there is any part in the existing documentation which is unclear or
    difficult to understand, please open an issue.

Features:
- New features should be compatible with the current protocol. If they are not, it may be possible to include them in a new version of
    this protocol in the future.

- New Features should be possible to implement for any device and connection type.




## License

This project is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0). For more information, see [LICENSE](https://github.com/Skyfighter64/ALUP/blob/master/LICENSE)
