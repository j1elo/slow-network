# slow-network

Simulate a low-bandwidth, high-latency network connection.



## Why

In the area of [WebRTC](https://en.wikipedia.org/wiki/WebRTC) streaming, as with any other network-related software project, code tends to be written and initially tested under *"laboratory"* conditions, where local network links are very reliable and transmission speeds are fast and stable.

However, before actually releasing and deploying the software, there is a constant need of testing every implementation with real-world networking conditions, where data transfer rates are irregular, packets get dropped, latency varies wildly, and lots of other conditions that are not even remotely visible in the laboratory tests.

As a way to try and *simulate* some of those bad network conditions that happen in real-world deployments, the Linux kernel provides the **NetEm** module, which is able to introduce several types of misbehavior into the *egress* queues of a network device.

The NetEm module is used through **tc**, the Linux Traffic Control tool. This brings with itself a lot of confusing (and not very well documented) terminology and required knowledge, so this tool tries to be a simplification layer that allows someone to apply basic restrictions to a machine's outbound traffic, in an attempt to facilitate testing.

## How

The **tc** tool uses the concept of `qdisc` (*Queue Discipline*) and `class` in order to handle packet shaping and other types of manipulation over the network devices. This tool uses those to set up rate limits, latency, and packet losses.

Note that **these only work for egress traffic** (i.e. **sending packets**) because the queues are outbound only.

If you want to act upon both egress and ingress traffic then you may want to place a router host with NetEm between the two test machines, and run NetEm on both interfaces (maybe with differing parameters on each one). The easiest way to achieve this is to run NetEm in a VM to route between two VM networks.

## What

Check out the `help` section to see what can be done with this tool:

```
$ ./slow --help
slow - Simulate a low-bandwidth, high-latency network connection.

This script uses the "netem" ([1], [2]) feature available in the Linux kernel,
via the "tc" ([3]) command and the "HTB" ([4]) qdisc.

NetEm is a highly configurable traffic control discipline module which
deliberately delays, drops, and reorders packets.

This only works for egress (i.e. sending packets) because the queues are
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

slow -i <Interface> { <Preset> | [-r <Rate>] [-d <Delay>] [-l <Loss>] }
slow -i <Interface> reset
slow -i <Interface> status

Arguments:

<Interface>
    Name of the network interface to be configured.
    E.g.: "eth0", "enp0s25", "wlan0", etc.
    Optional. Default: `"eth0"`.

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

<Rate>
    Maximum bandwidth that the connection must be restricted to,
    in kbps (kilobits per second).
    Optional. Default: `500`.

<Delay>
    Latency forced into the packet transmission, in milliseconds.
    Optional. Default: `0`.

<Loss>
    Average emulated packet drop rate, as a loss percentage.
    Optional. Default: `0.0`.
```


## More reading

* The Linux Foundation - [netem](https://wiki.linuxfoundation.org/networking/netem) | [Archive](https://web.archive.org/web/20191023125240/https://wiki.linuxfoundation.org/networking/netem)
* Use Linux Traffic Control as impairment node in a test environment
    - [Part 1](https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-1) | [Archive](https://web.archive.org/web/20191023121720/https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-1/)
    - [Part 2](https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-2) | [Archive](https://web.archive.org/web/20191023121821/https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-2)
    - [Part 3](https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-3) | [Archive](https://web.archive.org/web/20191023121905/https://www.excentis.com/blog/use-linux-traffic-control-impairment-node-test-environment-part-3)
* Ubuntu Manpages:
    - [tc - Traffic Control](https://manpages.ubuntu.com/manpages/bionic/en/man8/tc.8.html)
    - [NetEm - Network Emulator](https://manpages.ubuntu.com/manpages/bionic/en/man8/tc-netem.8.html)
    - [HTB - Hierarchy Token Bucket](https://manpages.ubuntu.com/manpages/bionic/en/man8/tc-htb.8.html)
* Simulating low bandwidths: how to make sure your apps work in the field | [Archive](https://web.archive.org/web/20121104183730/http://blog.aptivate.org/2010/01/23/make-sure-your-apps-work-in-the-field/)



## Credits

This tool was conceptualized after reading several sources on the topic of
network shaping tools available in Linux, and the actual implementation was
inspired by the work of Richard Bullington-McGuire, as found here:
https://gist.github.com/obscurerichard/3740206
