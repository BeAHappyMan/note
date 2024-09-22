# 摘要

无线中继是指AP在网络连接中起到中继的作用，实现信号的放大，从而延伸无线网络的覆盖范。这对于一些覆盖面积广或者门墙障碍物较多场所具有重要意义。然而由于中继路由器发送和接受帧在同一个信道上，这使得无线吞吐量至少减少了一半。基于此，我们提出了1，利用基于链路快慢因子的调度策略，基于链路吞吐量变化动态调整分组的策略，以及跨层TCP代理的策略，上实现了双频中继融合传输提升了链路汇聚的吞吐量，减少无线链路不稳定，乱序导致的TCP业务发送速率降低等不利影响。在真实实验结果展示1实现了更好的性能相比于其他的方案。

# intro

对于一些覆盖面积广或者门墙障碍物较多的需要网络覆盖的场所，单个路由器的信号难以覆盖到各个角落，多个路由器中继是一种能够Wi-Fi覆盖率的有效方式。A wireless repeater  refers a device that takes an existing signal from a wireless routeror wireless access point and rebroadcasts it to create a second network。相比于有线中继，无线中继不需要网线，更加灵活，因而渐渐成为了用户首选。

然而，在中继的过程中，会发生带宽失真的现象。Since only one wireless device can transmit at once, wireless transmissions are doubled (router to the repeater and then repeater to the client versus just router to the client), and so Wireless throughput is reduced by at least 50%. Furthermore, the quality of the connection to an extender is generally worse than if connected to the host access point.

现在越来越多的路由器设备支持双频发送数据，也就是2.4GHz和5Ghz。相比之下，2.4GHz穿墙能力更强，5Ghz抗干扰能力更强。用户可以根据自己的需求选择合适的频段进行中继。但双频路由器并没有提升中继带宽。如果能将两个频段的Wi-Fi融合中继，相比较于单频中继，路由器级联网络将更加稳定和高效。然而由于无线链路存在链路质量差异，需要制订一套合理的调度传输策略，才能充分利用双链路的传输带宽，避免质量差的链路拖累质量好的链路。

受到这样的启发，我们提出了1，通过融合两条链路中继进而提高带宽。核心思想是，在中继路由器侧，对每N个数据包进行一次调度，根据拥塞队列长度等指标选择合适的链路发送数据包。通过选取一个传输耗时较少的链路，我们可以进一步提高双频中继的带宽。

网关路由器和中继路由器通过双频中继，终端设备通过中继路由器的5GHz接入无线局域网(中继路由器的 5 GHz
处理单元是网关路由器和终端设备共享 )。



具体而言，本文的工作包括以下四点：

为了准备评估链路质量，我们提出了链路快慢因子用以表示数据包到达对端的快慢程度，并利用卡尔曼滤波器对相关指标进行预测。

为了选择合适的分组数据包大小，我们根据当前吞吐量，使用启发式的算法动态调整分组数据包大小。

为了减少乱序等原因对tcp发送速率的影响，我们对数据包标号整序，并在中继路由器侧使用tcp代理技术发送数据包。

我们在真实环境中实现了1 ，并将其与其他方法作对比，实验结果证明我们的设计可以提升聚合链路容量 。



# 相关工作

1充分利用双频设备的两条链路，从而增加中继带宽，提升用户体验。

多个无线网卡接入 WLAN 网络形成不同的无线链路连接。而由于每条无线链路的路径质量状况不同，且业务的传输要求不同，如何在聚合链路的多条无线链路中进行数据包的调度实现负载均衡，以满足业务的传输要求，成为关键问题之一。  

现有的多路径传输方案基本上都使用轮询（round-robin）算法调度数据包，当发送端传输数据时，依次在每条可用路径上分配数据包进行传输。轮询算法是最简单、最直接的数据包调度算法[34]，但是它并没有将多条可用路径的带宽、往返时延和丢包率等方面的不同考虑在内，这样就使得乱序到达的数据包数目大大增加，因此接收端需要占用大量的内存缓存乱序到达的分组 。

此外，通过链路质量估计可以选择合适的链路发送数据包。pTCP 和 cTCP 依照路径带宽比率来进行分组调度，pTCP 用往返时延和拥塞窗口的比估计每条路径的瞬时带宽；cTCP 也同时考虑丢包率，通过建立模型来估计每条路径的平均带宽。根据估计的结果，给带宽越高的路径分配越多的数据分组，这样该路径所需要传输的流量就越多。在 mTCP 中，根据调度算法估计每条路径中已发送但未确认的数据的大小与拥塞窗口的大小之比，将待发送的数据分组交给比值最小的路径传输。为了解决多条路径传输导致的公平性问题，文献提出一种自适应多路径拥塞控制机制，可快速检测这些问题的发生并将其多径流转换为单路径流，显著提高了多径和单径流间的网络公平性。  

基于往返时延对网络可用容量估计进行数据包的调度，可以在一定程度上实现负载的均衡，但是由于网络的质量状况不仅与往返时延有关，还与其他参数有关，故这种方法对可用容量的估计并不准确。而本文通过对中继建模，综合考虑时延，丢包率，  拥塞队列长度等指标，能够更好地评估链路质量。

```
@inproceedings{hsieh2002ptcp,
  title={pTCP: An end-to-end transport layer protocol for striped connections},
  author={Hsieh, Hung-Yun and Sivakumar, Raghupathy},
  booktitle={10th IEEE International Conference on Network Protocols, 2002. Proceedings.},
  pages={24--33},
  year={2002},
  organization={IEEE}
}
@inproceedings{dong2007multi,
  title={Multi-path load balancing in transport layer},
  author={Dong, Yu and Wang, Dingding and Pissinou, Niki and Wang, Jian},
  booktitle={2007 Next Generation Internet Networks},
  pages={135--142},
  year={2007},
  organization={IEEE}
}
@inproceedings{zhang2004transport,
  title={A Transport Layer Approach for Improving End-to-End Performance and Robustness Using Redundant Paths.},
  author={Zhang, Ming and Lai, Junwen and Krishnamurthy, Arvind and Peterson, Larry L and Wang, Randolph Y},
  booktitle={USENIX Annual Technical Conference, General Track},
  pages={99--112},
  year={2004}
}
@article{kheirkhah2017amp,
  title={AMP: A better multipath TCP for data center networks},
  author={Kheirkhah, Morteza and Lee, Myungjin},
  journal={arXiv preprint arXiv:1707.00322},
  year={2017}
}



@inproceedings{Zhao2010,
  author    = {Zhao, H. and Garcia-Palacios, E. and Xi, Y. and others},
  title     = {Estimating resources in wireless ad-hoc networks: A Kalman filter approach},
  booktitle = {International Symposium on Communication Systems Networks and Digital Signal Processing},
  pages     = {71--75},
  year      = {2010},
  publisher = {IEEE}
}

@inproceedings{MasudRana2017,
  author    = {Masud Rana, M. and Li, L.},
  title     = {Kalman Filter Based Microgrid State Estimation Using the Internet of Things Communication Network},
  booktitle = {International Conference on Advanced Communication Technology},
  pages     = {501--505},
  year      = {2017},
  publisher = {IEEE}
}

@inproceedings{Zhang2013,
  author    = {Zhang, X. and Nguyen, T. M. T. and Pujolle, G.},
  title     = {Kalman filter based bandwidth estimation and predictive flow distribution for concurrent multipath transfer in wireless networks},
  booktitle = {IEEE International Conference on Network Infrastructure and Digital Content},
  pages     = {305--309},
  year      = {2013},
  publisher = {IEEE}
}

@inproceedings{Senel2007,
  author    = {Senel, M. and Chintalapudi, K. and Lal, D. and others},
  title     = {A Kalman Filter Based Link Quality Estimation Scheme for Wireless Sensor Networks},
  booktitle = {Global Telecommunications Conference, 2007. GLOBECOM '07. IEEE},
  pages     = {875--880},
  year      = {2007},
  publisher = {IEEE}
}

@article{Qin2013,
  author    = {Qin, F. and Dai, X. and Mitchell, J. E.},
  title     = {Effective-SNR estimation for wireless sensor network using Kalman filter},
  journal   = {Ad Hoc Networks},
  volume    = {11},
  number    = {3},
  pages     = {944--958},
  year      = {2013}
}

@article{Jayasri2014,
  author    = {Jayasri, T. and Hemalatha, M.},
  title     = {Link Quality Estimation using Soft Computing Technique},
  journal   = {Middle East Journal of Scientific Research},
  year      = {2014}
}

```



# overview

中继需求

单频中继存在带宽减半的问题，严重影响用户体验。双频wifi为解决该问题提供了思路。现在虽然 网关路由器 和中继路由器 都支持并发双频，已经具备了使用 2.4 GHz和 5 GHz两个频段进行连接的条件，但是由于无线链路的不稳定性，若要充分发挥双频传输的特性，需要制定合理的链路感知及调度策略，才能保障双频传输能够实现吞吐量聚合的效果，本文的主要内容将围绕能提升聚合链路吞吐量的调度策略展开。为了准确衡量调度策略的功能，这里引入绑定效率 ， 绑定效率就是在输入稳定且流量足够大的情况下汇聚链路实际吞吐量 TP与各链路实际容量 ci之和的比值，即对两条链路的利用率。本文双频融合传输研究的优化目标是尽量使绑定效率 η 达到更大。  

链路质量感知与预估

网关路由器和中继路由器之间双频汇聚的场景可抽象成如图所示模型，发送方和接收方有两条链路link1、link2，其中c1 和c2 分别代表link1 和link2 的信道容量，h1 和h2 代表可观测的信道质量参数矩阵。无线环境经常受波动、多径衰落、移动性和竞争干扰的影响，信道容量一直处于变化之中，无法提前预知，因此信道容量c 需要通过信道质量参数h 来进行表征。一方面如何选择参数进行表征是需要考虑的问题，另一方面，链路质量会存在波动导致质量参数的观测存在误差，需要对参数观测值采取一定手段进行修正。

本文综合考虑各链路拥塞长度和链路服务时间，提出了链路快慢因子，在此基础上利用卡尔曼滤波器修正链路服务时间，更好地评估链路质量。



在对数据包的决策过程中，每来一组数据包，就进行一次链路决策，只对首包进行决策，分组的其他数据包的传输链路和首包保持一致。在这过程中，分组的大小是需要考虑的一个问题，分组个数太小，同一个业务流的数据链路切换会较为频繁，不能充分利用 802.11 底层的聚合机制，可能导致乱序较多吞吐量难以提升，另外，分组个数太小也就意味着调度算法执行次数越多，将会增加路由器 CPU 的负担。但是，分组数目也不能太大，数目过大将无法及时对链路质量变化做出调度调整，会导致难以做到负载均衡。

本文实时监控聚合链路可用容量变化动态调整调度分组的大小 ，当链路容量降低较多时，减小下一周期的调度分组数目，当链路容量提升较多时，增大下一周期的调度分组数目，从而提升总吞吐性能。  

  

双频发送以及无线环境的不稳定因素，数据包到达中继路由器时难以避免存在乱序现象，乱序的出现对于 TCP 业务而言会引起降速，对于 UDP 业务而言可能会导致业务出现卡顿 。同时会引入队头阻塞的问题，导致整体业务时延的增加，进而会导致吞吐量下降。

本文在网关路由器侧对数据包进行一次封装，以便携带序号信息在中继路由器用于保序。此外，利用tcp代理机制，减少乱序导致的tcp业务降速问题 。  

scheduling strategy based on link speed factor, dynamically adjusts packet grouping based on variations in link throughput, and cross-layer TCP proxy techniques

当数据达到稳定的情况下，Wi-Fi双频融合传输时两条链路的数据包排队模型

为了方面描述，我们把2.4Ghz和5GHz两条链路记作linkA和link B

# design

数据传输越快，链路吞吐量越大，给数据包选择链路时，为提升聚合链路吞吐量，应该选择传输时间更短的链路。  在数据到达稳定的情况下， Wi-Fi 双频融合传输时双链路可表示如图 所示。 能获取链路拥塞长度信息的情况下，当数据包 Q（长度为 S）到达时，假设链路 A 上数据包排队长度为 LA，链路 B 上数据包排队长度为 LB，链路 A 的可用带宽为 cA， 链路 B 的可用带宽为 cB， 那么选择链路 A 或链路 B 的预计传输时间 ωi 可表示公式1， 要让数据包发的快， 吞吐量更高， 那么选择一个 ωi 更小的链路即可。  

然而，用吞吐量表征链路质量能及时感知链路容量恶化，却无法及时感知链路容量变好的情况。  如果将单个数据包的传输时间分成排队时间和非排队时间，在传输时，数据包非排队时间是不受以往数据调度比例影响的，可以将其用来表示链路可用  带宽的变化。本文假设数据包的长度固定为 S，那么可用带宽 ci 可表示为公式2，其中 τi 表示链路 i 上传输单个数据包所需时间（不包含排队时间）， 因此可以将公式3换成如公式4所示。 



卡尔曼

根据 上文对$\tau_i$ 的定义，受环境干扰的影响，其会存在较大波动，同时对于其测量也会存在一定的误差，因此需要对$\tau_i$ 采取一定的预测估计手段，卡尔曼滤波器广泛用于网络状况的估计 。在开放环境下（有噪声），对于每条链路，卡尔曼预测过程如公式2所示：   



分组的数目会影响调度性能，图展示了我们的预实验的结果。在干扰不变的情况存在一个最佳分组数目使汇聚链路实际吞吐量最佳，并且链路可用容量越大，最佳分组数目的值越大；链路可用容量越小，最佳分组数目的值越小。



具体而言，我们将根据聚合链路可用容量变化动态调整调度分组的大小，调整过程分为两部分，分别为初始化阶段和动态调整阶段 。初始化阶段是寻找最佳分组使得吞吐量 ，分组数目 S 从 5 开始调整，以[5,60]为区间，以 5 为步进单位，每个分组数目持续一秒钟，记录各个分组下的吞吐量。 最终获得最小吞吐量TPmin，最大吞吐量 TPmax,以及该吞吐量周期对应的分组数目即最佳分组数目 S*  并将 TPmax 记为 TP* 。

我们据融合链路上一周期总吞吐量的表现  来调整下一周期分组的设定，以适应链路变化，提升后续周期的传输性能。由于链路存在一定波动，不能吞吐量一变化就调整分组数目 S，需要对吞吐量变化设置一个阈值，增大或者减小超过一定阈值，再去调整分组数目 S。这里根据经验，阈值设为初始化阶段记录的 TPmax 和 TPmin 的差值 ΔTP， 每次调整的步幅 ΔS 设为 5。 动态调整阶段的过程可表示如算法 2 所示。



tcp代理

乱序达到的数据包以及链路切换引起的RTT变化会导致tcp发送速率降低，本文在对数据包整序的基础上利用tcp代理技术减少这一现象对吞吐率的影响。具体而言，当服务器和终端设备建立起 TCP 连接后，网关路由器需要扮演两个角色 

1.代理终端：当服务器发送的 TCP 数据报文到达网关路由器时，网关路由器将其传送给终端设备之前缓存一份数据，同时代理终端向服务器提前发送 TCP 确认报文（ACK），以让服务器继续发送数据报文。当服务器和网关路由器之间发生丢包时，也需要由网关路由器发送冗余 ACK  

2.代理服务器：当接收方的确认报文到达网关路由器时，网关路由器将其过滤（ACK 过滤），不传递给服务器，网关路由器根据 ACK 的值删除相应的缓存；如果  终端设备发出三次及以上冗余 ACK 或者长时间未发 ACK，则由网关路由器代理服务器向终端设备重传数据报文，以避免 TCP 连接中断。  

具体代理过程如图所示。



为了防止 TCP 发送速率提升过大超出无线网络的实际带宽，导致网关路由器缓存数据过多、设备内存溢出，同时也为了防止服务器一次性发送的数据超出接收方的接收窗口，因此需要在代理过程中做相应的流量控制。  

对于每个 TCP 连接，代理过程分成两个阶段——加速代理和平滑代理，初始化时处于加速代理阶段，当网关路由器缓存的数据量(S)超过接收方的接收窗口(Winmax)值时，转换为平滑代理阶段，当网关路由器缓存的数据量减小到低于接收方最大接收窗口值时 70%时，重新恢复到加速代理阶段。状态转换关系如图所示。

在加速代理阶段，网关路由器收到服务器发来的数据后，缓存一份后立即发送给终端设备，并根据 RTT 欺骗策略向服务器提前发送确认报文。在平滑代理阶段，网关路由器收到服务器的发来的数据后，只做缓存（不计入 S）而不直接发给终端设备，也不提前向服务器发送确认，只有当终端设备的确认报文到达网关路由器后，网关路由器再将缓存未发的数据发送给终端设备 。



# 实现





本文双频传输的总体架构如图所示，汇聚传输主要是在无线路由器设备 Wi-Fi 驱动上实现的，针对从网关路由器发送至中继路由器的下行数据流，实现了前文阐述的方案。网络路由器和中继路由器的开发均基于由某知名通讯公司的路由器内核。

网路路由器主要被分为数据报文截获模块 ，链路决策模块  分组自适应模块  链路快慢因子更新模块  ，数据封装模块，tcp代理模块。具体如图所示。

首先是数据报文截获模块，由于本文主要针对 TCP 和 UDP 业务做并行传输，其主要内容是识别承载 TCP 和 UDP 业务的 Ethenet Ⅱ报文， 让报文能够进入本文的汇聚调度流程。其次是链路决策模块，其作用是针对每个分组的首包进行链路选择，分组的其他数据包则遵从首包的传输链路，分组的设置取决于分组自适应模块，在为数据包进行链路选择时，选择链路快慢因子最小的一个链路传输。然后是业务流区分与封装模块，当为链路选择好传输链路后，接下来就是要为其封装 LLC 报头，以附带业务流序号、链路序号以及统一序号，方便在接收方还原顺序。接下来是分组自适应模块，其主要内容是周期性获取各条链路的吞吐量，获得吞吐量后再动态调整分组大小。链路快慢因子更新模块，其主要内容是对组成链路快慢因子的三种参数进行测量，即链路服务时间、链路拥塞数据包个数、丢包率的测量，对于服务时间，还需要使用卡尔曼滤波进行修正。



中继路由器主要被分为报文截获模块，流量检测与反馈，丢包检测与反馈模块，数据整序和解封装模块，以及tcp代理模块。具体如图所示

报文截获模块的主要内容是截获自定义封装数据帧，以便让其进入整序序流程；流量（吞吐量）检测与反馈和丢包检测与反馈模块主要内容是对截获的报文进行流量统计和丢包统计，并周期性反馈给发送方，以配合发送方的调度；数据整序和解封装模块主要对数据包进行整序操作，保证数据有序转发并减少队头阻塞以降低业务时延并对整序后的自定义封装数据帧进行数据还原操作，还原成原始 IP 报文，以便 TCP/IP 协议栈能够识别。tcp代理模块主要 负责对数据缓存，代理确认，流量控制等功能。  

# 实验

双频汇聚综合调度算法的测试环境位于合作公司的办公区，办公区环境有其他 Wi-Fi 设备，比较贴近有干扰的实际环境。总体测试过程分为三个部分，首先是对基于链路快慢因子调度算法模型的测试，测试时，主要分析在不进行卡尔曼时延滤波和进行卡尔曼时延滤波的场景下，分别比较其与轮询调度算法、基于时延吞吐量变化调度算法的性能差异；其次是测试分组自适应调整策略带来的绑定效率收益，测试时叠加在基于链路快慢因子调度算法之上，分析加分组调整与不加调整策略的性能差异 ；最后是测试tcp代理策略，验证其与不使用该策略是否有性能提升。

实验过程中，网关路由器和中继路由器之间建立 2.4 GHz 和 5 GHz 双频连接， 2.4 GHz 使用 20 MHz 频宽， 5GHz 使用 40 MHz 频宽，台式主机通过以太网线连接网关路由器的 LAN 口，笔记本电脑通过无线连接上中继路由器的 5 GHz WiFi，笔记本电脑和台式机处于同一网段，测试时由于办公区环境限制网关路由器和中继路由器距离大概 0.7 m，中继路由器与笔记本电脑距离大概 1 m。  

Wi-Fi 双频中继场景下，由于中继路由器的 5 GHz 处理模块是复用的，即中继路由器的 5 GHz 处理模块既要与网关路由器通信，也要与终端设备通信，因此根据绑定率概念，实际情况下，聚合链路吞吐量绑定率计算公式为 公式2所示， 其中 D、 E、 A 是图 1所示场景的吞吐量 。



表1展示了分别使用2.4GHz和5GHz实现单频中继的场景，当使用5GHz中继时，平均吞吐率为107.18Mbps，当使用2.4GHz中继时 ，平均吞吐率为105.89Mbps。

表2展示了双频中继场景下，采用轮询调度策略和基于时延吞吐量变化调度策略的实验数据。其中，轮询调度下调度发送时 5 GHz 和 2.4 GHz 的比例为 1:1。结合公式可以计算得到，轮询调度平均吞吐量为111.36Mbps，平均绑定效率为9.5%，基于时延吞吐量变化调度平均吞吐量为118.87Mbps，平均绑定率为74.2%。  



在本实验中，我们针对链路快慢因子调度算法模型进行测试，实验场景与基线版本一致。针对是否进行卡尔曼滤波修正进行了两组实验，如图所示



我们可以计算出基于链路快慢因子调度为平均绑定效率为 79% ，明显由于轮询调度和基于时延的调度。当链路服务时间进行卡尔曼修正后，基于链路快慢因子调度方案的平均绑定效率为 82.2%，相比较于不做服务时延修正，吞吐量绑定效率更高。 这证明了基于链路快慢因子调度和卡尔曼滤波修正均发挥了一定的作用。



图1 展示了绑定效率的实验结果。当我们加入了分组自适应算法之后，5组实验的平均吞吐量是136.0Mbps，平均绑定效率为84.9%，相比较于不加入分组自适应算法，吞吐量绑定效率更高。



图1 展示了绑定效率的实验结果。当我们加入了TCP代理技术之后，5组实验的平均吞吐量是145.4Mbps，平均绑定效率为90.8%，相比较于不加入TCP代理技术，吞吐量绑定效率更高。

本文中，我们研究了wifi网络中的无线中继问题，为了提高中继带宽提高用户体验。为了提高汇聚链路的中继带宽，我们提出了链路快慢因子用以评估链路质量，并根据吞吐量动态调整分组大小，最后采用TCP代理技术减少乱序等因素对TCP发送速率的影响。实验结果表明，1能够提高中继带宽，改善用户体验。

```
@inproceedings{kakisim2023xai,
  title={XAI empowered dual band Wi-Fi based indoor localization via ensemble learning},
  author={Kakisim, Arzu Gorgulu and Turgut, Zeynep and Atmaca, Tulin},
  booktitle={2023 14th International Conference on Network of the Future (NoF)},
  pages={150--158},
  year={2023},
  organization={IEEE}
}
@inproceedings{hashemi2001concurrent,
  title={Concurrent dual-band CMOS low noise amplifiers and receiver architectures},
  author={Hashemi, Hossein and Hajimiri, Ali},
  booktitle={2001 Symposium on VLSI Circuits. Digest of Technical Papers (IEEE Cat. No. 01CH37185)},
  pages={247--250},
  year={2001},
  organization={IEEE}
}
@article{zhang2018data,
  title={A data-driven approach to client-transparent access selection of Dual-Band WiFi},
  author={Zhang, Jun and Zhang, Guangxing and Wu, Qinghua and Liao, Binbin and Xie, Gaogang},
  journal={IEEE Transactions on Network and Service Management},
  volume={16},
  number={1},
  pages={321--333},
  year={2018},
  publisher={IEEE}
}
@inproceedings{abusubaih2013ieee,
  title={IEEE 802.11 n dual band access points for boosting the performance of heterogeneous WiFi networks},
  author={Abusubaih, Murad A and Najem Eddin, Saif and Khamayseh, Ahmad},
  booktitle={Proceedings of the 8th ACM workshop on Performance monitoring and measurement of heterogeneous wireless and wired networks},
  pages={1--4},
  year={2013}
}
```

