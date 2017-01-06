layout: IOT
title: IOT及6Lowpan中iperf移植
date: 2016-10-16 16:53:47
categories:
- linux
tags: 
- IOT
description: "物联网的一些介绍及6Lowpan中iperf移植"
---

***
# 关于IOT目前的发展
前记：之前在Cisco实习的部门是搞了一会物联网（IOT），个人感觉IOT的道路还有一段距离啊，目前的IOT主要应该分为两个方向,(1)智能家居，主要以wifi,蓝牙协议为载体，部分结合了Zigbee，这类的网络一般范围不大，国内的小米，360都在做;(2)工业上面的发展研究，如智能电网等，这类的网络一般范围较大，节点较多，一般以Ieee802.15.4为基础。但是，这两类方向都没有一个统一的工业级的标准，现在国内的华为liteos，思科的cg-mesh，google的 openthread（由14年google收购nest发展而来）。他们基本都有一个共同点，都是以[6LowPan](https://en.wikipedia.org/wiki/6LoWPAN)这个架构为基础发展而来。

## IOT基本框架

如下图是IOT的基本框架结构。首先一堆IOT节点通过无线相互连接，最终的数据会从sersnor边界路由器发送到Internet中，最后汇聚到云端。远端通过Cloud边界路由器将数据包转发到不同功能的Server服务器，可能到web服务器，or数据库，或者一些其他的管理Server等。
![Alt text](/img/1.png)

## 6LowPan介绍
IOT面临的一个很大的问题就是传感器节点太多，云端通过怎样的方式与节点通信，由于ipv6的地址是128位，很明显的可以为这些节点提供管理方式。目前6LowPan模型已被很多厂商接受，Cisco，google也在这基础上提出了自己的协议栈。如google开源的[openthread](https://github.com/openthread/openthread),Cisco也提出了cg-mesh方案，但是并没有开源。
6LowPan架构如上图所示，与传统的tcp/ip五层模型不同。首先，目前tcp/ip的数据链路层及物理层采用的ieee802.11(wifi),IEEE802.2等，而6LowPan采用的是与Zigbee一样的IEEE802.15.4，一种非常低功耗的协议。在ip层，6LowPan是基于rpl的路由协议，而不是tcp/ip里给予OSPF(Dijkstra)的最短路径算法。6LowPan同过rpl路由协议，将节点组成tree形的网络，而非mesh网络。
目前一些为6Lowpan提出的OS有[tinyos](http://www.tinyos.net/),[liteos](http://www.huawei.com/minisite/iot/cn/liteos.html),openthread等。

***
# Iperf移植
[Iperf](https://iperf.fr/)是一个网络性能测试工具，可以测试最大TCP和UDP带宽性能。原来的iperf只支持linux和windows平台。用来测试两台pc之间的带宽。但是在上述的IOT框架中，云端的server至IOT的节点之间是linux<---->tinyos之间的链路。为此必须将iperf移植到tinyos上面，才可以测试两者之间的带宽。 
> 这里想特别说明一下，虽然我还是本硕阶段都是跟通信网络相关的，但还是要鄙视一下自己，之前一直知道带宽的概念原理，但是一直没能和实际工程上面将结合的很好。我们在用unix的posix socket api时知道，send/write都是支持同步阻塞or非阻塞的方式，其实如果我们在应用层写业务不注意的话，有一种情况会发生,就是前后两次调用send()时，虽然返回值返回表示发出的字节数，但是，实际上，这不是对方实际接收到的数据字节数。所以，这个时候提出带宽的概念。带宽的单位是$bit/s$,对其取倒数，即$s/bit$，表示发送一个bit or byte需要的时间s。其实这就表示在应用层，我们调用两次send的前后时间间隔，不过在如今的网速下，一般情况在业务层其实不需要太关注，但是当网速比较低时，这其实影响就大了，我们只要在业务层适当的控制调用send间隔，可以很好的控制丢包率。。。

未完,待续...



