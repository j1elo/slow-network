# slow-network

Simulate a low-bandwidth, high-latency network connection.

## Why

_Everybody should test their code under adverse conditions. Especially networking code._

While working on any network-related software project, such as [WebRTC](https://en.wikipedia.org/wiki/WebRTC) multimedia streaming, code tends to be written and tested under _"laboratory"_ conditions, where local network links are very reliable and transmission speeds are fast and stable. However, these programs should be tested against real-world networks, where data transfer rates are irregular, packets get dropped, latency varies wildly, and lots of other issues might happen.

To try and _simulate_ some of those real-world network conditions, the Linux kernel provides a module called **NetEm**, which is able to introduce several types of misbehavior into the _egress_ queues of a network interface. The NetEm module is used through **tc**, the Linux Traffic Control tool, which uses some not very well documented terminology and requires good knowledge of how it works.

_slow-network_ tries to be a simplification layer over these tools, allowing anyone to apply basic restrictions to a machine's outbound traffic, in an attempt to facilitate testing.

## How

The **tc** tool uses the concepts of `qdisc` (_Queue Discipline_) and `class` in order to handle packet shaping and other types of manipulation over the network devices. _slow-network_ uses those to set up rate limits, latency, jitter, and packet losses.

Note that **these only work for egress traffic** (i.e. **sending packets**) because the queues are outbound only. If you want to act upon both egress and ingress traffic then you may want to place a router host with NetEm between the two test machines, and run NetEm on both interfaces (maybe with differing parameters on each one). The easiest way to achieve this is to run NetEm in a VM to route between two VM networks.

## Running

```
$ ./slow --help
slow - Simulate a low-bandwidth, high-latency network connection.

This script uses the "netem" ([1], [2]) feature available in the Linux kernel,
via the "tc" ([3]) command and the "HTB" ([4]) qdisc.

NetEm is a highly configurable traffic control discipline module which
deliberately delays, drops, and reorders packets.

This only works for egress traffic (i.e. sending packets) because the queues are
outbound only, so you may want to place a router host with NetEm between the
two test machines, and run NetEm on both interfaces (maybe with differing
parameters on each one). The easiest way to achieve this is to run NetEm in
a VM to route between two VM networks.

[1]: https://wiki.linuxfoundation.org/networking/netem
[2]: https://manpages.ubuntu.com/manpages/bionic/en/man8/tc-netem.8.html
[3]: https://manpages.ubuntu.com/manpages/bionic/en/man8/tc.8.html
[4]: https://manpages.ubuntu.com/manpages/bionic/en/man8/tc-htb.8.html

Requirements:

* A Linux operating system with the "tc" traffic control tool.
  In Debian/Ubuntu, install the package "iproute2".

Usage:

slow -i <Interface> { <Preset> | [-r <Rate>] [-d <Delay>] [-j <Jitter>] [-l <Loss>] }
slow -i <Interface> reset
slow -i <Interface> status

Arguments:

-i <Interface>
  Name of the network interface to be configured.
  E.g.: "eth0", "enp0s25", "wlan0", etc.
  Optional. Default: `eth0`.

<Preset>
  Preconfigured set of <Rate>, <Delay> and <Loss> percentage. This
  parameter is a string corresponding to one of the following categories:

  -----------------------------------------------
  | <Preset>     |    <Rate> | <Delay> | <Loss> |
  ---------------------------|-------------------
  | "2.5g"       |           |         |        |
  | "gprs"       |   50 kbps |  400 ms |   5 %  |
  | "edge"       |           |         |        |
  -----------------------------------------------
  | "3g"         |  700 kbps |  300 ms |   5 %  |
  -----------------------------------------------
  | "4g"         |  4.5 mbps |  120 ms |   1 %  |
  -----------------------------------------------
  | "modem-9600" | 9600 bps  |  200 ms |   0 %  |
  -----------------------------------------------
  | "modem-56k"  |   56 kbps |  120 ms |   0 %  |
  -----------------------------------------------
  | "t1"         | 1500 kbps |   20 ms |   0 %  |
  -----------------------------------------------
  | "t3"         |   45 mbps |   10 ms |   0 %  |
  -----------------------------------------------
  | "dsl"        |    2 mbps |   60 ms |   0 %  |
  -----------------------------------------------
  | "cablemodem" |   10 mbps |   50 ms |   0 %  |
  -----------------------------------------------
  | "wifi-b"     |   11 mbps |   10 ms |   0 %  |
  -----------------------------------------------
  | "wifi-g"     |   54 mbps |    5 ms |   0 %  |
  -----------------------------------------------
  | "wifi-n"     |  110 mbps |    2 ms |   0 %  |
  -----------------------------------------------
  | "eth-10"     |   10 mbps |    1 ms |   0 %  |
  -----------------------------------------------
  | "eth-100"    |  100 mbps |    1 ms |   0 %  |
  -----------------------------------------------
  | "eth-1000"   | 1000 mbps |    1 ms |   0 %  |
  -----------------------------------------------
  | "vsat"       |    5 mbps |  500 ms |   0 %  |
  -----------------------------------------------
  | "vsat-busy"  |    2 mbps |  800 ms |   0 %  |
  -----------------------------------------------

-r <Rate>
  Maximum bandwidth that the connection must be restricted to,
  in kbps (kilobits per second).
  Optional. Default: `500`.

-d <Delay>
  Latency forced into the packet transmission, in milliseconds.
  Optional. Default: `0`.

-j <Jitter>
  Jitter introduced in the rate of package transmission, in milliseconds.
  Optional. Default: `0`.

-l <Loss>
  Average emulated packet drop rate, as a loss percentage.
  Optional. Default: `0.0`.
```

## More reading

- The Linux Documentation Project (TLDP): [Traffic Control HOWTO](https://tldp.org/HOWTO/Traffic-Control-HOWTO/index.html)
- The Linux Foundation: [netem](https://wiki.linuxfoundation.org/networking/netem) | [Archive](https://web.archive.org/web/20191023125240/https://wiki.linuxfoundation.org/networking/netem)
- Use Linux Traffic Control as impairment node in a test environment
  - [Part 1](https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-1) | [Archive](https://web.archive.org/web/20191023121720/https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-1/)
  - [Part 2](https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-2) | [Archive](https://web.archive.org/web/20191023121821/https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-2)
  - [Part 3](https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-3) | [Archive](https://web.archive.org/web/20191023121905/https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-3)
- Ubuntu Manpages:
  - [tc - Traffic Control](https://manpages.ubuntu.com/manpages/bionic/en/man8/tc.8.html)
  - [NetEm - Network Emulator](https://manpages.ubuntu.com/manpages/bionic/en/man8/tc-netem.8.html)
  - [HTB - Hierarchy Token Bucket](https://manpages.ubuntu.com/manpages/bionic/en/man8/tc-htb.8.html)
- [HTB User Guide](http://luxik.cdi.cz/~devik/qos/htb/manual/userg.htm)
- Simulating low bandwidths: how to make sure your apps work in the field | [Archive](https://web.archive.org/web/20121104183730/http://blog.aptivate.org/2010/01/23/make-sure-your-apps-work-in-the-field/)

## Credits

This tool was conceptualized after reading several sources on the topic of
network shaping tools available in Linux, and the actual implementation was
inspired by the work of Richard Bullington-McGuire, as found here:
https://gist.github.com/obscurerichard/3740206
