<!--
  Copyright (c) 2024 Eclipse Foundation
 
  This program and the accompanying materials are made available 
  under the terms of the MIT license which is available at
  https://opensource.org/license/mit.
 
  SPDX-License-Identifier: MIT
 
  Contributors: 
      Frédéric Desbiens - Initial version.
      Andy Riexinger - documentation for Mac M1
-->

# Eclipse ThreadX hackathon challenge

This starter application is an adaptation of a sample developed originally by Microsoft. The original code can be found here:

[https://github.com/eclipse-threadx/getting-started](https://github.com/eclipse-threadx/getting-started)

I removed code providing support for Azure IoT Cloud. The only board supported is the MXChip AZ3166.

## Cloning this repository
Eclipse ThreadX and Eclipse ThreadX NetX Duo are included as submodules.

When cloning, you must specify the `--recurse-submodules` option to get the code for the submodules. If you forget this option, just run the following commands in the root folder of your clone. 

```
git submodule init
git submodule update --recursive
```

## Prerequisites

### Computer
Theoretically, any recent laptop running Windows 11, Linux, or MacOS should do.

I tested on the following environments:
- Windows 11 24H2 (Version 10.0.26100.2161)
- Ubuntu 22.04.5 LTS (Windows Subsystem for Linux version 2.3.24.0)

Also tested on:
- Mac M1, macOS Sonoma 14.6.1 (23G93), native
- Mac M1, macOS Sonoma 14.6.1 (23G93), Parallels Desktop for Mac Version 17.1.7 (51588), Ubuntu 22.04.5 LTS 

### Evaluation board
This sample is preconfigured to work with the [MXChip AZ3166 board](https://docs.mxchip.com/en/nr6ggk/blyezpv6gkqicywi.html).

This board can run Eclipse ThreadX but is also Arduino compatible.

The board features a [STM32F412RG MCU](https://www.st.com/en/microcontrollers-microprocessors/stm32f412rg.html) from STMicroelectronics. The MCU is clocked at 100Mhz and comes with 1 Mbyte of flash memory and 256Kbytes of SRAM.

The AZ3166 has the following hardware onboard:
- 128*64 dot matrix OLED display: VGM128064
- RGB LED lights controlled by P9813:
- Temperature and Humidity Sensor: HTS221
- Atmospheric pressure sensor: LPS22HB
- Bidirectional mono audio ADC/DAC: NAU88C10, microphone and 3.5mm headphone jack
- Six-axis accelerometer: LSM6DSL
- Geomagnetic sensor: LIS2MDL
- DC motor
- Two buttons
- three LED indicators

The board also features WiFi connectivity. However, only 2.4Ghz networks are supported.

This starter application is preconfigured to provide access to some, but not all, of the peripherals above.

### Developer tools
In terms of tooling, all you need to work on the challenge is CMake, Ninja, and a suitable C compiler. Naturally, having Git installed could help as well. ;-)

The source code for ThreadX and related modules is very portable and compliant with all "required" and "mandatory" rules of MISRA-C:2004 and MISRA C:2012. Most modern C compilers should be able to compile it. The official build pipelines rely on Arm's embedded GNU toolchain.

Below are instructions to install the tools.

**Ubuntu**
```
apt install ninja-build cmake 
```

Then, download and install Arm's embedded GNU toolchain, available at [https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)

The following will download and unpack version 13.3.rel1 of the software to `/opt`.
``` 
wget https://developer.arm.com/-/media/Files/downloads/gnu/13.3.rel1/binrel/arm-gnu-toolchain-13.3.rel1-x86_64-arm-none-eabi.tar.xz
sudo tar xJf arm-gnu-toolchain-13.3.rel1-x86_64-arm-none-eabi.tar.xz -C /opt
```

To test, you can run the following commands:
```
export PATH=$PATH:/opt/arm-gnu-toolchain-13.3.rel1-x86_64-arm-none-eabi/bin
arm-none-eabi-gcc --version
```

For Mac M1, I used version 14.2.rel1 so you can install: 
- Mac M1 native: [arm-gnu-toolchain-14.2.rel1-darwin-arm64-arm-none-eabi.pkg](https://developer.arm.com/-/media/Files/downloads/gnu/14.2.rel1/binrel/arm-gnu-toolchain-14.2.rel1-darwin-arm64-arm-none-eabi.pkg)
- Ubuntu 22.04.5 on Mac M1 via Parallels Desktop: [arm-gnu-toolchain-14.2.rel1-aarch64-arm-none-eabi.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu/14.2.rel1/binrel/arm-gnu-toolchain-14.2.rel1-aarch64-arm-none-eabi.tar.xz)


**Windows**
```
winget install --id=Arm.GnuArmEmbeddedToolchain  -e
winget install --id=Ninja-build.Ninja  -e
winget install --id=Kitware.CMake  -e
```

## Compiling and running the application
To compile the application, simply execute the relevant script found in the `MXChip/AZ3166/scripts` folder.

If you run into some problems like:
```
bash: ./build.sh: Permission denied
```
Don´t forget to change the permissions of the build script. For Mac and Linux following command can help:
```
chmod +x build.sh
```

To deploy your code on the AZ3166, just plug the board on your computer. When you do so, this will create a virtual drive and a serial port over USB. On my Windows laptop, the board appears as the `d:` drive and the `COM4` serial port. 

Once compilation is finished, you will find the executable in the `MXChip/AZ3166/build/app` folder. The default filename is `mxchip_threadx.bin`. Just copy that file to the virtual drive and the AZ3166's boot loader will reset the board and execute your code. There is also a deploy script in the `MXChip/AZ3166/scripts` folder.

You can use any terminal application to connect to the serial port and monitor your application's output. Personally, I use Tera Term, which you can install using `winget`. Just make sure you set the baud rate to **115,200**.
For Mac M1 I used [SerialTools](https://apps.apple.com/de/app/serialtools/id611021963?mt=12) which you can install via the App Store.

If you deployed this application without any changes, you will get the following output in your terminal:

> ```
> Scanning I2C bus
> ..........................0x1a...0x1e.0x20...........................0x3c...............................0x5c..0x5f..........0x6a.....................
>
> Starting Eclipse ThreadX thread
> 
>
> Initializing WiFi
> ERROR: wifi_ssid is empty
> ERROR: wifi_init (0x00000043)
> ERROR: Failed to initialize the network (0x00000043)
> ```

### WiFi configuration
To connect the board to a WiFi network, edit the following constants found in `cloud_config.h`:

- `HOSTNAME`
- `WIFI_SSID`
- `WIFI_PASSWORD`

Make sure to select an appropriate value for `WIFI_MODE` as well.

If the WiFi is properly congiured, you will get the output below at application startup:

> ```
> Initializing WiFi
>       MAC address: C8:93:46:88:6F:9E
> SUCCESS: WiFi initialized
> ```

## Challenge
The ThreadX and beyond challenge is divided in two separate phases. 

### Phase 1: ThreadX basics
The starter application provided in this repository simply initiates the board and WiFi connectivity.

The first step is to study and understand:
1. The board's hardware features.
2. The ThreadX and NetX Duo APIs.

NetX Duo is a comprehensive TCP/IPv4 and v6 network stack. It offers built-in HTTP and MQTT clients, among other things.

For phase 1, your goal is to acquire some sensor data and publish it somewhere over HTTP (RESTFul call) or MQTT. 

You will get additional points for leveraging the board's LEDs, screen, and buttons.

If you need a local MQTT broker for testing, I recommend using Eclipse Mosquitto. Here is how to install it.

You will find the Eclipse ThreadX documentation here: [https://github.com/eclipse-threadx/rtos-docs](https://github.com/eclipse-threadx/rtos-docs)

**Ubuntu**
```
apt install mosquitto mosquitto mosquitto-clients
```

The mosquitto-clients package installs the `mosquitto_pub`and `mosquitto_sub` utilities.

**Windows**
```
winget install --id=EclipseFoundation.Mosquitto
```

The Windows installer provides the `mosquitto_pub`and `mosquitto_sub` utilities. The default installation directory is `Program Files\mosquittoC:\Program Files\mosquitto`.

### Phase 2: Beyond ThreadX
If you reach this point, congratulations! You have a basic but functional ThreadX application.

For the second phase of the challenge, you should try to demonstrate integration with other [Eclipse SDV projects](https://sdv.eclipse.org/projects/). This should be fun, as most of them expose a RESTFul API or can interact with MQTT brokers. 

You could also build ThreadX support for specific Eclipse SDV technologies. For example, [Eclipse zenoh](https://github.com/eclipse-zenoh) and [Eclipse uProtocol](https://github.com/orgs/eclipse-uprotocol) are two fundamental technologies that could be ported to work on the top of the ThreadX kernel and NetX Duo.

Finally, there are other projects of interest in the broader Eclipse open source ecosystem. For example:

- [Eclipse 4diac](https://eclipse.dev/4diac) is a mature and popular framework to build Programmable Logic Controllers (PLCs). You could leverage the 4diac Forte runtime on ThreadX and have a project illustrating the car factory of the future. 
- [Eclipse Hono](https://eclipse.dev/hono/) provides remote service interfaces for connecting large numbers of IoT devices to a backend and interacting with them in a uniform way regardless of the device communication protocol. You could use it to manage your evaluation boards through the Hono device registry.
- [Eclipse Ditto](https://eclipse.dev/ditto/) can be used to create digital twins of your boards and offers an extensive RESTFul API.

Where will your journey beyond ThreadX bring you? 