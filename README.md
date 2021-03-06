# air-blast
WI-FI Scanner and DeAuthing-Attack Software

[Prerequisites](#Prerequisites)

[Installation](#Installation)

[Usage](#Usage)

[Limitations](#Limitations)

## The Hell Is This??
This is a commandline tool that can perform the infamous
wifi [deauthentication attack](https://en.wikipedia.org/wiki/Wi-Fi_deauthentication_attack). 
Currently there are three modes of operation:

1. Discovery of active wifi sessions in the vicinity.
2. Deauthentication/Disassociation of a single client.
3. DoS-style mass deauthentication/disassociation of all discovered sessions.
4. Simple WI-FI-SCAN Option.

The program uses the standard pcap routines to read packets from a capture interface
and write forged ones to the wire. Mode 1 implements a minimal parser with the purpose
of identifying potential targets. Mode 3 builds on mode 1 by additionally dispatching 
frame injection threads for every connection discovered.

As of version *1.1*, the parsing routines also use a copy of the IEEE Registry to automatically identify
card vendors by their OUIs.

## Prerequisites
This program was designed for Debian Linux platforms, other operating systems are not supported.

##### Hardware requirements
Your wireless card should support capturing packets in [promiscuous mode](https://en.wikipedia.org/wiki/Promiscuous_mode) 
as well as packet injection, ideally **at the same time**. Otherwise errors are likely to occur.

##### Library Dependencies
* [libpcap](http://www.tcpdump.org/) *(packet capture and injection)*.
* [libmnl](https://www.netfilter.org/projects/libmnl/index.html) *(used for wifi-scan)*.

Additionally, the parallelism of mode 3 is implemented via Posix Threads. So make sure an implementation
of the pthreads API is present.

## Installation
sudo dpkg -i package.deb

##### Quickstart
Go to the [Release](https://github.com/Smiril/air-blast/releases) Page and locate the latest version. 

## Usage
The program is meant to be run from the commandline. You will most likely need elevated privileges in order
to create and perform I/O operations on the pcap pseudo-interfaces. You can toggle between the different
modes by supplying the appropriate parameter:

#### -m 
Monitoring active sessions that are close by. Command format:

`air-blast -i <myNIC> -s <spoofed client mac> [ -c max_connections ] -m`

The program will first perform a scan (via iwlib) to get a list of the
local access points. It will then watch for data frames sent to the stations
and print the source mac addresses to the console.

#### -g
Monitoring active sessions that are close by. Command format:

`air-blast -i <myNIC> -s <spoofed client mac> [ -b bssid ] [ -c max_connections ] -g`

The program will first perform a scan (via wifi-scan lib) to get a list of the
local access points. It will then watch for data frames sent to the clients
and print the source mac addresses to the console.

#### -d
Deauthentication of a single client device. Command format:

`air-blast -i <myNIC> -s <spoofed client mac> -b <bssid> -d` 

This mode can be used to target a single machine whose mac address is already known.
`<spoofed client mac>` should be a string of the form `XX:XX:XX:XX:XX:XX`. Likewise, `<bssid>` 
is supposed to be the mac address of the access point (in the same format).

#### -f
Automatic deauthentication of all discovered sessions. Command format:

`air-blast -i <myNIC> [ -c max_connections ] -f`

This mode acts as an extension of the watch functionality. 
Instead of just monitoring data frames, the program will actively block clients at first sight. 

**Warning:** This mode is anything but subtle. Unless the local network admins are on vacation
(and have taken the entire IDS with them) there's no way that sudden deauthentication flood will
pass unnoticed.

#### Options

###### Required
* `-i <myNIC>`: This parameter is mandatory for all three modes and determines which wireless card to use.
*myNIC* should be the device name as it appears in ifconfig/ip, e.g. `wlan0`.  

###### Optional
* `-c max_connections`: When using mode 1 or 3, *airkick* limits itself to a conservative default 
(tweakable at compile time) of tracked connections. This parameter can be used to increase
the tracking limit. Beware, though, that the impact of doing so might be significant, especially
in flood mode (every thread gets its own packet buffer and pcap handle). Use this option at your own risk.

## Limitations

* *air-blast* is by no means a complete wireless suite. If you need complete wireless auditing functionality, a tool
like [aircrack-ng](https://www.aircrack-ng.org/) will serve you far better than this steaming pile of code.

* It might be possible to build this tool on BSD systems, but no testing has been done so far.

* With the (not so) recent [updates](https://en.wikipedia.org/wiki/IEEE_802.11w-2009) to the IEEE 802.11 standard and 
WPA3 on the horizon *air-blast* might already be obsolete. Of course, protocol updates always carry with them a chance
of new loopholes (which translates to feature extensions for projects like this). Still, the wifi landscape is ever changing 
and in its current shape this little toy might soon be a relic of the "good old days".

## License
This project is licensed under the [MIT].(LICENSE) 
