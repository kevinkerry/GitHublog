title: Bluetooth on Mac OS
date: 2014-07-30 20:25:59
tags: [Bluetooth, Mac, Objective-C]

---

## How to Develop Bluetooth App on Mac OS

<!--more-->

## The Bluetooth Protocol Stack

![Bluetooth Protocol Stack](/img/bt_protocol_stack.gif)

## The OS X Bluetooth Protocol Stack

![OSX Bluetooth Protocol Stack](/img/bt_mosx_protocol_stack.gif)

#### HCI

A boundary between the lower layers of the Bluetooth protocol stack and the upper layers.

Transmits data and commands from the layers above to the Bluetooth module below. Conversely, the HCI layer receives events from the Bluetooth module and transmits them to the upper layers.

#### *L2CAP*

Employs the concept of channels to keep track of where data packets come from and where they should go.

- Multiplexing of data channels.
- Segmentation and reassembly of data packets to conform to a device’s maximum packet size.
- Support for different channel types and channel IDs, such as RFCOMM.

#### *SDP (service discovery protocol)*

Defines a service as any feature that is usable by another (remote) Bluetooth device.

Uses L2CAP channel.

When the client finds the desired service, it requests a separate connection to use the service. An SDP server maintains its own SDP database, which is a set of service records that describe the services the server offers.

#### *RFCOMM*

A serial-port emulation protocol. Its primary mission is to make a data channel appear as a serial port.

#### *OBEX (object exchange)*

A transfer protocol that defines data objects and a communication protocol two devices can use to easily exchange those objects.

Uses RFCOMM channel.

## Main Frameworks

- #### IOBluetoothUI.framework

	Provide a consistent user interface in your applications. Based on IOBluetooth.framework.
	1. Device browser controller
	2. Service selector controller
	3. Pairing controller

- #### IOBluetooth.framework

	Perform Bluetooth-specific tasks:
	
	1. Create and destroy connections to remote devices
	2. Discover services on a remote device
	3. Perform data transfers over various channels
	4. Receive Bluetooth-specific status codes or messages
	
![IOBluetooth](/img/bt_classes_in_stack.gif)

- #### CoreBluetooth.framwork

	1. For OSX, embedded in IOBluetooth.framework
	2. iOS
	3. Only supports BLE
	
![CoreBluetooth](/img/bt_corebluetooth.png)

``` objc
	CBCentralManager <-> CBPeripheralManager
```
