---
layout: "post"
title: "About Instruction and Control"
date: "2017-03-05 17:10"
author: "Juntong Liu"
catalog: true
---

## General Description

The car and smart phone communicates using a specific instruction system. This system has two sets of instructions, one is master (smart phone) to slave (car), and another is slave to master, using a very similar instruction structure.

This this system has two main characteristics:

- __Dynamic length__: It is able to send complex data like strings and will not waste time on simple instructions. However, this will increase the difficulty in decoding. In addition, the instructions needs to be designed very carefully to avoid semantic confusion.
- __Binary__: Text is only a very little part of the transmission. So binary data is used for efficiency. In this system, most of the numbers are encoded as one byte, able to represent 256 states, which is enough in such environment. While in text, they may occupy 3 bytes each and we need to also deal with dynamic text length, which is troublesome.

All of the instructions follows the following structure:

Section | Length | Description
--------|--------|------------
Head    | 1 byte | Identifier for instruction type
Body    | N/A    | The main content of the instruction
Tail    | 1 byte | End of instruction, should be 0x17

The size of body depends on the type of instruction, some might be fixed and others may be dynamic. This will be given in following sections. From the documentation of the BLE chip, a data chunk contains 20 bytes. Therefore, to include one instruction in one chunk, the maximum length of body is 18 bytes.

## Instruction List (Master to Slave)

#### 0x00 Text

This instruction transmits plain text. If the text is longer than 18 bytes, it is needed to cut into pieces with maximum length of 18 bytes. The end of one string is marked by `\0`, otherwise the receiver needs to wait for next instruction for a complete text.

#### 0x01 Request

This instruction is used to request information from slave. Similar to function call. This instruction is not used while is implemented in the library.

#### 0x02 Speed

This instruction sets the speed of car. The structure of body is given as follows:

Section | Length | Description
--------|--------|------------
Left    | 1 byte | signed char
Right   | 1 byte | signed char

The range is from -127 (full speed backwards) to 127 (full speed forwards).

## Instruction List (Slave to Master)

#### 0x00 Text

The same as the text instruction from master to slave.

#### 0x01 Distance

This instruction is used to report the distance to barrier in the front. The body is an unsigned char with unit of centimeter, 255 meaning nothing detected. The maximum range of the sensor is about 60 cm, so one byte is enough to conduct the information.

#### 0x02 Direction

This instruction is used to report the direction the car is facing compared to the original position. The body is one unsigned char, due to the limit of one byte, the angle is scaled from 360 to 256, for example, 128 represents 360°.

## Conclusion

So this is a very brief introduction. All the implementation is not given to make the document clear. On different device, the implementation varies a lot. On the Arduino side, it is implemented using `Stream` with manual memory management in C++, one the Android side, it is implemented using data chunk reading with help of `ByteBuffer` in Java. The code can be found at [here][1] for Arduino and [here][2] and [here][3] for Android.

[1]: https://github.com/HelloRobotics/ArduinoCar/blob/master/lib/instruction/instruction.h
[2]: https://github.com/HelloRobotics/CarController/blob/master/app/src/main/java/io/github/hellorobotics/carcontroller/core/Instruction.java
[3]: https://github.com/HelloRobotics/CarController/blob/master/app/src/main/java/io/github/hellorobotics/carcontroller/utils/HelperBle.java
