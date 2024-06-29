<!--yml

category: 未分类

date: 2024-05-27 13:14:48

-->

# NASA为Voyager 1的降级内存芯片问题制定解决方案 • The Register

> 来源：[https://www.theregister.com/2024/04/15/voyager_engineers_prepare_fix/](https://www.theregister.com/2024/04/15/voyager_engineers_prepare_fix/)

NASA的工程师们确定了一些受损内存是Voyager 1故障的原因，并正在进行远程修复以解决硬件问题。

这艘老兵探测器从[2023年底](https://www.theregister.com/2023/12/14/engineers_work_to_fix_voyager/)开始向地球发送无法阅读的数据，工程师们试图理解问题的性质。上个月，他们[发送了一个命令](https://www.theregister.com/2024/03/14/voyager_1_not_dead/) —— 对宇宙飞行器的飞行数据系统（FDS）进行了"探测" —— 结果数据流中包含了计算机的完整内存转储。

团队能够使用这个读出数据，其中包含了计算机的代码和变量，来确定大约3%的FDS存储器已经损坏。这种损坏阻碍了FDS的正常运行，该系统负责在将探测器的工程和科学数据传递给遥测调制单元（TMU）——无线电发射机并发送回地球之前打包这些数据。

Voyager团队[认为](https://blogs.nasa.gov/voyager/2024/04/04/engineers-pinpoint-cause-of-voyager-1-issue-are-working-on-solution/)，一个负责受损内存部分的单一芯片出了问题，尽管他们只能根据已知信息做出推测。

两种主要理论认为，这块芯片可能由于在太空中使用了46年而简单老化，或者可能是某个高能粒子造成了损坏。

由于Voyager 1已经远离了任何物理干预的范围，这个问题将需要在软件中进行解决。

根据NASA的说法："尽管可能需要几周或几个月的时间，工程师们乐观地认为他们可以找到一种方法，使FDS能够在没有可用内存硬件的情况下正常运行，这将使Voyager 1能够再次开始返回科学和工程数据。"

太空飞行器设计者长期以来一直面临由宇宙射线和高能粒子引起的计算机问题。有些问题可能只是导致位翻转，而其他问题可能导致卫星无法运行或受损。

Voyager 2在[2010年遭遇了位翻转](https://www.jpl.nasa.gov/news/engineers-diagnosing-voyager-2-data-system-update)，导致从飞船传输的科学数据出现问题，原因可以追溯到FDS。通过计算机重置解决了这个问题。

不过，这一次，硬件似乎已经失效，需要工程师们设计比简单的重启更复杂的解决方案。®
