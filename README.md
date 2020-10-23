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


## License

This project is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0). For more information, see [LICENSE](https://github.com/Skyfighter64/ALUP/blob/master/LICENSE)
