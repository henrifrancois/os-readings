# I/O Devices
How do input/output devices interact with the operating system and how do we efficiently integrate them into existing systems?

## System Architecture
There exists a tradeoff in speed versus length for I/O buses. It is also more expensive to engineer high performance buses. This explains the prototypical structure which we present below. 
![System Architecture](https://raw.githubusercontent.com/nehri97/os-readings/master/sysarch.PNG?token=AHmZJejWKJhh4Al3ow8oFmfUAjLL7Werks5cq_YHwA%3D%3D)

## A canonical device
A device presents an interface which allows the system software to control its operation and an internal structure which defines the abstraction the hardware is providing to the system.
![Canonical Device](https://raw.githubusercontent.com/nehri97/os-readings/master/canonicalDevice.PNG?token=AHmZJcTCd-kfLb-vnGE0hewJxHlYWrVOks5crAFUwA%3D%3D)

## The canonical protocol
A device's state can be described by three registers, status, command and data. The interactions between the device and the OS can be described in four distinct steps.
1. First the OS polls the device for state information.
2. The OS then sends data down to the data register.
3. The OS writes a command to the command register of the device.
4. Wait for device to finish by polling again.

## Lowering CPU overhead with interrupts
This process can be made more efficient with the use of interrupts. When the device is done with some work it raises an interrupt through an interrupt handler. 

## More efficient data movement with DMA
With PIO, the CPU spends too much time moving data to and from devices by hand. The solution to this problem is something we refer to as Direct Memory Access (DMA). A DMA engine is essentially a very specific device within a system that can orchestrate transfers between devices and main memory without much CPU intervention.

## Methods of device interaction
There are two primary ways of interaction between devices.
1. Explicit IO instructions
    They specify a way for the OS to send data to specific device registers.
2. Memory-mapped IO 
    This method makes devices registers available as if they were memory locations.

## Fitting into the OS: The device driver


## A simple IDE Disk Server


## Historical Notes



# Hard Disk Drives

## The interface

## Basic Geometry

## A simple disk drive

## I/O Time

## Disk Scheduling

