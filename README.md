# ALUP

The ALUP (Arduino LED USB Protocol, name may change) is a Protocol which handles connection-based transmission of RGB data.

## Overview

ALUP makes it possible for almost any device (Sender) to control addressable LED strips via an ALUP Receiver.


The original use was to simply control individually addressable LEDs like WS2812b with a more powerful computer over a USB connection to an Arduino Nano. Since then it expanded to other microcontrollers and connections as well with new features and improvements.

## Features
- __Connection-Independent:__ Almost any connection such as Serial (USB) or TCP (via Wi-Fi/Ethernet) can be used.

- __Customizable:__ Programs can add custom configuration values and trigger pre-defined commands on the Receiver.

- __Realtime:__ Designed to work as fast as possible with features like time stamps, time synchronization and frame buffering.

## Implementations
If you want to control LEDs as just described, see the reference implementations:
- [Python-ALUP (Sender)](https://github.com/Skyfighter64/Python-ALUP)
- [Arduino-ALUP (Receiver)](https://github.com/Skyfighter64/Arduino-ALUP)

## Documentation
The detailed protocol documentation is available at [Documentation/Documentation_en-us.md](#https://github.com/Skyfighter64/ALUP/blob/master/Documentation/Documentation_en-us.md)



## License
This project is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0). For more information, see [LICENSE](https://github.com/Skyfighter64/ALUP/blob/master/LICENSE)
