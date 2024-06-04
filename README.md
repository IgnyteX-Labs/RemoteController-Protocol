# RemoteController-Protocol v1.0.0

A communication protocol for RemoteControllers. This enables cross-platform applications using RemoteController libraries for different programming languages.

## API Guidelines

Every RemoteController implementation should follow an api simliar to the following:

There should be a `RemoteController` class (or similar). This class should setup and hold everything that the RC needs. Additionally, it should hold an Instance of the Abstract-Type `Connection`. The abstract `Connection` class (or similar) provides a unified interface to receive and transmit binary payloads. The RemoteController only uses this abstract connection to receive and transmit its payloads. This allows for a large variety of different connection possibilites with the same RemoteController class and api

### Connection Interfaces

Example `Connection` Implementations:
* SPIConnection (A wired connection interface)
* RF24Connection (A radio connection using the cheap NRF24L01 modules)
* BluetoothConnection
* ...

## Data Types

Every RemoteController implemenation has to implement the following two data types and encode them as specified.

### Commands

`Commands` should be the primary data type used in RemoteControllers. A command consists of (1) an `enum-like instruction` and (2) a `throttle float (32-bit) value` to go along with that instruction. E.g.: `GoForward` and 200.1, this would signal the other RemoteController to perform the GoForward action with a throttle of 200.1. 

<table>
    <thead>
        <tr>
            <th>Type</th>
            <th>Structure</th>
            <th>Recommended Type Representation</th>
            <th>Required Encoded Type</th>
            <th>Required Encoded Size</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=2>Command</td>
            <td>Instruction</td>
            <td>Enum</td>
            <td>unsigned 8-bit integer</td>
            <td>8 bit (0-255)</td>
        </tr>
        <tr>
            <td>Throttle</td>
            <td>Float</td>
            <td>32-bit float</td>
            <td>32 bit</td>
        </tr>
    </tbody>
</table>

As this table already suggests how exactly a Command is represented in a RemoteController implementation is not specified, it should be the most conveninet and practical representation that the programming language allows. 

It is, however, __crucial__ that the command before being transmitted is encoded in the follwing format:

When transmitting commands the first 16 bits (2 bytes) have to be the following `Command Identifier` bits:

```
0xEEAF
```

This makes the receiver know that the payload after these 16 identifier bits, is of Command Type. The payload following the Command Identifier consists of a binary array of 40-bit (5-byte) Commands.

A `Command` is encoded as follows: The first 8 bits represent the Instruction (thus 256 different instructions are possible), the following 32 bits represent a throttle value of type float, in total a command is of 40 bit or 5 byte size. 
| Instruction | Throttle |
|---|---|
| 8 bits | 32 bits |
| unsigned int (0-255) | float |


### Binaray Payloads

Binary payloads allow the user to send and receive whatever they want, at the cost of having to deal with the binary payloads themselves. Every RemoteController implementation has to include an api to send and receive pure binary payloads. 
