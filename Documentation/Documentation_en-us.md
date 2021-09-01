
> Copyright 2021 Skyfighter64
>
>   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at
>
>       http://www.apache.org/licenses/LICENSE-2.0
>
>   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.


--------------------------------------------------------------------------------
en-us

# ALUP - Arduino LED USB Protocol (name may change)

__Version: 0.1 (internal)__



## Description

The ALUP (Arduino LED USB Protocol, name may change) is a protocol for simple transmission of RGB data.

It can be used to make it possible for almost any device, like a computer or phone, to play real-time animations on addressable LED strips. However, this protocol is just a text which defines how the data transmission for such a purpose can work. For more info's on implementations, see [Implementations](#Implementations_link). and [Projects](#Projects_link).

## Table of contents
- TODO:
- add
- me

## Overview

Generally, you have two devices:
- A master device, which wants to control a addressable LED strip, but can't because it's lacking connectivity
  - Examples: PC, Smartphone.
- A slave device, which has GPIO pins and the ability to control addressable LEDs, but lacks the performance or features for a project
  - Examples: Arduino Microcontroller, ESP8266.

As said, the master device want's to control addressable LED strips, but can't because it has no way to connect to the LEDs directly, like GPIO pins, whereas the slave device
has this connectivity, but lacks other features or performance which are needed.

This is where this protocol comes in.

The ALUP describes a way how the RGB data gets from the PC (master device) to the Microcontroller (slave device) over any kind of connection like USB or Wi-Fi, which then can apply the RGB values to the LEDs. This makes it possible for the PC to control the addressable LEDs indirectly.    

[Fig. 1_en] (two devices connected to each other over a wired connection along with connected LEDs to the slave device)




## Features & Limits

#### Features:
- __Connection based:__ It first establishes a connection between two devices and then sends RGB Data continuously.

- __Operating on the application layer:__ Almost any type of physical connection such as USB or Wi-Fi can be used.

- __Custom configurations:__ Programs can add custom configuration values. See: [Configuration format](#Configuration_Format_link).

- __Subprograms:__ Execute your own code on the microcontroller while sending RGB data. See: [Subprograms](#Subprograms_link).

#### Properties:
- __Master-slave based:__ One of the two devices takes the role of the master (the sender) and the other one the role of the slave (the receiver).

- __Unidirectional:__ RGB data is only sent in one direction.


#### Limitations:
- __RGB only.__ RGBW Support may be added in the future



## Requirements
A list of requirements for the protocol.


#### Hardware requirements:
The protocol has the following hardware requirements:

- Master device
    - e.g. a windows pc

- Slave device
    - Has to be able to control addressable LEDs
    - e.g. Arduino microcontroller

- Data connection between the devices
    - e.g. USB, UART, Wi-Fi (TCP), etc.

#### Connection Requirements:
This protocol has the following requirements for data connections:

- Error-Free transmission
- In-order transmission
- No package loss



## Special Terms
This section gives a list of all special terms used within this protocol to better understand this document.

 __Term__ | Example | Description
 -----|---------|-------------
 __RGB data__ | `R:255, B:123, G:0` | One or multiple triples of 8bit color values, represented in the RGB format, which this protocol is all about. For more, see [RGB Data](#RGB_Data_link)
 __Master Device__ | PC, Smartphone | The device which sends __RGB data__ to the __slave device__ to have it displayed on the LED strip.
 __Slave Device__ | Arduino, ESP8266 | The device which will receive the __RGB data__ from the __master device__ and apply it to the LEDs.
 __Physical Connection__ | USB, Wi-Fi  | The type of physical connection between the __master device__ and the __slave device__ which the protocol uses. This includes the entire protocol stack needed for data transmission.
  |  |
 __Frame__ | - | A full set of data which gets sent from the __master device__ to the __slave device__. Consists of a __frame header__  and a __frame body__. For more, see [Frame](#Frame_link).
 __Frame Header__ | - | The part of the __frame__ which contains special information for this frame. For more, see [Frame](#Frame_link).
 __Frame Body__ | - | The part of the __frame__ which contains __RGB data__. For more, see [Frame](#Frame_link).
  |  |
 __Command__ | - | One command can be sent with every __frame__ inside its __frame header__. It can be either a __protocol command__ or a __subcommand__.
 __Subcommand__ | - | A type of __command__ which tells the __slave device__ to execute a specific __subprogram__. [Subprograms](#Subprograms_link).
 __Protocol Command__ | - | A special type of __command__ which tells the __slave device__ to execute a special task. For more, see [Protocol Commands](#Protocol_Commands_link).
 __Subprogram__ | - | A part of the code which gets executed, when a specific __subcommand__ gets received. Each subprogram has its own __subcommand__. For more, see [Subprograms](#Subprograms_link).

## Protocol Workflow
This section states how the protocol works in detail by explaining what each device does during each
of the 3 processes. Those processes are:

- [Connecting](#Connecting_link)
- [Data Transmission](#Data_Transmission_link)
- [Disconnecting](#Disconnecting_link)


### The Basics:
A quick overview of the protocol workflow.
When two devices get connected via the physical connection, they first establish a connection and share some important
configuration data like how many LEDs are connected to the slave device.

If this happened successfully, the master device sends data frames for the LEDs until it disconnects, or the physical connection is interrupted.

When the master device finished sending data frames (e.g. the program on the master device using the protocol gets closed), the master device
disconnects from the slave device and closes the physical connection.

Those are the basics of this protocol. They are explained again but in more detail below.



:information_source: General Note: all timeout values (unless stated explicitly) are defined by the protocol implementation itself. If implementing this protocol, it is advised to use a timeout of multiple seconds for the best results.


### <a name="Connecting_link"></a>Connecting / Connection process:

Before sending any [RGB data](#RGB_Data_link), both devices need to be connected and configured. This connection process consists of 3 Steps:
- [Requesting a connection](#Requesting_A_Connection_link)
- [Exchanging configuration data](#Exchanging_The_Configuration_link)
- [Confirming the configuration data](#Confirming_The_Configuration_link)


[Fig. 2_en] (Overview of the connection process)

----

#### <a name="Requesting_A_Connection_link"></a>Requesting a connection:

##### Slave Device:
As soon as the physical connection, along with the connection of all underlying protocols (e.g. Serial) is established,
the Slave device begins to send a [connection request byte](#Connection_Request_Byte_link) and listens for a [connection acknowledgement byte](#Connection_Acknowledgement_Byte_link) in fixed intervals,
the size of which can be specified by the implementation itself (e.g.: 0.1s). Sending [connection request bytes](#Connection_Request_Byte_link) and listening for a
[connection acknowledgement byte](#Connection_Acknowledgement_Byte_link) will not time out and continue indefinitely until a [connection acknowledgement byte](#Connection_Acknowledgement_Byte_link) was received.

[Fig. 2_1_en] (Slave device sending a connection request to the master device and waiting for connection acknowledgement)


##### Master Device:
The master device starts by listening for a single Byte of data containing a [connection request byte](#Connection_Request_Byte_link)
as soon as the connection is fully established. When receiving a [connection request byte](#Connection_Request_Byte_link), the master device prepares to receive the
[Configuration](#Configuration_Format_link) next and sends a [connection acknowledgement byte](#Connection_Acknowledgement_Byte_link) back to the slave device as soon as it is ready to
receive the configuration. Listening for a connection request byte [connection request byte](#Connection_Request_Byte_link) may time out, but can also continue until one was received. If a
timeout is used in the end depends on the implementation of the Master Device.

[Fig. 3_en] (Master device sending a connection acknowledgement byte to the Slave device)


----
#### <a name="Exchanging_The_Configuration_link"></a>Exchanging the configuration:

##### Slave Device:
When receiving the [connection acknowledgement](#Connection_Acknowledgement_Byte_link), the slave device stops sending [connection requests](#Connection_Request_Byte_link) and listening for [connection acknowledgements](#Connection_Acknowledgement_Byte_link).
It now prepares the configuration to be sent in the protocols [Configuration Format](#Configuration_Format_link)
and sends it as soon as possible. After sending it waits for a [configuration acknowledgement byte](#Configuration_Acknowledgement_Byte_link),
indicating that the master device received and applied the configuration successfully, or a [configuration error byte](#Configuration_Error_Byte_link),
indicating that the configuration could not be applied. In the later case, the connection process stops and the slave device
try's for a new connection attempt by starting at the beginning and sending new connection requests.
If no configuration acknowledgement byte or configuration error byte was received within a certain timeout, the slave device assumes that the
connection is dead and also continues by trying for a new connection attempt by starting at the beginning and sending new connection requests.


[Fig. 4_en] (Slave sending configuration to master )

##### Master Device:
After sending the connection acknowledgement, the master device starts listening for the configuration by listening for a [configuration start byte](#Configuration_Start_Byte_link).
This byte marks the start of the configuration and is part of the protocol [Configuration Format](#Configuration_Format_link). Upon receiving, the master device also receives
the configuration values directly followed by it. If there was no configuration start byte received within a certain timeout, the master device assumes that the
connection is dead and stops the connection process.

When receiving the configuration in the protocol [Configuration Format](#Configuration_Format_link), the master device stops
listening for the configuration and initializes everything necessary using the received configuration. As soon as the initialization
is finished it sends an [configuration acknowledgement byte](#Configuration_Acknowledgement_Byte_link), indicating that it applied the configuration
successfully. If the configuration could not be applied, it sends an [configuration error byte](#Configuration_Error_Byte_link) to the Slave device indicating that there was
a problem while applying the configuration. Reasons for sending a [configuration error byte](#Configuration_Error_Byte_link) may be something like an incompatible protocol version, but are in the end
defined by the implementation of the master device.


Note: When implementing you may notify the user via both devices if the configuration could not be applied (e.g. Popup, LEDs blinking, etc.)

[Fig. 5_en] (Master applying configuration and sending configuration acknowledgement, slave waiting for configuration acknowledgement )



#### <a name="Confirming_The_Configuration_link"></a>Confirming the configuration:

##### Slave Device:
If the slave now receives a configuration error Byte, it stops listening to any incoming data and may try to signal the user that
a configuration error occurred.

When it receives a configuration acknowledgement, it sends an configuration acknowledgement back to the master to indicate that
the configuration process is now finished and it is ready to receive Data. The connection process is now finished and the device continues
in the data transmission part.


##### Master Device:
After sending a configuration acknowledgement byte, the master device listens for a configuration acknowledgement byte from the Slave device.
When the master receives the configuration acknowledgement from the slave device, the connection is established successfully and the
devices are ready to move on to [Data Transmission](#Data_Transmission_link).


[Fig. 2_en] (Overview of the connection process)








### <a name="Data_Transmission_link"></a>Data transmission:
Now that the connection is established and the devices are configured, it is time to send [RGB data](#RGB_Data_link) from the master device to the slave device.
The following text and images will describe the process of sending and receiving [RGB data](#RGB_Data_link) on both devices.

The data transmission process has 3 steps:

- [Sending](#Sending_Frames_link)/Receiving a frame
- Applying the frame
- Acknowledging the frame


[Fig. 1_data_en] (Overview of the Data transmission process)


#### <a name="Sending_Frames_link"></a>Sending Frames

##### Slave Device:
As soon as the Slave device sent its Configuration acknowledgement, it gets ready to receive frames of [RGB data](#RGB_Data_link) in the specified [frame format](#Frame_link) by waiting for a [Frame Header](#Frame_Header_link).

Waiting for a header should continue indefinitely until either a disconnect command is received, the underlying connection either disconnects or times out,
or a user input like a button press. For more, see [Disconnecting](#Disconnecting_link). This means that the data transmission loop should never time out on the side of the slave device by itself.


When receiving the header, it first reads in the size of the frame body and then receives the whole frame body using this size value. It then checks the
[Command Byte](#Command_Byte_link) of the header if it has the value of a [protocol command](#Protocol_Commands_link) (decimal values 0 - 7)  or the
value of a [Subcommand](#Subcommands_link) (decimal values 8 - 255).
The Slave device then executes the protocol command or Subcommand if possible.
Note that the execution of Subcommands can also happen in parallel to the main program depending on the implementation and is therefore not guaranteed to happen
before the Frame Body gets applied to the LEDs.


Depending on the command, the Slave device also has to apply the [RGB data](#RGB_Data_link) to the LEDs. To do this, it first validates the length of the frame body and the frame body offset
given by the header by checking if they comply with the specifications for the [`body size`](#Frame_Body_Size_link) and [`body offset`](#Frame_Body_Offset_link) values and then, if successful, applies the data received
in the [Frame body](#Frame_Body_link) to the LEDs if needed, using the given offset (This may not apply to [`protocol commands`](#Protocol_Commands_link) such as "disconnect").


#### Checking the frame body size:

The frame body always consists of pairs of 3 bytes which represent RGB values, it's length therefore always has to be a multiple of 3.
If it's not a multiple of 3, the slave device sends a [`frame error byte`](#Frame_Error_Byte_link) to the master device and deletes the received
frame body.

If the Frame data is less than or equal to the `number of LEDs * 3`, the Slave device applies the [RGB data](#RGB_Data_link) to the LEDs and sends a
Frame Acknowledgement Byte to the master device to indicate for one, that the data was applied and
it is ready to receive the next Frame, and also indicating that the device was not disconnected.

If the Frame data is more than the (number of LEDs * 3), the Slave Device discards all of the data from this frame body and sends
a Frame Error Byte to the master device.

This has to be done in order to prevent desynchronisation of the data stream, in which case all of the incoming data afterwards would be invalid
and the connection would have to be shut down and reestablished manually.


It now returns to the start of the Data Transmission to receive the next frame. This continues indefinitely until a "disconnect" protocol command
gets received.


#### Checking the frame body offset:

The [`frame body offset`](#Frame_Body_Offset_link) indicates, by how much LEDs the sent data should be offset from `LED 0`. It therefore has to be 0 (no offset) or bigger.
Because the [RGB data](#RGB_Data_link) of the frame should not exceed the last LED of the LED strip, the `offset + body size / 3`
should be smaller or equal to the [`number of LEDs`](#Number_Of_Leds_link). If one of those conditions is not followed, the slave device
sends a [`frame error byte`](#Frame_Error_Byte_link) to the master device, indicating that the frame could not be applied, and skips applying the frame.



[Fig. 2_data_en] (Overview of the Data transmission process for the slave device)

##### Master Device:
As soon as the Master Device receives the Configuration acknowledgement from the Slave Device, it begins building [`Data Frames`](#Frame_link) and sends them
to the Slave Device whenever the program using the Protocol decides to send a Frame, meaning the intervals for sending frames are not fixed.

The header of the frame is built by first looking at the [RGB data](#RGB_Data_link) to be sent and calculating, how many bytes are needed to send this data.

:information_source: Note that the [RGB data](#RGB_Data_link) always comes in a pair of 3 bytes, one for the R, G and B channel, meaning that the resulting raw [RGB data's](#RGB_Data_link) length always has to be a multiple of 3.

The Master Device then checks if a special command like a protocol command such as "disconnect" or a custom Subcommand should be executed on
the Slave device. If so, it sets the command byte of the header to the value of the wanted command as stated in the frame header specification [TODO: link to frame header spec]
If no command should be executed, it sets the value of the command byte to "None" (0).


It then builds the Frame body by appending each R, G and B value, each represented as one byte value, of each LED for which data should be
sent, starting the first led up to the n'th. LED where n can be any number up to the number of LEDs connected to the Slave device, which
can be found inside the configuration which was communicated on the connection process.
Note: This means, you can change all LEDs in one Frame, but don't have to. All unchanged LEDs will stay the state they were in before the frame arrived
if not overridden by a protocol command or subcommand.

The built frame gets appended at the end of the header and sent to the slave device.


The master device now waits for a frame acknowledgement byte, indicating that the slave device is ready to receive the next frame and the
slave device did not disconnect, or a frame error byte indicating that the slave device did not disconnect, but could not apply the Frame.

If there is no frame acknowledgement byte or frame error byte received within a specified time interval (eg. 10 seconds), the Master device assumes that
the Slave device disconnected and may notify the user about the timeout. This is the main way to detect if the slave device disconnected during the data transmission.

Note that the timeout interval may vary on the implementation or a user set value as it may be necessary to use a longer or shorter
interval depending on the use case and the hardware used.


If a frame acknowledgement was received, the master device continues at the beginning of the data transmission process and is allowed to send the next frame.

When receiving a frame error, the master device should warn the user about this critical problem e.g. by throwing an exception. All three cases where a frame error
gets sent should not be possible to reach by the end user. Therefore, such a frame error represents most likely an issue with the implementation itself. By sending a frame error,
the slave device also signals the master device that it discarded the faulty frame body and is ready to receive the next frame.

When no Frame acknowledgement was received within a certain timeout, the Master Device assumes that the Slave Device disconnected and kills the connection on his side
without sending a "disconnect" command to the slave device.


[Fig. 2_data_en] (Overview of the Data transmission process for the master device)




### <a name="Disconnecting_link"></a> Disconnecting:

The disconnection process can only be initiated by the master device. When the slave device wants to disconnect, it can simply
stop responding with frame acknowledgements to time out on the master device, or kill the underlying connection if possible. This way of
disconnecting indirectly is used for the ease of implementation.

Master Device:

When the master device wants to disconnect, it sends an empty frame with a [`disconnect command`](#Protocol_Commands_link) to the Slave Device and then disconnects
his side of the connection by invalidating all connection relevant values and disconnecting all underlying protocols.


Slave Device:

Upon receiving a frame with a [`disconnect command`](#Protocol_Commands_link) inside the frame header, the slave device treats the connection as dead, invalidates all connection relevant values, and disconnects the underlying connection on his side if needed. It then continues by waiting for new incoming connections.

:information_source: Note that there may be no Frame acknowledgement sent or received for the disconnection frame.



[Fig. 1_disconnect_en] (Overview of the disconnection process)


## <a name="Installing_link"></a>Installing

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

### <a name="Official_Implementations_link"></a>Official Implementations:

Master Device:
- [Java-ALUP](https://github.com/Skyfighter64/Java-ALUP "Java Implementation of the ALUP")

Slave Device:
- [Arduino-ALUP](https://github.com/Skyfighter64/Arduino-ALUP "Arduino Implementation of the ALUP")
- [ESP8266-Wifi-ALUP](https://github.com/Skyfighter64/ESP8266-WiFi-ALUP "ESP8266 Implementation of the ALUP")


> List provided implementations (code) if existing and how to properly use it (create extra documentation for each one)
> Brief description of each implementation
> Most important requirements of each implementation (Which Programming Language(s) it's designed for / Operating System / Programs it depends on)

### <a name="Creating_An_Implementation_link"></a>Creating an implementation:

Generally, we want to encourage anyone to write their own implementation of the protocol using this documentation,
as this is the goal of the documentation. Therefore, we try to make it as easy to understand as possible.
If there are any parts of it that are not described well enough or are misleading, please open an issue.

If you want to create an official implementation or you wrote an implementation and want to add it to the official implementations, make sure
your project complies with the [`official implementation standard`](#Official_Implementation_Standard_link) and open a pull request adding your project to the official implementations.


## <a name="Official_Implementation_Standard_link"></a>Official Implementation Standard

  This section describes the standard which all `Official Implementations` have to follow.

  Unofficial implementations are free to do whatever they want within the boundaries of the [`License`](/LICENSE)

#### Core values:
- All official implementations are open source
- All official implementations are free to use for anyone


#### Documentation:

The implementation has to be properly documented to make it easy to use for anyone. For this reason, the documentation of your implementation
has to contain the following points:

:information_source: Templates for the documentation can be found in the [`Templates Folder`](/Templates)

#### Specifications:
  - Protocol version used / compatible protocol versions
  - Hardware compatibility (e.g. which devices can be used)
  - Software compatibility, dependencies (e.g. which OS is supported, which programs are needed)
  - Compatible connection types (e.g. Wi-Fi, USB, etc.)

  - Any additional specifications you may want to add (e.g. special features, limitations, etc.)

#### Guidance:
  - Installation guide
  - Configuration guide along with a list of all by the user tweakable variables along with valid values and default values.
  - (If needed:) Configuration changes which have to be made to the other device (e.g. adding code for special subcommands required by the implementation)

  - How to use the implementation
  - At least one easy example on how to use it (the more the better)
  - Some simple troubleshooting help or FAQ

#### Protocol specific:
  - Which timeouts are used in each step
  - Events which cause a Frame Error Byte to be sent (Slave Device only)

#### Hardware specific:
  - Which types of (addressable) LEDs are supported


#### Subprograms:
  - Documentation for all custom subprograms used (what do they do, how and when to call them, ...)

  - Documentation for the "extra values" of the [Configuration Values] (if used)

  - How to add own subprograms if possible (state specifically if not possible) and how to call them, along with
    additional documentation if needed (e.g. when the subprograms are executed, and if they run in parallel to the main thread)


#### Code style:
  - Use a consistent coding style
  - Please add comments to your code to help others read and understand it


#### Libraries and external resources:
Only use external resources and libraries which:
  - You are allowed to redistribute (see the licensing info of each resource).
  - Can be used commercially and non-commercially by everyone


And any other missing aspects named by the protocol maintainers upon reviewing.

#### License:
We want all official implementations to be usable for anyone, therefore your implementation should use one of the following licenses:
  - MIT License
  - APACHE 2.0
  - LGPL(????)
  //TODO: rework this list and whole section

  If you want to release your implementation under a different license, please state which license you want to use and
  the reasons in your pull request. We will only accept licenses that represent our core values as stated above.



## <a name="Configuration_link"></a>Configuration
This section is about configuring implementations of the protocol. For the protocol configuration format, see [Configuration Format]


Depending on the used connection type, the implementations may have to be configured differently. Therefore it's best to consult the
documentation of each used implementation for configuring both devices.

Generally, the values defined at [Configuration Format] do have to be configured on the slave device according to their definitions.
This will also be explained in the documentation of the used implementation for the slave device


## <a name="Projects_link"></a>Projects
Here are some cool projects using this protocol:
- (Projects will be added here)


## <a name="Contributing_link"></a>Contributing

This section is about the guidelines for contributing to this protocol. If you want to contribute by creating your own implementation
of the protocol, see [Implementations](#Implementations_link)


Currently work in progress; there may be new guides added to this section in the future.
//TODO: add additional style guides if needed

Documentation Style:
- When writing documentation, make sure that it is easy to understand. If there is any part in the existing documentation which is
    difficult to understand, please open an issue.

Features:
- New features should be compatible with the current protocol. If they are not, it may be possible to include them in a second version of
    this protocol in the future depending on the demand.

- Features should be possible to implement for any implementation. If there is an exception, the feature has to include considerations on what
    happens when it is not used/ignored.


## <a name="Detailed_Documentation_link"></a>Detailed documentation

This section contains definitions and constants of the protocol

:information_source: Note: Those definitions do not apply for the use of an implementation of the protocol, they only do for
sending and receiving data using the protocol (describing in which format data is sent and received).
Therefore they are only relevant if you are writing an implementation of the protocol.

### <a name="General_Definitions_link"></a>General Definitions:

#### <a name="RGB_Data_link"></a> RGB data:
One or multiple sets of 3 bytes representing the R, G and B value for a LED each within a range of 0-255

Byte nr. | Color
:---:|:---:
0 | Red
1 | Green
2 | Blue

//TODO: add byte indexes to the image and delete table

<img src="./media/general/en/rgb_values.svg" alt="An RGB triple" height=25%>


#### Subcommand:
A command with an ID corresponding to a Subprogram sent inside of a Frame Header to execute the Subprogram.
For more information, see [Subprograms](#Subprograms_link).

####Subprogram:
A small Program which gets executed on the Slave device when a Subcommand with its ID gets received
For more information, see [Subprograms](#Subprograms_link).



### <a name="Data_Transmissions_Definitions_link"></a>Definitions for data transmission:
All mentions of the data types within this documentation refer to the definitions below if not stated otherwise.





#### String:
A string is a combination of UTF-8 encoded characters followed by a null byte used as terminator.
String data has a dynamic length; The end of a string is marked with a Null byte (0x00) as a terminator.
Therefore: When sending String data, send a Null byte (`0x00`) afterwards if it is not done by the used programming language itself.

<img src="./media/general/en/string.svg" alt="A string as defined above" height=25%>

:warning: Note: Large Strings can have a significant impact on performance


#### Integer:
An integer number is a 32-bit 2s-compliment number.

<img src="./media/general/en/integer.svg" alt="An integer as defined above" height=25%>


#### Long:
A long is a 64bit 2s-compliment number.

<img src="./media/general/en/long.svg" alt="A long as defined above" height=25%>

#### Short:
A short is a 16bit 2s-compliment number.

<img src="./media/general/en/short.svg" alt="A short as defined above" height=25%>

#### Byte:
A byte is a 8bit unsigned number ranging from 0 to 255.

<img src="./media/general/en/byte.svg" alt="A byte as defined above" height=25%>


:information_source: This is important because:
- while most architectures do use this definition, depending on the board, the architecture of the Arduino may use 16-bit numbers as integer.
Therefore, when you want to send integer data, you may actually have to use variables of the type long (32bit-integer) on the Arduino
and int (32bit-integer) on the master system. At the end, the actual bit length of the types has to match.

- Strings may use a null terminator internally, but when sending strings, the null terminator may be cut off. It is Therefore
important to ensure a null terminator is also sent so the receiving device does know the end of the string.

--------------------------------------------------------------------------------


### <a name="Constants_link"></a>Constants:
This section describes constants that are relevant for this protocol

Version:
Value: One of:
        "version 0.1 (internal)"

 Description: A String value containing the protocol version.



<a name="Connection_Request_Byte_link"></a>__Connection Request Byte:__
Value: 255 (base 10)
Description: Byte value indicating a connection request.
For usage see: [Connecting](#Connecting_link)


<a name="Connection_Acknowledgement_Byte_link"></a>__Connection Acknowledgement Byte:__
Value: 254 (base 10)
Description: Byte value for acknowledging a connection request
For usage see: [Connecting](#Connecting_link)


<a name="Configuration_Start_Byte_link"></a>__Configuration Start Byte:__
Value: 253 (base 10)
Description: Byte value indicating the start of the Configuration
For usage see: [Connecting](#Connecting_link)


<a name="Configuration_Acknowledgement_Byte_link"></a>__Configuration Acknowledgement Byte:__
Value: 252 (base 10)
Description: Byte value indicating that the Configuration was received and applied successfully
For usage see: [Connecting](#Connecting_link)


<a name="Configuration_Error_Byte_link"></a>Configuration Error Byte:
Value: 251 (base 10)
Description: Byte value indicating that the Configuration was not received and applied successfully
  and the connection attempt will be stopped
Causes:
    - Invalid configuration received by the slave device
For usage see: [Connecting](#Connecting_link)


<a name="Frame_Acknowledgement_Byte_link"></a>Frame Acknowledgement Byte:
Value: 250 (base 10)
Description: Byte value indicating that a Frame was received and applied successfully
For usage see:
  - [Data Transmission](#Data_Transmission_link)
  - [Frame Header](#Frame_Header_link)
  - [Frame Body](#Frame_Body_link)


<a name="Frame_Error_Byte_link"></a>Frame Error Byte:
Value: 249 (base 10)
Description: Byte value indicating that a Frame could not be received and applied successfully.
Causes:
  - Invalid [`frame body size`](#Frame_Body_Size_link) received by the slave device
  - Invalid [`frame body offset`](#Frame_Body_Offset_link) received by the slave device

  :information_source: Additional causes are listed in the documentation of each implementation

For usage see:
  - [Data Transmission](#Data_Transmission_link)
  - [Frame Header](#Frame_Header_link)
  - [Frame Body](#Frame_Body_link)


--------------------------------------------------------------------------------

### <a name="#Configuration_Format_link"></a>Configuration Format:
This section describes the format of the configuration which gets exchanged between the devices when connecting.


The configuration has to be in the following format:

Configuration Start Byte (1 byte)
Protocol Version (String, dynamic size)
device name (String, dynamic size)
number of LEDs connected (integer, 32bit);
data pin (integer, 32bit);
clock pin (integer, 32bit);
Extra values (String, dynamic size)

[Fig. 1_docs_configuration_en] (The configuration format for the ALUP v. 0.1)



<a name="Configuration_Start_Byte_link"></a>Configuration Start Byte
  Description: The configuration start byte marks the start of the configuration when sent over any kind of connection.
               It is always followed by the configuration values according to the protocol configuration format.
               For more details see: [Configuration Start Byte](#Configuration_Start_Byte_link)



#### <a name="Configuration_Values_link"></a>Configuration Values:

<a name="Protocol_Version_link"></a>
Protocol Version
  Type: String (UTF-8)
  Description: the protocol version used by the slave device
  Valid values:
    One of:
      "0.1 (internal)"

<a name="Device_Name_link"></a>
Device name
  Type: String (UTF-8)
  Description: A descriptive name of the slave device; Does not have to be unique
  Valid values: Any String value

<a name="Number_Of_Leds_link"></a>
Number of LEDs connected
  Type: Integer
  Description: The number of LEDs on the connected addressable LED strip
  Valid values: A positive Integer value > 0

<a name="Data_Pin_link"></a>
Data pin
  Type: Integer
  Description: The digital pin at which the data line of the addressable LED strip is connected
  Valid values: A positive Integer value; Should be a valid Data pin of the connected slave device (e.g. Arduino)

<a name="Clock_pin_link"></a>
Clock pin
  Type: Integer
  Description: The digital pin at which the clock line of the addressable LED strip is connected.
    If the connected LED strip does not need a clock signal, this value can be ignored and set to 0
  Valid values: A positive Integer value; Should be a valid Data pin of the connected slave device (e.g. Arduino) or 0 if no clock
    signal is needed by the type of connected LED strip

<a name="Extra_Values_link"></a>
Extra Values
    Type: String (UTF-8)
    Description: A string containing any kind of values; This can be used by any developer to send additional configuration values, but may
    be ignored by the Protocol implementation itself and just passed on to the implementing program or the user itself.
    Valid values: Any String value



--------------------------------------------------------------------------------


### <a name="Frame_link"></a>Frame:
A frame or "data frame" consists of 2 parts:
The frame header and the frame body.

[Fig. 1_docs_frame_en] (Frame structure)

Those parts are structured as stated below:

### <a name="Frame_Header_link"></a>Frame Header:
The frame header consists of 9 bytes:

[Fig. 2_docs_frame_en] (Frame header structure)

<a name="Frame_Body_Size_link"></a>
Byte: 0-3
  - Name: Frame body size
  - Type: Integer
  - Description: The size of the upcoming frame body in bytes
  - Valid values:
    - A positive Number or 0; Has to be a multiple of 3 as the data of the upcoming body is [RGB data](#RGB_Data_link).
      It therefore also has to be <= [`number of LEDs * 3`](#Number_Of_Leds_link) (as specified in the exchanged [configuration](#Configuration_Values_link)).

    - 0 when there is no body (e.g. when the [command](#Command_Byte_link) is [`disconnect`](#Protocol_Commands_link)).

    :warning: Causes a frame error byte to be sent if invalid.

<a name="Frame_Body_Offset_link"></a>
Byte 4-7
  - Name: Frame body offset
  - Type: Integer
  - Description: The offset of the first LED for the data inside the frame body used when applying the frame
  - Valid values: A positive number or 0; Has to be < [`number of LEDs`](#Number_Of_Leds_link).

    :warning: The frame body offset + frame body length /3 should never exceed the [number of LEDs](#Number_Of_Leds_link).

    :warning: Causes a frame error byte to be sent if invalid.

<a name="Command_Byte_link"></a>
Byte: 8
  - Name: Command
  - Type: Byte
  - Description: A byte value specifying a command to be executed before the upcoming [RGB data](#RGB_Data_link) gets applied. They are split into 2 different categories: Protocol commands and Subcommands.
    - Protocol commands are commands defined by the protocol to fulfill special tasks.
    - Subcommands are commands that execute small, user defined programs on the slave device.
  To implement your own subprograms and more, see [Subprograms](#Subprograms_link)

  Valid values: Any byte value (0-255)

  __Commands:__
    - <a name="Protocol_Commands_link"></a>
      __Protocol commands:__
      Decimal value | Name | Description
      :---: | --- | ---
      0 | "None" | Command indicating that no Command should be executed
      1 | "Clear" | Command setting all LED values to 0 before applying the [frame body](#Frame_Body_link). This means, that, if the [frame body](#Frame_Body_link) is empty, all LEDs get set to black, and if the body contains [RGB data](#RGB_Data_link), all unchanged LEDs get set to black.
      2 | "Disconnect" |  Command telling the slave device to [disconnect](#Disconnecting_link)
      3 - 7 | [RESERVED] | Protocol command values reserved for future use.

    - <a name="Subcommands_link"></a>
      __Subcommands:__
      Decimal value | Name | Description
      :---: | --- | ---
      8 - 255 | - | Commands executing the [subprogram](#Subprograms_link) with the corresponding ID on the slave device. The ID is the corresponding value subtracted by an offset of 8. Therefore, a value of 8 executes the subprogram with the ID 0, a Value of 9 executes the subprogram with the ID 1, etc. For adding your own subprograms, see [Subprograms](#Subprograms_link).




### <a name="Frame_Body_link"></a>Frame Body:
The frame body consists of the amount of bytes specified in the [`Frame Body size`](#Frame_Body_Size_link) each representing one R, G or B value of one LED,
starting at the LED with the index 0 up to the LED with an index of (Frame Body size / 3).

[Fig. 3_docs_frame_en] (Frame body structure)

The size of the Body always has to be the Size specified in the Header, otherwise the Data will desynchronize and unexpected behavior will occur
causing the protocol to stop functioning. It therefore has to comply to all specifications stated in the documentation of the [`Frame Body size`](#Frame_Body_Size_link), which state, that the number of bytes always have to be a multiple of 3 and are not allowed to
exceed the [`number of LEDs * 3`](#Number_Of_Leds_link) communicated within the [Connection Process](#Connecting_link). If this is not the case,
a [`Frame Error Byte`](#Frame_Error_Byte_link) will be sent instead of the [`Frame Acknowledgement Byte`](#Frame_Acknowledgement_Byte_link) to
the master device when the frame gets received. For more, see [Data Transmission](#Data_Transmission_link).



## <a name="Subprograms_link"></a>Subprograms
This sections explains subprograms and subcommands.

The protocols supports little subprograms that can be executed whenever a Frame is received by sending a [Subcommand](#Subcommands_link)
with the ID of a Subprogram.

Their goal is to provide a possibility to the end user, a program using the protocol or a protocol implementation itself to
execute small scripts on the slave device whenever a Frame gets received by it.

Those Subprograms have a ID ranging from 0 - 247 (internally represented as 8 - 255, for details see the [Frame Header](#Frame_Header_link) Documentation)
whereas each ID represents a function which can be executed by setting the corresponding ID in the [Command byte](#Command_Byte_link) of the [Frame Header](#Frame_Header_link).
By default, each of those functions is empty (doing nothing when executed).

#### Creating custom Subprograms:
You can create your own subprograms by following the documentation for the code of the used slave device.
Note that some of the IDs may already be occupied by the implementation itself. There is also no guaranty for when the subprogram gets executed
(before, during or after applying the Frame Body to the LEDs) and if it runs linearly or in parallel. All of those factors depend on the implementation
of the Slave Device and are therefore found in the documentation of its code.

:warning: When writing custom Subprograms, please note that, depending on the implementation, those Subprograms may not be executed parallel and therefore
may have a significant impact on performance and frame rates of the LEDs.




## <a name="Future_Ideas_link"></a>Future Ideas
- explicit Normal (non-addressable) led Support
- RGBW support

TODOs until release:
change name
create wiki
add Disclaimer
add licensing