
## Getting Started
The following instructions will get you a Link022 AP and its GW on a single Raspberry Pi device .

### Prerequisites
Have a Raspberry Pi device set up, with a wifi dongle. (Tested with Raspbian)

# Link022 Demo
This doc contains the steps to run a demo.

## Setup demo environment
Here is the structure of the demo setup.
![alt text](./SinglePIlink022.png "Demo setup architecture")

The setup uses network namespaces one for Link022 AP and another for the gnmi client, linked  by veth pari.
  - The Link022 AP (gnmi target) in nsap network namespace with wlan1 and veth1
  - The gateway (gnmi client) in lk22 and it provides reqiured services (dhcp, radius and gnmi client).

### Setup Link022 AP
On the root network namesapce run the commands in this [instruction](../README.md) with the sample certificates (certificated is created with target_name=www.example.com).

### Setup Gateway
On the Link022 AP device, run the commands in this [instruction](../demo/README.md). 
Note: skep go installation step.
