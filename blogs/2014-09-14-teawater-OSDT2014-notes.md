转换时间： 2021-02-01

# 2014开源开发工具大会见闻录
Posted on 2014/09/14 by teawater

上午因为家里有事，没能看到上午两个精彩话题，也辛苦了明杰和吴伟以及开源协会的各位同学。

中午到达，下午第一个演讲是ARM来的王哲宇介绍的pyOCD，中午发困导致精神松懈，所以体会不深，整体感觉是一个用PYTHON写的OCD软件。
osdt2014_1
附图一张，总让人觉得快要从讲台上掉下来的王同学。

然后就是一代宗师LUBA的话题，因为一些不可抗力，话题从Xposed for ART on Android换成了Beyond Compiler Engineering，一位同学竟然愤然提起了抗议：“原来不是ART话题吗？”。
我现在真想对这位同学说，ART什么都是浮云，没准哪天突然就跟Dalvik一样就随手被GOOGLE换掉了。但是LUBA话题谈的显然是能走的更远的。
话题前一段总结起来就是摩尔定律逐渐衰落后给编译器带来了巨大的机会，最后他还拿出了一张图，对今后40年内，哪一年哪项技术会推动前进，没错具体到年的，不知道这张图是怎么来的，不过台下所有弄编译器的同学心里都兴奋了，沸腾了。明杰同学之后就一直在问我：“编译器真的这么有前途？”
然后话锋一转，大师开心的说，可以开始讲技术了。之后对GCC系和LLVM系进行了比较和分析外加小八卦。
osdt2014_2
忙着创业每天只能睡3个小时，仍然准备了双份精彩话题（有一份Xposed不能讲）的LUBA大神。

接着是硬件人中的软件人软件人中的硬件人——翟哥的话题。我刚进场没看见他，后来才知道，他想演示的板子还没搞定，带着表坐在最后忙活着呢。
最后到时间时还是没搞定，在我的建议下改为现场举起板子让大家看一眼。
整个话题都围绕OPENOCD展开，不过我主要注意点集中到了这玩意脚本支持的部分用的TCL，于是作为一个饱受GDB testsuite中TCL虐待的GDB开发者，在话题讨论时对使用TCL脚本进行了丧心病狂的吐槽，还试图拉齐姚同学一起来吐槽。
齐姚同学不愧是和稀泥的高手，分析说是因为OPENOCD是德国人作的，德国人作事力图严谨，肯定会在通读过整个TCL手册才会开弄。
osdt2014_3
挥舞着板子，致力于向广大屌丝码农提供全栈开源的方式（软硬件都开源）搭建ARM处理器的JTAG调试环境的翟哥。

下午最后一个常规话题是专注于模拟等方向的SKYEYE开发者康老师，小小炫耀一下，我曾经也是SKYEYE开发者。
他的话题比较详细的介绍了符号执行以及其和其他程序分析技术的区别等等，之后又演示了他们的Android的全系统符号执行工具-Android_S2E。
osdt2014_4
话题时间对他来说永远不够，被我们安排到最后一个可以随便拖时间很开心的康老师。

之后两个是闪电演讲，先是一位阿里的同学讲了JAVASCRIPT的相关内容，我走神了没太听仔细，希望看视频能仔细看看。另外阿里工具组我们早有耳闻，希望明年能看见他们的常规话题。
osdt2014_5
我没记住名字的阿里同学。

之后是我的老友，和我一样也是曾经的模拟器开发者，现在专注Android安全的在读博士亚金。对他话题的提问都是问他用什么系统的手机，用什么Android ROM的。
osdt2014_6
专注吓唬Android用户，看我站在台下忍住没黑我小米的亚金。

之后是欢乐的抽奖活动，然后就是更欢乐的演讲者聚餐（最快乐的是吐槽了没到场的同学，老唐我们没说你啦，嘿嘿！），明年再见啦各位。

====================================================================

补几张照片，算是凑成整个大会的见闻录。（by xmj）

上午两场都非常精彩，嘉宾演讲得精彩，工作本身也做得非常出色。对做工具链的同学来说，是很好的学习参考。

qi yao
齐尧，给大家讲述了gdb性能方面的故事

kito
Kito，把经典的寄存器分配算法用实例给大家展现了一遍

中午，有一位报名自由演讲的朋友因为下午有事，改在了午餐后的休息时间，我不在现场，但看起来人气很旺。我这里只有一张，他自己用开源软硬件做的飞行器照片：

开源软硬件搭建的飞行器
开源软硬件做的飞行器

下午，最后是抽奖和合影：
抽奖
盒子，音箱和书，抽奖是个体力活

最后的合影：
合影
大家都笑了，是因为茶总喊了句玩笑口号：“打倒llvm”。

另外，luba给稍微有点绕口的大会名称起了一个好记的绰号，“开开工”。:-)
