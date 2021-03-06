转换时间： 2021-02-01

# 开源开发工具大会OSDT19议程确定，11月9日下午见！（文末报名）
Posted on 2019/10/29 by ww	

开源开发工具大会（OSDT，原 HelloGCC Workshop）是开源软件开发者的交流会，今年已经是第11届。我们在这里分享自己在开源软件方面的开发工作，研究成果，经验学习。话题主要面向开源开发工具。欢迎各位新老朋友通过点击原文报名或扫描文末二维码参加。

时间：2019年11月9日周六下午1点至6点

地点：北京市海淀区中关村南四街四号软件研究所5号楼4层大报告厅 （地铁知春路站西北方向300米）

13:00 – 13:30 签到 & Social

13:30 – 14:00 郭任 Csky Intro – What’s the meaning of a new arch for linux

14:00 – 14:30 邱吉 NVDLA开源编译器的功能分析和对比

14:30 – 15:00 朱凌宇 托管堆直写技术初探–降低JVM的序列化开销

15:00 – 15:30 康烁 开源项目L2C：经过形式化验证的可信编译器

15:30 – 16:00 茶歇

16:00 – 16:30 Gopher Web Assembly & Go Implementation

16:30 – 17:00 李枫 在内核虚拟机与在内核服务

17:00 – 17:30 吴伟 共促方舟开源：HelloGCC社区和PLCT实验室做的努力

17:30 – 17:40 闪电1 周亚金 基于RISC-V的软硬件安全协同扩展

17:40 – 17:50 闪电2 詹荣开 V-Spec ISA：Keystone for RISC-V based HPC

17:50 – 18:00 闪电3 待定

18:00 – 18:10 闪电4 待定

话题：Csky Intro – what’s the meaning of a new arch for linux

内容简介：The csky architecture officially merged the main line in linux-4.20. Before that, eight architectures have just been removed from the main line. Many people ask what is the meaning of csky upstream? Also includes our colleagues. Here, we will give some examples to introduce the progress of the csky architecture in the past months and the value and significance of linux-csky. This is an open discussion about the csky architecture and any questions are welcomed.

个人简介：郭任，linux kernel csky maintainer, 目前专注于 CPU ISA,  IOMMU,  Perf。

话题：NVDLA开源编译器的功能分析和对比

内容简介：NVDLA是由NVIDIA公司推出的开源深度学习推理加速器，继2017年10月开源RTL代码、模拟器、Linux内核和驱动、用户态运行时环境后，在今年8月开源了编译器代码，支持Caffe框架的模型编译。从2018年年初，台湾编译技术公司Skymizer开始开源深度学习加速器编译框架ONNC，逐渐加入并完善了NVDLA后端的支持，从公开资料上看，ONNC的NVDLA后端比NVIDIA的官方编译器具有更多功能和性能的支持。本次报告将对这两个开源NVDLA编译器的实现和功能进行剖析和比较，并探讨深度学习编译器的框架设计和实现方法。

个人简介：邱吉，计算机系统结构博士，毕业于中科院计算所龙芯课题组，曾先后在龙芯中科和展讯通信参与多款自研架构处理器的设计验证，并主持工具链的开发，现任中科重德智能科技有限公司工具链组主管，负责通用处理器和各类领域专用处理器的工具链研发。

话题：托管堆直写技术初探–降低JVM的序列化开销

内容简介：序列化与反序列化操作被用于大数据系统的多种操作中。过去研究表明计算密集型的序列化反序列化是影响系统性能的一个显著瓶颈。现有的序列化库例如kryo虽然能够提供高性能的序列化机制，但在这些技术还是无法完全避免堆内堆外表现形式的转换过程。我们的堆对象直写技术将发送端数据直接写入到接收端的托管堆内中。工作流程是1）接收端通知发送端数据在堆内的地址；2）发送端遍历待发送对象，将其复制到发送缓冲区中，在复制过程中更新指针域，直接指向对象在远程地址的值；3）接收端接收数据，就绪后可直接使用。该技术几乎去掉了接收端的反序列化开销，而发送端的序列化操作几乎没有增加额外的开销。我们基于RDMA网络与Jikes RVM虚拟机实现了原型，其接收端的延迟基本等同于硬件延迟。对于发送端，虽然无法同样避免序列化，但是我们正在尝试将序列化的延迟隐藏于应用层的shuffle过程。

个人简介：朱凌宇，国防科技大学在读博士。主要研究兴趣包括基于持久化内存的文件系统与虚拟机内存管理技术。

话题：开源项目L2C：经过形式化验证的可信编译器

内容简介：本次话题和大家分享清华大学王生原老师的开源项目L2C，L2C是一个把工业界的模型语言Lustre转换成C语言到二进制代码的工具，由于整个转换过程经过了形式化验证，所以具有极高的安全性和可靠性，在安全攸关领域具有重要的应用价值。讨论分为两部分：第一部分、介绍形式化的编译器的背景，包括最近的一些进展，希望为没有听说过形式化的朋友做一些背景介绍。二、介绍我们的开源项目L2C，其软件架构，解决的问题，以及未来的一些展望。

个人简介：康烁，迪捷软件科技有限公司创始人，有超过17年的系统软件科研和工程经验，涉及操作系统、编译器、虚拟机、区块链和形式化等多个领域，主持和参与了多个国内外开源项目，且在实际工程中均得到广泛应用。其中，SkyEye全数字仿真产品应用于国内航空航天领域的众多型号的研发测试领域；符号执行软件android_s2e被华为以及国内军工单位应用于软件测试方面；基于LLVM的安卓虚拟机入选了2015年《LLVM开发者大会》的项目展示环节；参与的开源项目L2C被国内核电单位应用于相关设备中。

话题：Web Assembly & Go Implementation

主题简介：Wasm是运行在web系统（浏览器和node.js）的虚拟机，他比JavaScript的优势是：

1. 支持c++/Rust/go这样的高级语言；

2. 比JS的解释执行或JIT速度快；

本文将浅析go语言对wasm的支持；

个人简介：gopher，Go 社区活跃开发者，曾给 Go 提过多个编译器优化补丁并被采纳。

话题：在内核虚拟机与在内核服务

内容简介：自Linux内核3.15开始引入 eBPF (extended Berkeley Packet Filter)后, 快速演进和不断完善中的eBPF正在成为重构整个Linux网络、安全、调试/调优等子系统的基石。本话题将一探eBPF的原理与实现，分析基于eBPF的在内核服务设计，解析相关优秀开源项目，并在ARM开放平台上实践，还将重新思考基于Lua/LuaJIT的在内核虚拟机技术、内核空间和用户空间的划分以及基于eBPF的领域专用语言。

个人简介：李枫，先后就职于摩托罗拉、三星等IT公司，现为独立开发者。在移动开发/云计算上积累了十余年的研发经验。近期主要专注于物联网/边缘计算/人工智能基础设施。积极参与包括HelloGCC在内的开源社区各种活动，并多次做过技术分享，涉及编程语言、操作系统、边缘计算等领域。

话题：共促方舟开源：HelloGCC社区和PLCT实验室做的努力

内容简介：尽管网络上风评毁誉参半，华为方舟编译器的开源依然是国内编译技术领域的里程碑事件，让“编译器”这个概念被更多的人知晓。作为国内最早的编译技术社区，HelloGCC社区非常希望能够与方舟开源社区一起推动国内编译技术的发展与人才的培养。本次报告，我们将介绍软件所智能软件研究中心PLCT实验室与HelloGCC社区为推动和促进方舟开源社区的发展壮大所做的一系列尝试，包括：开发出第一个面向方舟编译器的 Toy Runtime、开设使用方舟编译器作为教学用具的编译技术学习班、组织方舟线下代码学习讨论会等。

个人简介：吴伟，HelloGCC社区和HelloLLVM社区负责人、中科重德CTO、PLCT实验室项目总监。主要研究领域包括程序语言与编译技术、系统性能分析和优化、基于ROS的机器人研发等。在方舟编译器开源之后，积极推动HelloGCC社区以及PLCT实验室参与方舟开源社区的建设。

闪电演讲1： 基于RISC-V的软硬件安全协同扩展

演讲者：周亚金，浙江大学网络空间安全学院

闪电演讲2：V-Spec ISA：Keystone for RISC-V based HPC

演讲者：詹荣开，北京希姆计算科技有限公司

闪电3、闪电4： TBD 欢迎现场报名。

感谢中国科学院软件研究所智能软件研究中心提供场地支持。感谢浙江迪捷软件科技有限公司提供小礼品赞助。

欢迎各位新老朋友通过点击原文报名或扫描二维码报名参加。

OSDT19报名 https://www.bagevent.com/event/6112393
