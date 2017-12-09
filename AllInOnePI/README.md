
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

### Prerequesit software
1. Install Golang on Raspberry Pi.
```
wget https://storage.googleapis.com/golang/go1.7.linux-armv6l.tar.gz
sudo tar -C /usr/local -xzf go1.7.linux-armv6l.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
2. Install dependencies for the Link022 AP
```
sudo apt-get install udhcpc bridge-utils hostapd
```
3. Install dependencies for the gateway 
``` 
sudo apt-get install dnsmasq freeradius
```
4. Download GNMI clients for the gateway.
```
export GOPATH=$HOME/go
go get github.com/google/gnxi/gnmi_set
go get github.com/google/gnxi/gnmi_get
```
5. Download gNMI agent for the link022 AP
```
export GOPATH=$HOME/go
go get github.com/google/link022/agent
```

### Download demo folder
From github.com/alshaboti/link022/tree/master/demo
server folder contains gnmi agent (Link022 AP) certificates, client folder contains gnmi client (gateway) certificates. These certificates created for a gnmi target name=www.example.com
Or you can use your own cert.

Demo folder also contains freeradius configuration file pre-configured with default values. 
### Setup Link022 AP
Editing the file /etc/network/interfaces on Pi.
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet manual
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    
# Disable WLAN1 interface.
auto wlan1
iface wlan1 inet static
    address 0.0.0.0

# Repeat for other WLAN interfaces.
```
Note: Reboot the device to make change take effect.

### Create link022 network namespace
```
sudo ip netns add nsap
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns nsap
```
Switch to nsap namesapce and run a process and get its id:
```
sudo ip netns exec nsap bash
# for example get the id of this process
read x &
```
Back in the root namespace add wlan1 to nsap namespace and bring them up.
```
sudo iw phy phy1 set netns <pid>
sudo ip netns exec nsap ip addr add 192.168.11.2/24 dev veth1
sudo ip netns exec nsap ip link set dev veth1 up
sudo ip netns exec nsap ip link set dev wlan1 up
```

### Running Link022 agent
Switch to nsap and run the agent
```
sudo ip netns exec nsap bash
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
sudo env PATH=$PATH agent -ca=<path to ca.crt> -cert=<path to server.crt> -key=<path to server.key> -eth_intf_name=veth1 -wlan_intf_name=wlan1 -gnmi_port=<port number>
```
Note: 
 - Default gnmi_port is 8080. 
 - Make sure the chosen wireless device supports AP mode and has enough capability.

### Running the gateway

```
cd <dir>/demo/util/
./servers_clients.sh
sudo ip netns exec lk22 bash
```

### Push the full configuration to AP
Pushing the entire configuration to AP. It wipes out the existing configuration and applies the incoming one.
Use the [sample configuration](./ap_config.json) here.
```
export PATH=$PATH:/usr/local/go/bin:/home/pi/go/bin
sudo env PATH=$PATH gnmi_set \
-ca=cert/client/ca.crt \
-cert=cert/client/client.crt \
-key=cert/client/client.key \
-target_name=www.example.com \
-target_addr=<link022 AP IP address>:<gnmi port> \
-replace=/:@ap_config.json
```
The veth1 IP is the link022 AP IP address.
After configuration pushed, two WIFI SSIDs should appear.
  - Auth-Link022: The authenticated network with Radius (username: host-authed, password: authedpwd).
  - Guest-Link022: The open network. No authentication requried.
