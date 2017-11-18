
## Getting Started
The following instructions will get you a Link022 AP on a Raspberry Pi device.

### Prerequisites
Have a Raspberry Pi device set up. (Tested with Raspbian)

# Link022 Demo
This doc contains the steps to run a demo.

## Setup demo environment
Here is the structure of the demo setup.
![alt text](./SinglePIlink022.png "Demo setup architecture")

The setup has two components, linked directly by an ethernet cable.
  - One Raspberry Pi device as the Link022 AP (gnmi target).
  - The other device (such as Raspberry Pi) as the gateway. It provides reqiured services (dhcp, radius and gnmi client).

### Setup Link022 AP
On the Link022 AP device, run the commands in this [instruction](../README.md) with the sample certificates.

### Setup Gateway
On the device for gateway follow the steps below.
