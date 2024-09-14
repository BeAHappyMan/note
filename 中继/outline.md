

对于一些覆盖面积广或者门墙障碍物较多的需要网络覆盖的场所，单个路由器的信号难以覆盖到各个角落，多个路由器中继是一种能够Wi-Fi覆盖率的有效方式。A wireless repeater  refers a device that takes an existing signal from a wireless routeror wireless access point and rebroadcasts it to create a second network。相比于有线中继，无线中继不需要网线，更加灵活，因而渐渐成为了用户首选。

然而，在中继的过程中，会发生带宽失真的现象。Since only one wireless device can transmit at once, wireless transmissions are doubled (router to the repeater and then repeater to the client versus just router to the client), and so Wireless throughput is reduced by at least 50%. Furthermore, the quality of the connection to an extender is generally worse than if connected to the host access point.

现在越来越多的路由器设备支持双频发送数据，也就是2.4GHz和5Ghz。相比之下，2.4GHz穿墙能力更强，5Ghz抗干扰能力更强。用户可以根据自己的需求选择合适的频段进行中继。但双频路由器并没有提升中继带宽。如果能将两个频段的Wi-Fi融合中继，相比较于单频中继，路由器级联网络将更加稳定和高效。然而由于无线链路存在链路质量差异，需要制订一套合理的调度传输策略，才能充分利用双链路的传输带宽，避免质量差的链路拖累质量好的链路。

受到这样的启发，我们提出了，通过融合两条链路中继进而提高带宽。核心思想是，在ont侧，对每N个数据包进行一次调度，根据等待时间选择合适的链路发送数据包。

具体而言，本文的工作包括以下四点：

为了准备评估链路质量，我们提出了链路快慢因子用以计算数据包发送时间，并利用 卡尔曼滤波器对带宽进行预测。

为了选择合适的分组数据包大小，我们根据当前吞吐量，使用启发式的算法动态调整分组数据包大小。

为了减少双频发送导致的乱序，我们对数据包标号整序，并使用tcp代理技术减少乱序导致的发送降速。

我们在正式环境中实现了 并将其与其他方法作对比，验证 可以提高中继带宽。



相关工作



overview

中继需求

解决方案



