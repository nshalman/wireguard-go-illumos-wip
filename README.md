# illumos port of wireguard-go

This is a work-in-progress port of the Go version of
[WireGuard](https://www.wireguard.com/).  Basic functionality has been verified
on a current SmartOS system, which includes by default the necessary [TUN/TAP
Driver](http://www.whiteboard.ne.jp/~admin2/tuntap/).

In addition to the `wireguard-go` program built from this repository, you will
need to build the `wg` tool from the upstream
[WireGuard.git](https://git.zx2c4.com/WireGuard/); e.g.,

```
$ git clone https://git.zx2c4.com/WireGuard
$ cd WireGuard/src/tools
$ make LDLIBS='-lnsl -lsocket'
  CC      /ws/wireguard/WireGuard/src/tools/wg.o
  CC      /ws/wireguard/WireGuard/src/tools/set.o
  CC      /ws/wireguard/WireGuard/src/tools/mnlg.o
  CC      /ws/wireguard/WireGuard/src/tools/pubkey.o
  CC      /ws/wireguard/WireGuard/src/tools/showconf.o
  CC      /ws/wireguard/WireGuard/src/tools/genkey.o
  CC      /ws/wireguard/WireGuard/src/tools/setconf.o
  CC      /ws/wireguard/WireGuard/src/tools/curve25519.o
  CC      /ws/wireguard/WireGuard/src/tools/encoding.o
  CC      /ws/wireguard/WireGuard/src/tools/ipc.o
  CC      /ws/wireguard/WireGuard/src/tools/terminal.o
  CC      /ws/wireguard/WireGuard/src/tools/config.o
  CC      /ws/wireguard/WireGuard/src/tools/show.o
  LD      /ws/wireguard/WireGuard/src/tools/wg
$ ./wg
interface: tun0
```

At present, this port of `wireguard-go` must be run in the foreground (i.e.,
using `-f`) and only accepts the interface name `tun`.  A dynamic `tun[0-9]+`
device will be created for the duration of the process.

```
# ./wireguard-go -f tun
WARNING WARNING WARNING WARNING WARNING WARNING WARNING
W                                                     G
W   This is alpha software. It will very likely not   G
W   do what it is supposed to do, and things may go   G
W   horribly wrong. You have been warned. Proceed     G
W   at your own risk.                                 G
W                                                     G
WARNING WARNING WARNING WARNING WARNING WARNING WARNING
device tun0
ip_muxid 131
INFO: (tun0) 2019/03/26 08:26:38 Starting wireguard-go version 0.0.20181222
INFO: (tun0) 2019/03/26 08:26:38 Interface set up
INFO: (tun0) 2019/03/26 08:26:38 Device started
INFO: (tun0) 2019/03/26 08:26:38 UAPI listener started
```

Once the daemon is running, you'll need to configure the IP address and any
routes on your `tun` interface.  Due to the way point-to-point links need to be
configured on illumos at the moment, you'll need to set aside a "destination"
address to represent the remote side of the tunnel.  In the example below I've
used `5.0.1.1` as the IP address of this system, and `5.0.1.2` as the fake
destination address.  You can use the same fake destination IP on all systems
but no system on the VPN can use it as its actual IP.

```
# ifconfig tun0 5.0.1.1 5.0.1.2 netmask 255.255.255.255 mtu 1300 up
# ifconfig tun0
tun0: flags=10010008d1<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST,IPv4,FIXEDMTU> mtu 1300 index 34
  inet 5.0.1.1 --> 5.0.1.2 netmask ffffffff
  ether 80:1c:25:4e:24:fe

# route add 5.0.1.0/24 5.0.1.2

# wg setconf tun0 tun0.conf
# wg
interface: tun0
  public key: <....>
  private key: (hidden)
  listening port: 51820

peer: <....>
  endpoint: 10.0.0.1:51820
  allowed ips: 5.0.1.3/32
  latest handshake: 1 minute, 16 seconds ago
  transfer: 331.72 KiB received, 12.15 MiB sent
```

The rest of the README is from the upstream repository:

# Go Implementation of [WireGuard](https://www.wireguard.com/)
# Temp fork of git.zx2c4.com/wireguard-go

This repository is a staging for for https://git.zx2c4.com/wireguard-go.
It sometimes carries experimental patches before they go upstream.

_WARNING:_ The main branch is frequently rebased.
Do not depend on this repository.

## License

    Copyright (C) 2017-2020 WireGuard LLC. All Rights Reserved.

    Permission is hereby granted, free of charge, to any person obtaining a copy of
    this software and associated documentation files (the "Software"), to deal in
    the Software without restriction, including without limitation the rights to
    use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
    of the Software, and to permit persons to whom the Software is furnished to do
    so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
