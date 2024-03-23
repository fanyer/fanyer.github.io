---
layout: post
title:  "wireshark filter手记"
date:   2015-09-03 16:23:52 +0800
categories: jekyll update
---

[ref1](http://biot.com/capstats/bpf.html)
[ref2](https://ask.wireshark.org/questions/6660/what-is-the-difference-between-capture-filter-and-display-filter)
[display filter](https://wiki.wireshark.org/DisplayFilters)
[explanation](https://wiki.wireshark.org/CaptureFilters#Capture_filter_is_not_a_display_filter)
>A capture filter is used to select which packets should be saved to disk while capturing. For capture filters wireshark uses the BPF syntax. BPF is module that runs in the kernel and can therefor maintain high rates of capturing because the packets do not have to move from kernel space to user space when filtering. The things that can be filtered on are predefined and limited (compared to display filters) as full dissection has not been done on the packets.

>Display filters are used to change the view of a capture file. They take advantage of the full dissection of all packets. This makes it possible to do very complex and advanced filtering when analyzing a network tracefile.

言之，`capture filter`必须是`bfs`（`berkerly packet filter` 语法）


### 1.capture filter

`bpf`主要形式是`qualifier`+`id`(name or number)

`qualifier`有3种：
* `type`, 可能的类型有: `host`, `net` , `port` and `portrange`,etc
* `dir`, 可能的directon有: `src`, `dst`, `src or dst` and `src and dst`, etc
* `protocol`, 可能的协议有: `ether`, `fddi`, `tr`, `wlan`, `ip`, `ip6`, `arp`, `rarp`, `decnet`, `tcp` and `udp` etc



### 2.display filter
[ref](https://www.wireshark.org/docs/dfref/)

该filter 的 [ColoringRules](https://wiki.wireshark.org/ColoringRules)

#### Examples

Show only SMTP (port 25) and ICMP traffic:
```
 tcp.port eq 25 or icmp
```

Show only traffic in the LAN (192.168.x.x), between workstations and servers -- no Internet:
```
ip.src==192.168.0.0/16 and ip.dst==192.168.0.0/16
```

TCP buffer full -- Source is instructing Destination to stop sending data
```
 tcp.window_size == 0 && tcp.flags.reset != 1
```

Filter on Windows -- Filter out noise, while watching Windows Client - DC exchanges
```
 smb || nbns || dcerpc || nbss || dns
```

Sasser worm: --What sasser really did--
```
  ls_ads.opnum==0x09
```

Match packets containing the (arbitrary) 3-byte sequence 0x81, 0x60, 0x03 at the beginning of the UDP payload, skipping the 8-byte UDP header. Note that the values for the byte sequence implicitly are in hexadecimal only. (Useful for matching homegrown packet protocols.)
```
  udp[8:3]==81:60:03
```

The "slice" feature is also useful to filter on the vendor identifier part (OUI) of the MAC address, see the Ethernet page for details. Thus you may restrict the display to only packets from a specific device manufacturer. E.g. for DELL machines only:
```
  eth.addr[0:3]==00:06:5B
```

It is also possible to search for characters appearing anywhere in a field or protocol by using the `contains` operator.

Match packets that contains the 3-byte sequence 0x81, 0x60, 0x03 anywhere in the UDP header or payload:
```
  udp contains 81:60:03
```

Match packets where SIP To-header contains the string "a1762" anywhere in the header:
```
  sip.To contains "a1762"
```

The matches, or ~, operator makes it possible to search for text in string fields and byte sequences using a regular expression, using Perl regular expression syntax. Note: Wireshark needs to be built with libpcre in order to be able to use the matches operator.

Match HTTP requests where the last characters in the uri are the characters "gl=se":
```
  http.request.uri matches "gl=se$"
```

Note: The $ character is a `PCRE` punctuation character that matches the end of a string, in this case the end of `http.request.uri` field.

Filter by a protocol ( e.g. SIP ) and filter out unwanted IPs:
```
  ip.src != xxx.xxx.xxx.xxx && ip.dst != xxx.xxx.xxx.xxx && sip
```

#### 陷阱

Some filter fields match against multiple protocol fields. For example, "ip.addr" matches against both the IP source and destination addresses in the IP header. The same is true for "tcp.port", "udp.port", "eth.addr", and others. It's important to note that
```
 ip.addr == 10.43.54.65
```
is equivalent to
```
 ip.src == 10.43.54.65 or ip.dst == 10.43.54.65
 ```

This can be counterintuitive in some cases. Suppose we want to filter out any traffic to or from 10.43.54.65. We might try the following:
```
 ip.addr != 10.43.54.65
 ```
which is equivalent to
```
 ip.src != 10.43.54.65 or ip.dst != 10.43.54.65
 ```
This translates to "pass all traffic except for traffic with a source IPv4 address of 10.43.54.65 and a destination IPv4 address of 10.43.54.65", which isn't what we wanted.

Instead we need to negate the expression, like so:
```
 ! ( ip.addr == 10.43.54.65 )
```
which is equivalent to
```
 ! (ip.src == 10.43.54.65 or ip.dst == 10.43.54.65)
```

This translates to "pass any traffic except with a source IPv4 address of 10.43.54.65 or a destination IPv4 address of 10.43.54.65", which is what we wanted.

This can also happen if, for example, you have tunneled protocols, so that you might have two separate IPv4 or IPv6 layers and two separate IPv4 or IPv6 headers, or if you have multiple instances of a field for other reasons, such as multiple IPv6 "next header" fields.

If you have a filter expression of the form name op value, where name is the name of a field, op is a comparison operator such as == or != or < or..., and value is a value against which you're comparing, it should be thought of as meaning "match a packet if there is at least one instance of the field named name whose value is (equal to, not equal to, less than, ...) value". The negation of that is "match a packet if there are no instances of the field named name whose value is (equal to, not equal to, less than, ...) value"; simply negating op, e.g. replacing == with != or < with >=, give you another "if there is at least one" check, which is not the negation of the original check.
