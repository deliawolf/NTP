# NTP

NTP Versions
NTPv4 is an extension of NTPv3 and provides the following capabilities:

- NTPv4 supports IPv6, making NTP time synchronization possible over IPv6.
- Security is improved over NTPv3. NTPv4 provides a whole security framework that is based on public key cryptography and standard X.509 certificates.
- Using specific multicast groups, NTPv4 can automatically calculate its time-distribution hierarchy through an entire network. NTPv4 automatically configures the hierarchy of the servers to achieve the best time accuracy for the lowest bandwidth cost.
- In NTPv4 for IPv6, IPv6 multicast messages instead of IPv4 broadcast messages are used to send and receive clock updates.

# NTP Modes

NTP can operate in these four different modes that provide you with the flexibility for configuring time synchronization in your network:
- Server
- Client
- Peer
- Broadcast/multicast

## Server
It provides accurate time information to clients.

## Client
It synchronizes its time to the server. This mode is most suited for file server and workstation clients that are not required to provide any form of time synchronization to other local clients. It can also provide accurate time to other devices.

The server and client modes are usually combined with Cisco network devices. A device that is an NTP client can act as an NTP server to another device. The client/server mode is the most common internet configuration. A client sends a request to the server and expects a reply at some future time. This process could also be called a poll operation because the client polls the time and authentication data from the server.

A client is configured in client mode by using the server command and specifying the DNS name or address of the server. The server requires no prior configuration. In a common client/server model, a client sends an NTP message to one or more servers and processes the replies as received. The server exchanges addresses and ports, overwrites certain fields in the message, recalculates the checksum, and returns the message immediately. The information that is included in the NTP message allows the client to determine the server time regarding local time and adjust the local clock accordingly. In addition, the message includes information to calculate the expected timekeeping accuracy and reliability, and to choose the best server.

## Peer
Peers exchange time synchronization information. The peer mode is also commonly known as symmetric mode. It is intended for configurations where a group of low stratum peers operate as mutual backups for each other.

Each peer operates with one or more primary reference sources, such as a radio clock or a subset of reliable secondary servers. If one of the peers loses all the reference sources or ceases operation, the other peers automatically reconfigure so that time values can flow from the surviving peers to all the others in the group. In some contexts, this operation is described as push-pull, in that the peer either pulls or pushes the time and values depending on the configuration.

Symmetric modes are most often used between two or more servers operating as a mutually redundant group and are configured with the ntp peer command. In these modes, the servers in the group members arrange the synchronization paths for maximum performance, depending on network jitter and propagation delay. If one or more of the group members fail, the remaining members automatically reconfigure as required.

## Broadcast/multicast
It is a special "push" mode of NTP server. Where the requirements in accuracy and reliability are modest, clients can be configured to use broadcast or multicast modes. Normally, servers with dependent clients do not utilize these modes. The advantage is that clients do not need to be configured for a specific server, allowing all operating clients to use the same configuration file.

The broadcast mode requires a broadcast server on the same subnet. Because routers do not propogate broadcast messages, only broadcast servers on the same subnet are used. Broadcast mode is intended for configurations that involve one or a few servers and a potentially large client population. On a Cisco device, a broadcast server is configured by using the broadcast command with a local subnet address. A Cisco device acting as a broadcast client is configured by using the broadcast client command, allowing the device to respond to broadcast messages that are received on any interface.

# Configuring and Verifying NTP

![NTP](https://raw.githubusercontent.com/deliawolf/NTP/main/NTP.png)
```
Central(config)# ntp master 2
Branch(config)# ntp server 192.168.1.1
SW1(config)# ntp server 10.1.1.1
```

The stratum value is a number from 1 to 15. The lowest stratum value indicates a higher NTP priority. It also indicates the NTP stratum number that the system will claim.

Some People considering using interface loopback for NTP.

Configure the Central router as an NTP server:
```
Central(config)# interface Loopback 10
Central(config-if)# ip address 192.168.255.1 255.255.255.0
Central(config)# ntp master 2
Central(config)# ntp source Loopback10
```
Configure the Branch 1 router as an NTP client, which will synchronize its time with the Central router:
```
Branch1(config)# ntp server 192.168.255.1
```
Configure the Branch 2 router as an NTP client, which will synchronize its time with the Central router.
```
Branch2(config)# ntp server 192.168.255.1
```
Use the show ntp associations and the show ntp status commands to verify your configuration.

## Securing NTP

Configure NTP authentication on the NTP server.
```
NTPServer(config)# ntp authentication -key 1 md5 MyPassword
NTPServer(config)# ntp authenticate
NTPServer(config)# ntp trusted-key 1
```
Configure NTP authentication on the NTP client.
```
NTPClient(config)# ntp authentication -key 1 md5 MyPassword
NTPClient(config)# ntp authenticate
NTPClient(config)# ntp trusted-key 1
NTPClient(config)# ntp server 10.0.1.22 key 1
```
Access lists on the NTP server ensure that only authorized clients can synchronize with it.

Configure Core1 to peer with only a specified IP address.
```
Corel(config)# access-list 1 permit host 10.1.0.15
Corel(config)# ntp access-group peer 1
```
Configure Core1 to answer synchronization requests from only 10.1.0.0/16 subnet devices.
```
Corel(config)# access-list 1 permit 10.1.0.0.0.0.255.255
Corel(config)# ntp access-group serve-only 1
```

I understand that this might seem a bit confusing. Let's break it down:

1. peer: This means that a device can both ask for the time from the devices that pass the access list and also provide the time to them. In other words, it's a two-way relationship. The devices can synchronize their time with each other.

2. serve: This means that a device can provide the time to the devices that pass the access list, but it won't synchronize its own time with them. So, it's a one-way relationship, only serving time to others.

3. serve-only: This further restricts the "serve" condition. The device will only respond to time synchronization requests but won't respond to control queries. A control query is a request for other types of information, like system variables or the current time.

4. query-only: This is the opposite of "serve-only". The device will only respond to control queries but won't respond to time synchronization requests.

These restrictions are used to control the level of interaction a device can have with others in the context of NTP. They can be implemented to enhance the security and integrity of time synchronization within a network.

If your device is configured as the NTP primary, then you must allow access to the source IP address of 127.127.x.1. The reason is because 127.127.x.1 is the internal server that is created by the ntp master command. The value of the third octet varies between platforms.

After you secure the NTP server with access lists, make sure to check if the clients still have their clocks that are synchronized via NTP by using the show ntp status command. You can verify which IP address was assigned to the internal server by using the show ntp associations command.
