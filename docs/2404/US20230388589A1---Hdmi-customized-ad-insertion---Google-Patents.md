<!--yml

类别：未分类

日期：2024年5月27日 12:53:58

-->

# US20230388589A1 - HDMI定制广告插入 - Google专利

> 来源：[https://patents.google.com/patent/US20230388589A1/en](https://patents.google.com/patent/US20230388589A1/en)

<heading id="h-0001">相关申请的交叉引用</heading>

+   <para-num num="[0001]"></para-num>

    本申请是美国申请号17/674,339的继续申请，该申请于2022年2月17日提交，并完全纳入本文。

<heading id="h-0002">背景</heading>

<heading id="h-0003">领域</heading>

+   <para-num num="[0002]"></para-num>

    本公开通常涉及插入广告以显示，更具体地涉及基于暂停的媒体内容插入定制广告。

<heading id="h-0004">背景</heading>

+   <para-num num="[0003]"></para-num>

    媒体内容（如电影或电视节目）通常显示在电视或其他显示屏上供用户观看。因此，用户对内容的体验通常限于电视和连接到电视的扬声器。用户可以通过选择媒体内容进行流媒体播放、播放、暂停和取消暂停来控制媒体服务的消费。

<heading id="h-0005">摘要</heading>

+   <para-num num="[0004]"></para-num>

    本文提供了高清多媒体接口（HDMI）定制广告插入的系统、装置、制造物品、方法和/或计算机程序产品实施例以及/或其组合和子组合。一个示例实施例通过检测通过HDMI连接接收的一个或多个帧（例如音频或视频帧）中的暂停事件，并在暂停事件期间识别帧内的内容来运行。一些实施例包括通过HDMI连接接收控制信号并确定源设备的信息和/或运行在源设备上的应用程序的信息。一些实施例包括根据i）帧内的内容和/或ii）关于源设备的信息和/或运行在源设备上的应用程序的信息来确定广告。广告可以显示在显示设备上。

+   <para-num num="[0005]"></para-num>

    一些实施例包括显示设备检测到通过HDMI连接接收的一个或多个帧中的暂停事件，并在暂停事件期间识别一或多个帧内的内容。一些实施例包括基于一或多个帧内的内容确定广告，并在显示设备上呈现广告。检测暂停事件可以包括：i）接收远程控制传递暂停信号；ii）通过HDMI连接检测到静音音频信号，并确定一个或多个帧的视频帧未更改；和/或iii）使用计算机视觉（CV）技术从一个或多个帧中检测到暂停图标，并检测到从第一个视频帧到第二个视频帧未发生变化，其中第一个视频帧和第二个视频帧属于一个或多个帧。

+   <para-num num="[0006]"></para-num>

    在一些实施例中，识别内容包括使用自动内容识别（ACR）技术分析一个或多个帧的第一个视频帧或第一个音频帧，并确定与第一个视频帧或第一个音频帧对应的指纹、水印或提示音，其中指纹、水印和/或提示音用于识别与第一个视频帧或第一个音频帧对应的内容信息。在一些实施例中，识别内容包括使用计算机视觉（CV）技术分析一个或多个帧的第一个视频帧，并确定与第一个视频帧对应的元数据，其中元数据用于识别与第一个视频帧对应的一个或多个对象。

+   <para-num num="[0007]"></para-num>

    一些实施例包括通过HDMI连接接收控制信号，并从控制信号确定服务产品描述（SPD），其中广告基于SPD确定。一些实施例进一步包括从控制信号检测到自动低延迟模式（ALLM），其中广告基于ALLM。一些实施例进一步包括从控制信号检测到可变刷新率（VRR），其中广告基于VRR。

+   <para-num num="[0008]"></para-num>

    在一些实施例中，确定广告包括将VRR传输到网络，并从网络接收与VRR对应的广告。在一些例子中，确定广告包括将与内容对应的指纹、水印、提示音或元数据传输到网络，并从网络接收相应的广告。

+   <para-num num="[0009]"></para-num>

    一些实施例包括通过HDMI连接接收的控制信号对应的SPD或ALLM的传输到网络，并从网络接收与SPD或ALLM对应的广告。一些实施例包括在显示设备的图形平面上呈现广告，将图形平面与包含一个或多个帧的第一个视频帧的视频平面混合，并在显示设备上呈现混合平面。

+   <para-num num="[0010]"></para-num>

    进一步实施例、特征和本公开的优点，以及本公开的各种实施例的结构和操作，将在下文详细描述，参考附图。

<description-of-drawings><heading id="h-0006">图示详述</heading>*   <para-num num="[0011]"></para-num>

    附图已纳入本说明并构成其一部分。

    +   <para-num num="[0012]"></para-num>

    <figref idrefs="DRAWINGS">图 **1**</figref> 描述了根据某些实施例的多媒体环境的块图。

    +   <para-num num="[0013]"></para-num>

    <figref idrefs="DRAWINGS">图 **2**</figref> 描述了根据某些实施例的流媒体设备的块图。

    +   <para-num num="[0014]"></para-num>

    <figref idrefs="DRAWINGS">图 **3**</figref> 描述了根据某些实施例的显示设备的第一个块图。

    +   <para-num num="[0015]"></para-num>

    <figref idrefs="DRAWINGS">图 **4**</figref> 描述了根据某些实施例的显示设备的方法。

    +   <para-num num="[0016]"></para-num>

    <figref idrefs="DRAWINGS">图 **5**</figref> 描述了一个实施各种实施例的计算机系统的示例。

    +   <para-num num="[0017]"></para-num>

    <figref idrefs="DRAWINGS">图 **6**</figref> 描述了根据某些实施例的第二媒体设备的块图。

    +   <para-num num="[0018]"></para-num>

    <figref idrefs="DRAWINGS">图 **7**A</figref> 描述了根据某些实施例的第二显示设备的块图。

    +   <para-num num="[0019]"></para-num>

    <figref idrefs="DRAWINGS">图 **7**B</figref> 描述了根据某些实施例的第三显示设备的块图。

    +   <para-num num="[0020]"></para-num>

    <figref idrefs="DRAWINGS">图 **8**</figref> 描述了根据某些实施例的第三媒体设备的块图。

    +   <para-num num="[0021]"></para-num>

    <figref idrefs="DRAWINGS">图 **9**</figref> 描述了根据某些实施例的第四显示设备的块图。</description-of-drawings>

+   <para-num num="[0022]"></para-num>

    在图中，类似的参考编号通常表示相同或相似的元素。此外，参考编号的最左边的数字标识了参考编号首次出现的图纸。

<heading id="h-0007">详细说明</heading>

+   <para-num num="[0023]"></para-num>

    提供了通过高清媒体接口（HDMI）连接的显示设备进行广告插播的系统、装置、设备、方法和/或计算机程序产品实施例及/或其组合和子组合，以及通过媒体设备向媒体内容提供媒体内容的情况。当媒体设备暂停媒体内容时，显示设备可以确定发生了暂停事件，并在显示设备上插播广告。换句话说，检测到暂停事件可以触发相关广告的插播。在检测到暂停事件之后，一些实施例包括确定被暂停的媒体内容的上下文和/或内容，并选择根据确定的上下文和/或内容定制的广告在显示设备上显示。在一些实施例中，显示设备可以通过除HDMI之外的媒体接口连接到媒体设备。

+   <para-num num="[0024]"></para-num>

    本公开的各种实施例可以使用和/或可以是 <figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 如 <figref idrefs="DRAWINGS">图 **1**</figref> 所示。然而需要注意的是，<figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 仅供说明目的，不具有限制性。根据本文所包含的教导，本公开的实施例可以使用和/或可以是不同于和/或额外于<figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 的环境中实施，如技术领域相关人员将会理解的那样。现在将描述<figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 的一个示例。

<heading id="h-0008">多媒体环境</heading>

+   <para-num num="[0025]"></para-num>

    <figref idrefs="DRAWINGS">图 **1**</figref> 展示了根据某些实施例的 <figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 的块图。在一个非限定性的示例中，<figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 可用于流媒体。但是，本公开适用于任何类型的媒体（而不仅限于流媒体），以及用于分发媒体的任何机制、手段、协议、方法和/或过程。

+   <para-num num="[0026]"></para-num>

    <figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 可包括一个或 <figure-callout id="104" label="更多媒体系统" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">更多媒体系统</figure-callout> **104**。一个 <figure-callout id="104" label="媒体系统" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">媒体系统</figure-callout> **104** 可代表家庭房间、厨房、后院、家庭影院、学校教室、图书馆、汽车、船、公共汽车、飞机、电影院、体育场、礼堂、公园、酒吧、餐厅或任何其他需要接收和播放流媒体内容的位置或空间。用户 **132** 可以与 <figure-callout id="104" label="媒体系统" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">媒体系统</figure-callout> **104** 协作选择和消费内容。

+   <para-num num="[0027]"></para-num>

    每个 <figure-callout id="104" label="媒体系统" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">媒体系统</figure-callout> **104** 可能包括一个或 <figure-callout id="106" label="更多媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">更多媒体设备</figure-callout> **106**，每个设备连接到一个或 <figure-callout id="108" label="更多显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">更多显示设备</figure-callout> **108**。注意，“连接”，“连接到”，“附加”，“链接”，“组合”等术语可能指物理、电气、磁性、逻辑等连接，除非另有规定。

+   <para-num num="[0028]"></para-num>

    <figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可能是流媒体设备、DVD 或 BLU-RAY 设备、音视频播放设备、有线电视盒以及/或数字视频录制设备，仅举几例。<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可能是监视器、电视（TV）、计算机、智能手机、平板电脑、可穿戴设备（如手表或眼镜）、家电、物联网（IoT）设备以及/或投影仪，仅举几例。在某些实施例中，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可以是其<figure-callout id="108" label="相应显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">相应显示设备</figure-callout> **108** 的一部分、集成于其内、操作连接或连接至其之中。

+   <para-num num="[0029]"></para-num>

    每个<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可配置为通过<figure-callout id="114" label="通信设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">通信设备</figure-callout> **114** 与<figure-callout id="118" label="网络" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">网络</figure-callout> **118** 进行通信。<figure-callout id="114" label="通信设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">通信设备</figure-callout> **114** 可包括例如有线调制解调器或卫星电视接收器。<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可通过<figure-callout id="116" label="链接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">链接</figure-callout> **116** 与<figure-callout id="114" label="通信设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">通信设备</figure-callout> **114** 进行通信，其中<figure-callout id="116" label="链接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">链接</figure-callout> **116** 可包括无线（如WiFi）和/或有线连接。

+   <para-num num="[0030]"></para-num>

    在各种实施例中，<figure-callout id="118" label="network" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">网络</figure-callout> **118** 可以包括但不限于有线和/或无线的企业内部网络、外部网络、互联网、蜂窝网络、蓝牙、红外线以及/或任何其他短距离、长距离、本地、区域、全球通信机制、手段、方法、协议和/或网络，以及它们的任意组合。

+   <para-num num="[0031]"></para-num>

    <figure-callout id="104" label="Media system" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">媒体系统</figure-callout> **104** 可能包括一个<figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110**。这个<figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 可以是任何用于控制<figure-callout id="106" label="media device" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 和/或<figure-callout id="108" label="display device" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 的组件、部件、设备和/或方法，例如遥控器、平板电脑、笔记本电脑、智能手机、可穿戴设备、屏幕上的控制按钮、集成控制按钮、音频控制，或它们的任意组合，仅举几例。在一种实施例中，<figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 通过蜂窝网络、蓝牙、红外线等方式或它们的任意组合与<figure-callout id="106" label="media device" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 和/或<figure-callout id="108" label="display device" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 进行无线通信。这个<figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 可能包括一个<figure-callout id="112" label="microphone" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">麦克风</figure-callout> **112**，下文将进一步描述。

+   <para-num num="[0032]"></para-num>

    <figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 可能包括多个内容服务器 **120**（也称为内容提供者、频道或来源）。虽然图 **1** 中仅显示了一个<figure-callout id="120" label="内容服务器" filenames="US20230388589A1-20231130-D00001.png" state="{{state}}">内容服务器</figure-callout> **120** ，但在实际操作中，<figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 可能包括任意数量的<figure-callout id="120" label="内容服务器" filenames="US20230388589A1-20231130-D00001.png" state="{{state}}">内容服务器</figure-callout> **120**。 每个<figure-callout id="120" label="内容服务器" filenames="US20230388589A1-20231130-D00001.png" state="{{state}}">内容服务器</figure-callout> **120** 可以配置为与<figure-callout id="118" label="网络" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">网络</figure-callout> **118** 进行通信。

+   <para-num num="[0033]"></para-num>

    每个<figure-callout id="120" label="内容服务器" filenames="US20230388589A1-20231130-D00001.png" state="{{state}}">内容服务器</figure-callout> **120** 可能存储<figure-callout id="122" label="内容" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">内容</figure-callout> **122** 和<figure-callout id="124" label="元数据" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">元数据</figure-callout> **124**。 <figure-callout id="122" label="内容" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">内容</figure-callout> **122** 可能包括音乐、视频、电影、电视节目、多媒体、图像、静止图片、文本、图形、游戏应用程序、广告、编程内容、公益内容、政府内容、地方社区内容、软件和/或任何其他电子形式的内容或数据对象的任意组合。

+   <para-num num="[0034]"></para-num>

    在某些实施例中，<figure-callout id="124" label="metadata" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">元数据</figure-callout> **124** 包含关于<figure-callout id="122" label="content" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">内容</figure-callout> **122** 的数据。例如，<figure-callout id="124" label="metadata" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">元数据</figure-callout> **124** 可能包括与作者、导演、制片人、作曲家、艺术家、演员、摘要、章节、制作、历史、年份、预告片、备选版本、相关内容、应用程序及/或与<figure-callout id="122" label="content" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">内容</figure-callout> **122** 相关的其他信息相关或有关的相关信息。<figure-callout id="124" label="Metadata" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">元数据</figure-callout> **124** 也可以或者另外包括指向任何与<figure-callout id="122" label="content" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">内容</figure-callout> **122** 相关的此类信息的链接。<figure-callout id="124" label="Metadata" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">元数据</figure-callout> **124** 也可以或者另外包括一个或多个与<figure-callout id="122" label="content" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">内容</figure-callout> **122** 相关的索引，例如但不限于技巧模式索引。

+   <para-num num="[0035]"></para-num>

    <figure-callout id="102" label="多媒体环境" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">多媒体环境</figure-callout> **102** 可包括一个或<figure-callout id="126" label="更多系统服务器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">更多系统服务器</figure-callout> **126**。 <figure-callout id="126" label="系统服务器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">系统服务器</figure-callout> **126** 可用于支持云端的<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**。注意，<figure-callout id="126" label="系统服务器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">系统服务器</figure-callout> **126** 的结构和功能方面可能完全或部分存在于相同或不同的<figure-callout id="126" label="系统服务器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">系统服务器</figure-callout> **126**。

+   <para-num num="[0036]"></para-num>

    <figure-callout id="126"

    label="system servers"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">系统服务器</figure-callout> **126** 还可以包括音频<figure-callout id="130"

    label="command processing module"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">命令处理模块</figure-callout> **130**。

    如上所述，<figure-callout id="110" 

    label="遥控器"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">遥控器</figure-callout> **110** 可包括一个<figure-callout id="112"

    label="麦克风"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">麦克风</figure-callout> **112**。

    <figure-callout id="112"

    label="microphone"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">麦克风</figure-callout> **112** 可接收来自用户**132**（以及其他来源，如显示设备**108**）的音频数据。

    在某些实施例中，<figure-callout id="106" 

    label="media device"

    filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png"

    state="{{state}}">媒体设备</figure-callout> **106** 可响应音频，并且音频数据可以表示来自<figure-callout id="132"

    label="user"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">用户</figure-callout> **132** 控制 <figure-callout id="106"

    label="媒体设备"

    filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png"

    state="{{state}}">媒体设备</figure-callout> **106** 以及 <figure-callout id="104"

    label="媒体系统"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">媒体系统</figure-callout> **104** 中，例如 <figure-callout id="108"

    label="display device"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">显示设备</figure-callout> **108**。

    在某些实施例中，由 <figure-callout id="112"

    label="麦克风"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">麦克风</figure-callout> **112** 在 <figure-callout id="110"

    label="遥控器"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">遥控器</figure-callout> **110** 被传送到 <figure-callout id="106"

    label="媒体设备"

    filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png"

    state="{{state}}">媒体设备</figure-callout> **106**，然后转发到音频 <figure-callout id="130"

    label="command processing module"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">命令处理模块</figure-callout> **130** 在 <figure-callout id="126"

    label="系统服务器"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">系统服务器</figure-callout> **126**。

    音频 <figure-callout id="130"

    label="command processing module"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">命令处理模块</figure-callout> **130** 可能操作以处理和分析接收的音频数据，以识别 <figure-callout id="132"

    label="用户"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">用户</figure-callout> **132** 的口头命令。

    音频 <figure-callout id="130"

    label="command processing module"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">命令处理模块</figure-callout> **130** 然后将口头命令转发回到 <figure-callout id="106"

    label="媒体设备"

    filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png"

    state="{{state}}">媒体设备</figure-callout> **106** 进行处理。

+   <para-num num="[0037]"></para-num>

    在某些实施例中，音频数据可能会被音频**命令处理模块** **216** 在媒体设备 **106** 中替代性或附加地处理和分析（参见<figref idrefs="DRAWINGS">图 **2**</figref>）。媒体设备 **106** 和系统服务器 **126** 可能会合作选择要处理的口头命令之一（即由音频**命令处理模块** **130** 在系统服务器 **126** 中识别的口头命令，或由音频**命令处理模块** **216** 在媒体设备 **106** 中识别的口头命令）。

+   <para-num num="[0038]"></para-num>

    <figref idrefs="DRAWINGS">图 **2**</figref> 描述了根据某些实施例的一个示例媒体设备 **106** 的块图。示例媒体设备 **106** 可能包括一个 **流媒体模块** **202**，一个 **处理模块** **204**，存储/缓冲区 **208**，以及一个 **用户界面模块** **206**。正如上述，用户界面模块 **206** 可能包括音频 **命令处理模块** **216**。

+   <para-num num="[0039]"></para-num>

    媒体设备 **106** 还可能包括一个或多个 **音频解码器** **212** 和一个或多个 **视频解码器** **214**。

+   <para-num num="[0040]"></para-num>

    每个 **音频解码器** **212** 可配置为解码一个或多个音频格式，例如但不限于 AAC、HE-AAC、AC3（杜比数字）、EAC3（杜比数字增强版）、WMA、WAV、PCM、MP3、OGG GSM、FLAC、AU、AIFF 和/或 VOX，仅列举一些例子。

+   <para-num num="[0041]"></para-num>

    同样，每个<figure-callout id="214" label="video decoder" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">视频解码器</figure-callout> **214** 可配置为解码一种或多种视频格式，例如但不限于 MP4（mp4、m4a、m4v、f4v、f4a、m4b、m4r、f4b、mov）、3GP（3gp、3gp2、3g2、3gpp、3gpp2）、OGG（ogg、oga、ogv、ogx）、WMV（wmv、wma、asf）、WEBM、FLV、AVI、QuickTime、HDV、MXF（OPla、OP-Atom）、MPEG-TS、MPEG-2 PS、MPEG-2 TS、WAV、Broadcast WAV、LXF、GXF 和/或 VOB 等。每个<figure-callout id="214" label="video decoder" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">视频解码器</figure-callout> **214** 可包括一种或多种视频编解码器，例如但不限于 H.263、H.264、HEV、MPEG1、MPEG2、MPEG-TS、MPEG-4、Theora、3GP、DV、DVCPRO、DVCProHD、IMX、XDCAM HD、XDCAM HD422 和/或 XDCAM EX 等。

+   <para-num num="[0042]"></para-num>

    现在参考 <figref idrefs="DRAWINGS">图 **1** 和 **2**</figref> ，在某些实施例中，<figure-callout id="132" label="user" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 可以通过例如 <figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 与 <figure-callout id="106" label="media device" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 进行交互。例如，<figure-callout id="132" label="user" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 可以使用 <figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 与 <figure-callout id="206" label="user interface module" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">用户界面模块</figure-callout> **206** 的 <figure-callout id="106" label="media device" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 进行交互，选择内容，例如电影、电视节目、音乐、书籍、应用程序、游戏等。 <figure-callout id="202" label="streaming module" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">流媒体模块</figure-callout> **202** 可以从内容服务器 **120** 通过 <figure-callout id="118" label="network" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">网络</figure-callout> **118** 请求所选内容。内容服务器 **120** 可以将请求的内容传输给 <figure-callout id="202" label="streaming module" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">流媒体模块</figure-callout> **202**。 <figure-callout id="106" label="media device" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可以将接收到的内容传输给 <figure-callout id="108" label="display device" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 进行播放，以供 <figure-callout id="132" label="user" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 观看。

+   <para-num num="[0043]"></para-num>

    在流媒体实现中，<figure-callout id="202" label="流媒体模块" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">流媒体模块</figure-callout> **202** 可以在从内容服务器 **120** 接收到内容后，实时或几乎实时地将内容传输到<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**。在非流媒体实现中，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可能会将从内容服务器 **120** 接收到的内容存储在存储/<figure-callout id="208" label="缓冲区" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">缓冲区</figure-callout> **208** 中，以便稍后在<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 上播放。

<heading id="h-0009">HDMI 自定义广告插入</heading>

+   <para-num num="[0044]"></para-num>

    参考<figref idrefs="DRAWINGS">图 **1**</figref>，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可能是一个流媒体设备（例如机顶盒或游戏设备），通过 HDMI 连接 **107** （例如 HDMI 线缆）与<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 连接。例如，<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 可以选择 HDMI 作为媒体流媒体的输入源，用于在<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**上播放媒体。<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 可以使用<figure-callout id="110" label="遥控器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110**与<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**通过<figure-callout id="111" label="无线接口" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">无线接口</figure-callout> **111**通信。在先前的解决方案中，当<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132**暂停由媒体设备**106**（例如源设备）提供的媒体内容时，相应的视频帧可能会暂停（例如停止）。在一些<figure-callout id="106" label="例子中的媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">例子中的媒体设备</figure-callout> **106**可能会通过信号传输一个暂停图标，并且该暂停图标可能会显示在显示设备**108**（例如智能电视）上。在先前的解决方案中，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**没有检测到暂停事件。因此，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**不会在暂停事件期间插入广告，或者分析暂停的媒体内容和/或上下文以确定定制广告，并在<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**的图形平面上显示定制广告。

+   `<para-num num="[0045]"></para-num>`

    <figref idrefs="DRAWINGS">图 **6**</figref> 描述了根据某些实施例的<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 的框图。出于说明目的而非限制，可以参考公开文件中其他图中的元素描述<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600**。例如，<figure-callout id="680" label="HDMI 输出" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">HDMI 输出</figure-callout> **680** 可支持<figure-callout id="107" label="HDMI 连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI 连接</figure-callout> **107**，该连接在<figref idrefs="DRAWINGS">图 **1**</figref> 中，其中<figure-callout id="107" label="HDMI 连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI 连接</figure-callout> **107** 连接到<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 在<figref idrefs="DRAWINGS">图 **1**</figref> 中。用户界面模块 **620** 和音频<figure-callout id="625" label="命令处理模块" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">命令处理模块</figure-callout> **625** 可对应于<figure-callout id="206" label="用户界面模块" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">用户界面模块</figure-callout> **206** 和音频<figure-callout id="216" label="命令处理模块" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">命令处理模块</figure-callout> **216** 在<figref idrefs="DRAWINGS">图 **2**</figref> 中。处理器 **630** 可对应于处理模块<figure-callout id="204" label="处理模块" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">处理模块</figure-callout> **204** 在<figref idrefs="DRAWINGS">图 **2**</figref> 中。缓冲区/<figure-callout id="640" label="存储" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">存储</figure-callout> **640**，音频解码器 **660**，和视频解码器 **650** 可对应于存储/<figure-callout id="208" label="缓冲区" filenames="US20230388589A1-20231130-D00002.png" state="{{state}}">缓冲区</figure-callout> **208**，音频解码器 **212**，和视频解码器 **214** 在<figref idrefs="DRAWINGS">图 **2**</figref> 中。

+   <para-num num="[0046]"></para-num>

    <figure-callout id="600"

    label="媒体设备"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">媒体设备</figure-callout> **600** 包括<figure-callout id="610"

    label="应用程序"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">应用程序</figure-callout> **610**，其中可以包括<figure-callout id="612"

    label="流媒体应用程序"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">流媒体应用程序</figure-callout> **612**，广播/<figure-callout id="614"

    label="调谐器"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">调谐器</figure-callout> **614**，USB 输入 **616**，和<figure-callout id="618"

    label="游戏应用程序"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">游戏应用程序</figure-callout> **618**。

    在某些实施例中，<figure-callout id="600"

    label="媒体设备"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">媒体设备</figure-callout> **600** 可以成为游戏设备。

    <figure-callout id="610"

    label="应用程序"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">应用程序</figure-callout> **610** 可作为信息来源。

    <figure-callout id="630"

    label="处理器"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">处理器</figure-callout> **630** 可从相应应用程序中提取音频和视频数据，并将提取的音频和视频数据放入缓冲区/<figure-callout id="640"

    label="存储"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">存储</figure-callout> **640**.

    音频和视频解码后，<figure-callout id="670"

    label="后处理模块"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">后处理模块</figure-callout> **670** 可以执行图像缩放、颜色改善和唇同步等功能，例如，在通过<figure-callout id="680"

    label="HDMI 输出"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">HDMI 输出</figure-callout> **680** 到显示设备（例如，<figure-callout id="108"

    label="显示设备"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">显示设备</figure-callout> **108** of <figref idrefs="DRAWINGS">FIG.

    **1**</figref>，<figure-callout id="108"

    label="显示设备"

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png"

    state="{{state}}">显示设备</figure-callout> **108** of <figref idrefs="DRAWINGS">FIG.

    **3**</figref>，<figure-callout id="700"

    label="显示设备"

    filenames="US20230388589A1-20231130-D00007.png"

    state="{{state}}">显示设备</figure-callout> **700** of <figref idrefs="DRAWINGS">FIG.

    **7**A</figref>，<figure-callout id="705"

    label="显示设备"

    filenames="US20230388589A1-20231130-D00008.png"

    state="{{state}}">显示设备</figure-callout> **705** of <figref idrefs="DRAWINGS">图

    **7**B</figref>，和/或<figure-callout id="900"

    标签="显示设备"

    filenames="US20230388589A1-20231130-D00010.png"

    state="{{state}}">显示设备</figure-callout> **900** of <figref idrefs="DRAWINGS">图

    **9**</figref>）。

    在某些实施例中，<figure-callout id="600"

    标签="媒体设备"

    filenames="US20230388589A1-20231130-D00006.png"

    state="{{state}}">媒体设备</figure-callout> **600**传输服务产品描述（SPD）、自动低延迟模式（ALLM）或可变刷新率（VRR）到<figure-callout id="700"

    标签="显示设备"

    filenames="US20230388589A1-20231130-D00007.png"

    state="{{state}}">显示设备</figure-callout> **700** of <figref idrefs="DRAWINGS">图

    **7**A</figref>，<figure-callout id="705"

    标签="显示设备"

    filenames="US20230388589A1-20231130-D00008.png"

    state="{{state}}">显示设备</figure-callout> **705** of <figref idrefs="DRAWINGS">图

    **7**B</figref>，或<figure-callout id="900"

    标签="显示设备"

    filenames="US20230388589A1-20231130-D00010.png"

    state="{{state}}">显示设备</figure-callout> **900** of <figref idrefs="DRAWINGS">图

    **9**</figref>。

+   <para-num num="[0047]"></para-num>

    <figref idrefs="DRAWINGS">图 **8**</figref> 描述了根据某些实施例的<figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 的块图。为了说明而非限制，可以参考披露中其他图中的元素来描述<figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800**。例如，<figure-callout id="880" label="HDMI输出" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">HDMI输出</figure-callout> **880**，<figure-callout id="830" label="处理器" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">处理器</figure-callout> **830**，缓冲区/ <figure-callout id="840" label="存储" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">存储</figure-callout> **840**，<figure-callout id="860" label="音频解码器" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">音频解码器</figure-callout> **860** 和<figure-callout id="850" label="视频解码器" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">视频解码器</figure-callout> **850** <figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 可能对应于<figure-callout id="680" label="HDMI输出" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">HDMI输出</figure-callout> **680**，<figure-callout id="630" label="处理器" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">处理器</figure-callout> **630**，缓冲区/ <figure-callout id="640" label="存储" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">存储</figure-callout> **640**，<figure-callout id="660" label="音频解码器" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">音频解码器</figure-callout> **660** 和<figure-callout id="650" label="视频解码器" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">视频解码器</figure-callout> **650** 的<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 图中的<figref idrefs="DRAWINGS">图 **6**</figref>。

+   <para-num num="[0048]"></para-num>

    <figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 包括 <figure-callout id="810" label="应用程序" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">应用程序</figure-callout> **810**，可以是数据源，包括但不限于输入 <figure-callout id="816" label="应用程序" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">应用程序</figure-callout> **816**、<figure-callout id="814" label="网络应用程序" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">网络应用程序</figure-callout> **814** 和媒体应用程序 **817**。 <figure-callout id="816" label="输入应用程序" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">输入应用程序</figure-callout> **816** 可以包括但不限于对应摇杆、麦克风和/或遥控设备的输入应用程序。 <figure-callout id="814" label="网络应用程序" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">网络应用程序</figure-callout> **814** 可以包括但不限于Wi-Fi™、蓝牙™、Zigbee™ 和/或调谐器。 媒体应用程序 **817** 可以包括但不限于传输流、电影、游戏和/或音乐。

+   <para-num num="[0049]"></para-num>

    <figure-callout id="830" label="处理器" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">处理器</figure-callout> **830** 可以从相应的应用程序中提取音频和视频数据，并将提取的音频和视频数据放入缓冲区/<figure-callout id="840" label="存储" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">存储</figure-callout> **840**。音频和视频解码后，输出视频帧通过 <figure-callout id="880" label="HDMI 输出" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">HDMI 输出</figure-callout> **880** 传输到显示设备（例如，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">图 **1**</figure-callout> 的显示设备 **108**，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">图 **3**</figure-callout> 的显示设备 **108**，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">图 **7**A</figure-callout> 的显示设备 **700**，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">图 **7**B</figure-callout> 的显示设备 **705**，以及/或者 <figure-callout id="900" label="显示设备" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">图 **9**</figure-callout> 的显示设备 **900**）。在某些实施例中，<figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 向显示 <figure-callout id="700" label="设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">设备</figure-callout> **700**、<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">图 **7**B</figure-callout> 的显示设备 **705**，或者 <figure-callout id="900" label="显示设备" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">图 **9**</figure-callout> 的显示设备 **900** 传输 SPD、ALLM 或 VRR。  

+   <para-num num="[0050]"></para-num>

    <figref idrefs="DRAWINGS">图 **3**</figref> 描述了根据某些实施例的<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**的框图。仅供说明，不限于<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**可能会参考披露中其他图中的元素来描述。例如，<figure-callout id="310" label="HDMI模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">HDMI模块</figure-callout> **310**可以支持<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107**，其位于<figref idrefs="DRAWINGS">图 **1**</figref>中的<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107**与<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**连接。<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**可以包括<figure-callout id="320" label="ACR模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> **320**，<figure-callout id="325" label="CV模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV模块</figure-callout> **325**，<figure-callout id="330" label="广告插入模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330**，<figure-callout id="340" label="处理模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">处理模块</figure-callout> **340**，视频缩放和帧速率转换（FRC）<figure-callout id="350" label="模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">模块</figure-callout> **350**，<figure-callout id="360" label="图形模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">图形模块</figure-callout> **360**，<figure-callout id="370" label="显示模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">显示模块</figure-callout> **370**，和<figure-callout id="380" label="内存" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">内存</figure-callout> **380**。<figure-callout id="390" label="总线" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">总线</figure-callout> **390**可以连接<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**的各个元素。

+   <para-num num="[0051]"></para-num>

    <figure-callout id="320" label="ACR模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> **320** 能够分析视频帧并生成带有视频帧内容信息的唯一指纹和/或唯一水印。例如，这些唯一指纹和/或水印可以包括视频帧内的多个内容片段。这些唯一指纹和/或水印可以与已知的参考指纹或参考水印进行比对，以基于匹配来识别视频内容。匹配可能识别出电影标题、演员、评级、可观看电影的位置、制作公司等信息。在某些示例中，唯一指纹和/或水印可以对应于视频帧本身的特定内容。可以比较从连续视频帧生成的不同指纹和/或水印。如果生成的两个或更多指纹和/或水印相同，这可能表示内容没有变化（例如，视频或电影可能已暂停）。

+   <para-num num="[0052]"></para-num>

    在某些实施例中，<figure-callout id="320" label="ACR模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> **320** 能够分析音频帧并生成带有音频帧内容信息的唯一指纹和/或唯一水印。例如，这些唯一指纹和/或水印可以包括音频帧内的多个内容片段。这些唯一指纹和/或水印可以与已知的参考指纹或参考水印进行比对，以基于匹配来识别音频内容。匹配可能识别出电影标题、演员、评级、可观看电影的位置、制作公司等信息。在某些示例中，唯一指纹和/或水印可以对应于音频帧本身的特定内容。可以比较从连续音频帧生成的不同指纹和/或水印。如果生成的两个或更多指纹和/或水印相同，这可能表示内容没有变化（例如，音频或电影可能已暂停）。

+   <para-num num="[0053]"></para-num>

    在某些<figure-callout id="320" label="实施例 ACR 模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">实施例 ACR 模块</figure-callout> **320** 能够分析视频帧和/或音频帧并生成符合美国国家标准学会（ANSI）/有线电视工程师协会（SCTE）标准的唯一提示音（例如，符合ANSI/SCTE 35-2013定义的提示音），其中包含视频帧和/或音频帧内容的信息。

+   <para-num num="[0054]"></para-num>

    <figure-callout id="325" label="CV 模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV 模块</figure-callout> **325** 可以分析视频帧并生成元数据。该元数据可用于识别视频帧内的对象，或者一个或多个视频帧。例如，元数据可以识别视频帧中的图像和/或对象。元数据示例包括视频帧中的物品，如自行车、汽车、啤酒瓶、山和/或风景。

+   <para-num num="[0055]"></para-num>

    <figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可以通过以下方式检测暂停事件：i）使用遥控器穿透功能；ii）静音音频信号；和/或 iii）暂停图标识别。

+   <para-num num="[0056]"></para-num>

    在某些实施例中，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可能使用远程控制穿透功能来检测暂停事件。例如，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 可能配置支持消费电子控制（CEC）（例如，远程控制穿透）功能，包括但不限于：开机、关机、暂停键、播放键等。例如，<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 使用<figure-callout id="110" label="遥控器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 可以通过<figure-callout id="111" label="无线接口" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">无线接口</figure-callout> **111** 将暂停键传输至传输暂停键信号至显示<figure-callout id="108" label="设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">设备</figure-callout> **108**。在此示例中，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可以根据从媒体设备通过<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107** 收到的暂停键信号轻松检测到暂停事件。可以通过处理<figure-callout id="340" label="模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">模块</figure-callout> **340** 中的硬件、执行存储在<figure-callout id="380" label="内存" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">内存</figure-callout> **380** 中的软件或二者的组合来执行暂停事件的检测。例如，<figure-callout id="340" label="处理模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">处理模块</figure-callout> **340** 可能包括一个或多个处理器。

+   <para-num num="[0057]"></para-num>

    在某些实施例中，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可以基于从<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**接收到的静默音频信号检测暂停事件，以及确定从<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**接收到的一个或多个视频帧未发生变化。结合静默音频信号和未变化的视频帧可以用来确定检测到暂停事件。静默音频信号可以是对应于从<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**接收到的一个或多个视频或音频帧的音频信号，可能不在人类可听频率范围内。换句话说，静默音频信号的音量很低，以至于<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132**可能听不到静默音频信号，但静默音频信号确实存在。除了检测到静默音频信号外，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 还可以利用周期采样确定接收到的一个或多个视频帧未发生变化。在一些示例中，<figure-callout id="320" label="ACR 模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR 模块</figure-callout> **320** 和/或 <figure-callout id="325" label="CV 模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV 模块</figure-callout> **325** 可用于确定连续视频帧之间内容未发生变化。  

+   <para-num num="[0058]"></para-num>

    在某些实施例中，每个接收到的帧由<figure-callout id="320" label="ACR module" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> **320**生成指纹和/或水印，并可以进行比较。如果生成的指纹和/或水印不同，则一个或多个视频帧正在更改，即使检测到静音音频信号也不太可能发生暂停事件。然而，如果指纹和/或水印相同（例如，来自连续3-4帧的指纹和/或水印未更改），并且检测到静音音频信号，则显示<figure-callout id="108" label="device" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">设备</figure-callout> **108**确定检测到了暂停事件。换句话说，<figure-callout id="132" label="user" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132**可能在<figure-callout id="106" label="media device" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**处暂停了流数据，也许使用了<figure-callout id="110" label="remote control" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110**。可以通过<figure-callout id="340" label="module" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">模块</figure-callout> **340**的硬件处理、通过处理存储在<figure-callout id="380" label="memory" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">内存</figure-callout> **380**中的软件执行，或二者结合来执行静音音频信号的检测和/或不同视频帧的指纹和/或水印的比较。

+   <para-num num="[0059]"></para-num>

    在某些实施例中，元数据可以由<figure-callout id="325" label="CV模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV模块</figure-callout> **325**为每个接收的帧生成，并且生成的元数据可以进行比较。如果生成的元数据不同，则一个或多个视频帧正在变化，并且即使检测到静音音频信号，暂停事件也不太可能发生。但是，如果每帧的元数据相同（例如，来自3-4连续帧的元数据未更改），并且检测到静音音频信号，则显示<figure-callout id="108" label="设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">设备</figure-callout> **108**确定检测到暂停事件。换句话说，<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132**可能在<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**处暂停了流数据，可能是通过使用<figure-callout id="110" label="遥控器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110**。可以通过处理<figure-callout id="340" label="模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">模块</figure-callout> **340**中的硬件执行静音音频信号的检测和/或比较来自不同视频帧的元数据，或者通过处理存储在<figure-callout id="380" label="存储器" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">存储器</figure-callout> **380**中的软件执行这些操作，或者两者兼而有之。

+   <para-num num="[0060]"></para-num>

    在一些实施例中，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可以根据对暂停图标的识别来检测暂停事件。例如，<figure-callout id="325" label="CV模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV模块</figure-callout> **325** 可以分析通过<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107** 从<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 接收的媒体内容（例如一个或多个视频帧），以生成标识视频帧中对象（例如汽车类型、演员、暂停图标、企鹅或花上的蜜蜂）的元数据。当通过<figure-callout id="310" label="HDMI模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">HDMI模块</figure-callout> **310** 接收到的一个或多个连续视频帧中检测到暂停图标时，则检测到暂停事件。通过处理<figure-callout id="340" label="模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">模块</figure-callout> **340** 中的硬件，通过执行存储在<figure-callout id="380" label="内存" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">内存</figure-callout> **380** 中的软件或其组合来执行一个或多个连续视频帧中的暂停图标的检测。

+   <para-num num="[0061]"></para-num>

    当 **108**<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> 确定检测到暂停事件（例如，基于遥控器通过功能，检测到静音音频与不变的视频帧，或者检测到暂停图标与视频帧中的静止），**108**<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> 可以对暂停事件的视频帧和/或音频帧执行内容识别（例如，暂停的电影场景）。在某些实施例中，**320**<figure-callout id="320" label="ACR模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> 可以为视频帧和/或音频帧生成指纹和/或水印。在某些实施例中，**325**<figure-callout id="325" label="CV模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV模块</figure-callout> 可以生成与视频帧中识别或识别出的对象对应的元数据。

+   <para-num num="[0062]"></para-num>

    <figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可以使用指纹、水印和/或元数据来确定一个或多个适当的广告（例如广告），将广告添加到图形平面，并在<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 上呈现广告的部分或持续时间内。例如，在暂停事件检测后（例如，<figure-callout id="330" label="广告插入模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330** 可能会从处理模块 **340** 收到暂停事件检测的通知），来自<figure-callout id="320" label="ACR模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> **320** 的相应指纹和/或水印，以及来自<figure-callout id="325" label="CV模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV模块</figure-callout> **325** 的相应元数据。 <figure-callout id="330" label="广告插入模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330** 可以使用相应的指纹、水印和/或元数据来选择一个或多个相关广告，以呈现在<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 上。指纹和/或水印可能对应于某部电影标题、知名演员和电影类型。元数据可能对应于香槟酒瓶和山景。相关广告可能包括知名演员、某种香槟酒、包含被认可山景的度假机会。一个或多个相关广告可以是静态的（例如静止图像）或动态的（例如gif、短视频、动画、交互式屏幕）。在某些实施例中，关于交互式屏幕（例如调查）的来自<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 的输入可以被收集并传输到众包<figure-callout id="128" label="服务器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">服务器</figure-callout> **128**。在某些实施例中，作为对一或多个相关广告输入的响应，额外的信息可以被传输到与显示设备 **108** 关联的客户账户对应的移动设备（例如电子邮件、短信）。

+   <para-num num="[0063]"></para-num>

    <figure-callout id="330" label="广告插入模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330** 可能会将一个或多个相关广告传输到 <figure-callout id="360" label="图形模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">图形模块</figure-callout> **360**，以添加到显示设备 <figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 的图形平面（未显示）。接收到的一个或多个视频帧可以经过视频缩放和 <figure-callout id="350" label="FRC" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">FRC</figure-callout> **350** 处理。一个或多个视频帧可以添加到显示设备 **108** 的视频平面（例如，在检测暂停事件时使用的最后一个视频帧可以添加到视频平面）。在某些实施例中，视频平面和图形平面可以混合（例如，合并在一起），并且混合平面的数据可以由 <figure-callout id="370" label="显示模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">显示模块</figure-callout> **370** 接收，用于在 <figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 上呈现。在某些实施例中，图形平面可能显示在视频平面的前面（例如，叠加以在视频平面顶部查看）。在某些实施例中，图形平面的图像和/或广告可以是透明的。在某些实施例中，来自 <figure-callout id="110" label="遥控器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 的选择功能可以通过 <figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 传递，以显示 <figure-callout id="108" label="设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">设备</figure-callout> **108** 从图形平面进行选择。上述功能可以由 <figure-callout id="340" label="处理模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">处理模块</figure-callout> **340** 执行，执行存储在 <figure-callout id="380" label="内存" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">内存</figure-callout> **380** 中的各模块软件，或者硬件和软件的组合。

+   <para-num num="[0064]"></para-num>

    <figref idrefs="DRAWINGS">图 **7**A</figref> 描述了根据某些实施例的<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 的块图。例如，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可能利用通过 HDMI 连接接收的控制信号信息以及视频帧，来确定在<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 上呈现适当和相关广告。在某些实施例中，相关广告在<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 的暂停事件期间呈现。

+   <para-num num="[0065]"></para-num>

    为了说明目的，但不限于，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可能参考公开的其他图中的元素进行描述。例如，HDMI 输入 **710** 可能支持<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107**，如<figref idrefs="DRAWINGS">图 **1**</figref>中的。其中<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107**连接到<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**，如<figref idrefs="DRAWINGS">图 **1**</figref>中的。 <figure-callout id="710" label="HDMI输入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">HDMI输入</figure-callout> **710**， <figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730**， <figure-callout id="740" label="计算机视觉" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">计算机视觉</figure-callout> **740**， <figure-callout id="750" label="广告插入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">广告插入</figure-callout> **750**， <figure-callout id="770" label="图形模块" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">图形模块</figure-callout> **770**， <figure-callout id="780" label="内存" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">内存</figure-callout> **780**， 和视频缩放与<figure-callout id="782" label="FRC" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">FRC</figure-callout> **782**，以及<figure-callout id="790" label="显示面板" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790**可以分别对应于<figure-callout id="310" label="HDMI模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">HDMI模块</figure-callout> **310**， <figure-callout id="320" label="ACR模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR模块</figure-callout> **320**， <figure-callout id="325" label="CV模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV模块</figure-callout> **325**， <figure-callout id="330" label="广告插入模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330**， <figure-callout id="360" label="图形模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">图形模块</figure-callout> **360**， <figure-callout id="380" label="内存" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">内存</figure-callout> **380**，视频缩放和<figure-callout id="350" label="FRC" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">FRC</figure-callout> **350**，和<figure-callout id="370" label="显示模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">显示模块</figure-callout> **370**，如<figref idrefs="DRAWINGS">图 **3**</figref>中的。

+   <para-num num="[0066]"></para-num>

    在某些实施例中，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可能通过 HDMI 连接与媒体设备（例如，<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 在图 **6** 或 <figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 在图 **8** ）连接。在某些实施例中，<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 可以是游戏设备。<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可以通过 <figure-callout id="710" label="HDMI 输入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">HDMI 输入</figure-callout> **710** 从<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 接收视频帧以及一个或多个控制信号。控制信号可以包括来自<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 的产品信息信号、延迟模式信号或刷新率信号。<figure-callout id="722" label="产品信息功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">产品信息功能</figure-callout> **722** 可以是由<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 决定的有关源（例如<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600**）的 SPD 的函数。SPD 的示例包括游戏设备平台的类型和/或游戏设备平台的型号。例如，产品信息信号可以包括一个 SPD InfoFrame，该帧将源设备（如<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600**，例如视频游戏控制台）的名称和产品类型传输给接收设备，例如显示设备<figure-callout id="700" label="设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">设备</figure-callout> **700**（例如电视）。当视频游戏控制台连接到电视时，视频游戏控制台可以通过产品信息信号向显示设备<figure-callout id="700" label="设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">设备</figure-callout> **700** 传输以下 SPD。这些 SPD 可以包括以下内容：

+   +   来源信息=8（表示例如游戏）

    +   供应商名称=公司A

    +   产品描述=视频游戏主机模型1

+   <para-num num="[0070]"></para-num>

    使用上述SPD信息，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可以展示一个更新型号的<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 的广告，例如，比视频游戏主机模型1更新的型号。 <figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 也可以展示竞争对手产品的广告。

+   <para-num num="[0071]"></para-num>

    <figure-callout id="724" label="延迟模式功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">延迟模式功能</figure-callout> **724** 可以是决定来自<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 的延迟模式信号的一个功能。ALLM 可以指示<figure-callout id="618" label="游戏应用" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">游戏应用</figure-callout> **618** 是否正在<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 上运行（例如，执行）是一个低延迟应用。因此，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 利用资源支持<figure-callout id="618" label="游戏应用" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">游戏应用</figure-callout> **618** 的相应低延迟性能。例如，ALLM使得媒体设备**600**（例如，像视频游戏控制台这样的源）能够自动启用或禁用显示设备**700**（例如，像电视这样的接收器）的低延迟性能，而无需用户导航到显示设备**700**的菜单（例如，接收器的菜单）来设置与媒体设备**600**内容对应的最佳延迟。媒体设备**600**可以通过HDMI Forum Vendor Specific Info Frame（HF-VSIF）传输延迟模式信号。在HF-VSIF中的第5个字节可能包含一个ALLM_Mode字段，可以是0或1。如果ALLM_Mode=1，媒体设备**600**（例如，像视频游戏控制台这样的源）请求显示设备**700**（例如，像电视这样的接收器）切换到游戏模式。如果ALLM_Mode=0，媒体设备**600**请求显示设备**700**关闭游戏模式。因此，<figure-callout id="724" label="延迟模式功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">延迟模式功能</figure-callout> **724** 可以确定作为用户开始玩游戏的指示的ALLM值，因此可以通过<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 向用户显示与游戏相关的广告。

+   <para-num num="[0072]"></para-num>

    <figure-callout id="726" label="刷新率功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">刷新率功能</figure-callout> **726** 可以是由 <figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 决定的高级游戏应用的可变刷新率（VRR）。在某些实施例中，VRR 可以是动态显示刷新率，可以在飞行中连续和无缝变化（例如，在流媒体播放期间更改）。VRR 可以通过 <figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 使用刷新率信号中的视频定时扩展元数据包（EMP）进行传输。使用 VRR，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可以广告源和/或提供类似先进类型的VRR和/或更先进类型的VRR的应用程序（例如，由AMD显卡提供的Free Sync支持的VRR或由NVIDIA显卡提供的GSync）。在某些实施例中，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可以利用控制信号信息（例如来自 <figure-callout id="722" label="产品信息功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">产品信息功能</figure-callout> **722** 的SPD，来自 <figure-callout id="724" label="延迟模式功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">延迟模式功能</figure-callout> **724** 的ALLM，以及来自刷新率功能 **726** 的VRR）来确定在 <figure-callout id="790" label="显示面板" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790** 上呈现的相关广告。

+   <para-num num="[0073]"></para-num>

    在某些实施例中，**显示设备** 700 可通过以下方式检测暂停事件：i) 通过遥控器直通功能；ii) 无声音频信号；和/或 iii) 暂停图标识别。例如，**显示设备** 700 可根据从媒体设备通过 HDMI 连接（例如，HDMI 连接 **107**）接收到的暂停键信号（例如，遥控器直通功能）轻松检测到暂停事件，通过 **HDMI 输入** 710。硬件上可以由 **处理器** 720 进行暂停事件的检测，也可以由存储在 **内存** 780 中的软件由 **处理器** 720 执行，或者二者结合。例如，**处理器** 720 可包括一个或多个处理器。在某些实施例中，**处理器** 720 可执行存储在不同内存中的软件（未显示）。

+   <para-num num="[0074]"></para-num>

    在一些实施例中，**显示设备**<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可以基于从媒体设备（例如，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**，<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 或媒体设备 **800**）接收到的静音音频信号检测暂停事件，以及确定从媒体设备接收到的一个或多个视频帧未发生变化。例如，可以由<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730** 为每个接收到的帧生成指纹和/或水印，并可以对生成的指纹和/或水印进行比较。在一些实施例中，<figure-callout id="740" label="CV" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">CV</figure-callout> **740** 可以为每个接收到的视频帧生成元数据，并可以比较连续视频帧的生成元数据。当连续视频帧的指纹、水印和/或元数据在静音音频信号的作用下未发生变化时，就检测到暂停事件。基于静音音频信号和未变化的视频帧检测暂停事件可以由硬件<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720**、通过执行存储在<figure-callout id="780" label="存储器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">存储器</figure-callout> **780** 中的软件的<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 或两者的组合执行。例如，<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 可能包括一个或多个处理器。在一些实施例中，<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 可以执行存储在不同存储器中的软件（未显示）中的软件。

+   <para-num num="[0075]"></para-num>

    在某些实施例中，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 可以基于暂停图标识别检测暂停事件。例如，<figure-callout id="740" label="CV" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">CV</figure-callout> **740** 可以分析通过 HDMI 输入 **710** 接收的媒体内容（例如一个或多个视频帧），生成标识视频帧中对象（例如暂停图标）的元数据。当接收到的一个或多个连续视频帧的元数据包含暂停图标，并且确定视频帧未发生变化时，处理器可以确定检测到了暂停事件。检测到暂停图标和不变的视频帧可以由<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 在硬件中执行，或者由<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 执行存储在<figure-callout id="780" label="内存" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">内存</figure-callout> **780** 中的软件，或者二者的组合。例如，<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 可能包括一个或多个处理器。在某些实施例中，<figure-callout id="720" label="处理器" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720** 可能执行存储在不同存储器中的软件（未显示）。

+   <para-num num="[0076]"></para-num>

    当 **700** 确定检测到暂停事件（例如，基于遥控器透传功能、检测到静音音频以及不变的视频帧或者识别到暂停图标而视频帧无运动）后，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 可以对暂停事件的视频帧执行内容识别（例如，电影暂停时的场景），以确定在暂停事件的期间或持续时间内应在<figure-callout id="790" label="显示面板" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790** 上呈现的相关广告。例如，<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730** 可能为视频帧生成指纹和/或水印，以便确定包括视频帧在内的视频（例如电影）以及与视频相关的数据（例如演员、电影标题、流派）。在某些实施例中，<figure-callout id="740" label="计算机视觉" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">计算机视觉</figure-callout> **740** 可以生成对视频帧内识别或识别的对象（例如香槟瓶、山景）对应的元数据。生成的指纹、水印和/或元数据可用于确定在<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 上显示的相关广告的部分或整个暂停事件的持续时间。

+   <para-num num="[0077]"></para-num>

    在某些实施例中，在<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**确定检测到暂停事件（例如，基于遥控器直通功能、检测到无声音频与不变的视频帧以及识别暂停图标但视频帧没有运动），<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**可以对暂停事件的音频帧（例如电影暂停时的场景）进行内容识别，以确定在<figure-callout id="790" label="显示面板" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790**上显示相关广告的时间或持续时间。例如，<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730**可以为音频帧生成指纹和/或水印，从而确定包括音频帧的内容（例如电影），以及与音频帧相关的数据（例如演员、电影标题、流派）。由计算机视觉**740**生成的指纹、水印和/或元数据可用于确定在<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**上显示的相关广告的部分或整个暂停事件的持续时间。

+   <para-num num="[0078]"></para-num>

    在某些实施例中，在<figure-callout id="700" label="display device" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**确定检测到暂停事件之后（例如，基于远程控制通过功能，检测到静音音频与不变的视频帧，和/或暂停图标以及视频帧无运动），<figure-callout id="700" label="display device" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**可以利用从控制信号（例如，SPD、ALLM和/或VRR）获取的信息来确定在<figure-callout id="790" label="display panel" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790**在暂停事件的部分或持续时间内呈现一个或多个适当的广告。在某些实施例中，<figure-callout id="700" label="display device" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**与SPD、ALLM和/或VRR一起利用一个或多个视频帧的内容识别数据（例如来自<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730**的指纹和/或水印以及计算机视觉的元数据 **740**）来确定在<figure-callout id="790" label="display panel" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790**在暂停事件的部分或持续时间内呈现一个或多个适当的广告。

+   <para-num num="[0079]"></para-num>

    例如，在检测到暂停事件之后（例如，<figure-callout id="750" 

    label="ad insertion" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">广告插入</figure-callout> **750**可能从处理器 **720**）接收到关于暂停事件检测的通知，来自<figure-callout id="730" 

    label="ACR" 

    filenames="US20230388589A1-20231130-D00007.png"

    state="{{state}}">产品信息功能</figure-callout> **730**，对应的元数据来自<figure-callout id="740" 

    label="CV" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">刷新率功能</figure-callout> **740**，SPD from <figure-callout id="722" 

    label="product information function" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">产品信息功能</figure-callout> **722**的相应指纹和/或水印，来自<figure-callout id="724" 

    label="latency mode function" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">延迟模式功能</figure-callout> **724**，和/或来自<figure-callout id="726" 

    label="refresh rate function" 

    filenames="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">刷新率功能</figure-callout>` **726** 可被<figure-callout id="750" 

    标签="广告插入" 

    文件名="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    `state="{{state}}">广告插入</figure-callout>` **750** 选择在<figure-callout id="700" 

    标签="显示设备" 

    文件名="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">显示设备</figure-callout>` **700**。

    例如，指纹和/或水印可能对应于<figure-callout id="618" 

    标签="某些游戏应用程序" 

    文件名="US20230388589A1-20231130-D00006.png" 

    `state="{{state}}">某些游戏应用程序</figure-callout>` **618**。

    元数据可能对应于游戏应用程序的类型**618**（例如，魔法生物、体育场、车辆）。

    SPD可能指示特定游戏系统或游戏主机以及型号，ALLM可能指示需要低延迟以及某种VRR用于高级游戏。

    SPD可以包括与特定硬件设备和/或硬件设备型号对应的唯一ID。

    在某些实施例中，<figure-callout id="750" 

    标签="广告插入" 

    文件名="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    `state="{{state}}">广告插入</figure-callout>` **750** 确定一个或多个相关广告。

    相关广告可能包括新型游戏系统、竞争对手游戏系统和/或利用相应ALLM和/或相应VRR的一个或多个相似类型游戏应用程序。

    可能存在许多变体。

    在某些实施例中，<figure-callout id="750" 

    标签="广告插入" 

    文件名="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    `state="{{state}}">广告插入</figure-callout>` **750** 传输来自<figure-callout id="730" 

    标签="ACR" 

    文件名="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">ACR</figure-callout>` **730**，来自<figure-callout id="740" 

    标签="CV" 

    文件名="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">CV</figure-callout>` **740**，SPD来自<figure-callout id="722" 

    标签="产品信息功能" 

    文件名="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">产品信息功能</figure-callout>` **722**，从<figure-callout id="724" 

    标签="延迟模式功能" 

    文件名="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">延迟模式功能</figure-callout>` **724**，和/或从<figure-callout id="726" 

    标签="刷新率功能" 

    文件名="US20230388589A1-20231130-D00007.png" 

    `state="{{state}}">刷新率功能</figure-callout>` **726** 用于网络**760**（例如，云网络），以确定相关广告并接收来自<figure-callout id="760" 

    标签="网络" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">网络</figure-callout> **760**.

+   <para-num num="[0080]"></para-num>

    一个或多个相关广告可以是静态的（例如，静止图像），动态的（例如，gif、短视频、动画、交互式屏幕）或用于购买一个或多个相关广告中物品的链接。在某些实施例中，来自<figure-callout id="132" label="user" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 对于交互式屏幕（例如调查）的输入可以被收集并传输到众包<figure-callout id="128" label="server" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">服务器</figure-callout> **128**。在某些实施例中，作为对一个或多个相关广告的输入的响应，额外信息可以被传输到与显示设备**700**关联的客户帐户对应的移动设备（例如电子邮件、短信）。在某些实施例中，来自<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730**的相应指纹和/或水印，来自<figure-callout id="740" label="CV" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">CV</figure-callout> **740**的相应元数据，来自<figure-callout id="722" label="product information function" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">产品信息功能</figure-callout> **722**的SPD，来自<figure-callout id="724" label="latency mode function" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">延迟模式功能</figure-callout> **724**的ALLM，来自<figure-callout id="726" label="refresh rate function" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">刷新率功能</figure-callout> **726**的VRR以及/或者一个或多个相关广告可以被存储在与<figure-callout id="132" label="user" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132**和/或<figure-callout id="600" label="media device" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600**对应的历史资料相关的历史配置文件中。<figure-callout id="750" label="广告插入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">广告插入</figure-callout> **750**和/或<figure-callout id="760" label="网络" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">网络</figure-callout> **760**可以利用历史配置文件与当前数据（例如来自<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730**的指纹和/或水印，来自<figure-callout id="740" label="CV" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">CV</figure-callout> **740**的元数据，来自<figure-callout id="722" label="product information function" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">产品信息功能</figure-callout> **722**的SPD，来自<figure-callout id="724" label="latency mode function" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">延迟模式功能</figure-callout> **724**的ALLM以及来自<figure-callout id="726" label="refresh rate function" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">刷新率功能</figure-callout> **726**的VRR）来确定当前的相关广告。

+   <para-num num="[0081]"></para-num>

    <figure-callout id="750"

    label="广告插入"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">广告插入</figure-callout> **750**可能将一个或多个相关广告传输到<figure-callout id="770"

    label="图形模块"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">图形模块</figure-callout> **770**以添加到<figure-callout id="775"

    label="图形平面"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">图形平面</figure-callout> **775**的<figure-callout id="700"

    label="显示设备"

    filenames="US20230388589A1-20231130-D00007.png"

    state="{{state}}">显示设备</figure-callout> **700**。

    接收到的一个或多个视频帧可以存储在<figure-callout id="780"

    label="存储器"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">memory</figure-callout> **780**，并通过视频缩放和<figure-callout id="782"

    label="FRC"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">FRC</figure-callout> **782**。

    处理过的一个或多个视频帧可以添加到<figure-callout id="784"

    label="视频平面"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">视频平面</figure-callout> **784**的显示设备**700**上呈现（例如，在检测到暂停事件时，最后一个视频帧可添加到视频平面**784**）。

    在某些实施例中，<figure-callout id="784"

    label="视频平面"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">视频平面</figure-callout> **784**和<figure-callout id="775"

    label="图形平面"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">图形平面</figure-callout> **775**可以混合（例如合并）在一起，混合平面的数据可以由<figure-callout id="790"

    label="显示面板"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">显示面板</figure-callout> **790**用于在<figure-callout id="700"

    label="显示设备"

    filenames="US20230388589A1-20231130-D00007.png"

    state="{{state}}">显示设备</figure-callout> **700**。

    在一些<figure-callout id="775"

    label="实施例图形平面"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png"

    state="{{state}}">实施例图形平面</figure-callout> **775**可以显示在视频平面**784**的前面（例如叠加在视频平面**784**的顶部查看）。

    在某些实施例中，图像和/或<figure-callout id="775"

    label="图形平面"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">图形平面</figure-callout> **775** 的选择功能可以是透明的。 

    在某些实施例中，从<figure-callout id="110" 

    label="遥控器" 

    filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" 

    state="{{state}}">遥控器</figure-callout> **110** 可以通过<figure-callout id="600" 

    label="媒体设备" 

    filenames="US20230388589A1-20231130-D00006.png" 

    state="{{state}}">媒体设备</figure-callout> **600** 执行，以显示<figure-callout id="700" 

    label="设备" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">设备</figure-callout> **700** 从图形平面中进行选择。 

    上述功能可以由<figure-callout id="720" 

    label="处理器" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">处理器</figure-callout> **720**，执行可能存储在<figure-callout id="780" 

    label="内存" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">内存</figure-callout> **780**，或硬件和软件的组合。

+   <para-num num="[0082]"></para-num>

    <figref idrefs="DRAWINGS">图 **7**B</figref> 描述了根据某些实施例的<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705** 的块图。仅为说明目的，而非限制，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705** 的元素可能通过引用披露中其他图示中的元素来描述。例如，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705** 的功能可能类似于<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 的<figref idrefs="DRAWINGS">图 **7**A</figref>，但网络 **765** 包括以下功能：<figure-callout id="722" label="产品信息功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">产品信息功能</figure-callout> **722**，<figure-callout id="724" label="延迟模式功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">延迟模式功能</figure-callout> **724**，<figure-callout id="726" label="刷新率功能" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">刷新率功能</figure-callout> **726**，<figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730** 和/或<figure-callout id="740" label="计算机视觉" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">计算机视觉</figure-callout> **740**，确定要在<figure-callout id="790" label="显示面板" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790** 上显示的相关广告，并将相关广告传输到<figure-callout id="750" label="广告插入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">广告插入</figure-callout> **750**。

+   <para-num num="[0083]"></para-num>

    <figref idrefs="DRAWINGS">图

    **9**</figref> 描述了<figure-callout id="900" 

    label="显示设备"

    filenames="US20230388589A1-20231130-D00010.png"

    state="{{state}}">显示设备</figure-callout> **900**，根据某些实施例。

    仅为说明目的，而非限制，<figure-callout id="900" 

    label="显示设备"

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">显示设备</figure-callout> **900** 可能通过引用披露中其他图示中的元素来描述。

    例如，<figure-callout id="900" 

    label="显示设备"

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">显示设备</figure-callout> **900** 可能与 <figure-callout id="700" 

    label="显示设备" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">显示设备</figure-callout> **700** of <figref idrefs="DRAWINGS">FIG. 

    **7**A</figref> 相似。 

    例如，<figure-callout id="910" 

    label="HDMI 输入" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">HDMI 输入</figure-callout> **910**, <figure-callout id="922" 

    label="产品信息功能" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">产品信息功能</figure-callout> **922**, <figure-callout id="924" 

    label="延迟模式功能" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">延迟模式功能</figure-callout> **924**, <figure-callout id="926" 

    label="刷新率功能" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">刷新率功能</figure-callout> **926**, <figure-callout id="930" 

    label="ACR" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">ACR</figure-callout> **930**, <figure-callout id="940" 

    label="计算机视觉"

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">计算机视觉</figure-callout> **940**, <figure-callout id="920" 

    label="处理器" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">处理器</figure-callout> **920**, <figure-callout id="950" 

    label="广告插入" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">广告插入</figure-callout> **950**, <figure-callout id="970" 

    label="图形模块" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">图形模块</figure-callout> **970**, <figure-callout id="975" 

    label="图形平面" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">图形平面</figure-callout> **975**，<figure-callout id="984" 

    label="视频平面" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">视频平面</figure-callout> **984**, <figure-callout id="980" 

    label="内存" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">内存</figure-callout> **980** 和视频缩放以及 <figure-callout id="982" 

    label="FRC" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">FRC</figure-callout> **982**，以及 <figure-callout id="990" 

    label="显示面板" 

    filenames="US20230388589A1-20231130-D00010.png" 

    state="{{state}}">显示面板</figure-callout> **990** 可以分别对应 HDMI 输入 **710**, <figure-callout id="722" 

    label="产品信息功能" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">产品信息功能</figure-callout> **722**, <figure-callout id="724" 

    label="延迟模式功能" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">延迟模式功能</figure-callout> **724**, <figure-callout id="726" 

    label="刷新率功能" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">刷新率功能</figure-callout> **726**, <figure-callout id="730" 

    label="ACR" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">ACR</figure-callout> **730**, <figure-callout id="740" 

    label="计算机视觉" 

    filenames="US20230388589A1-20231130-D00007.png" 

    state="{{state}}">计算机视觉</figure-callout> **740**, <figure-callout id="720" 

    label="处理器" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">处理器</figure-callout> **720**, <figure-callout id="750" 

    label="广告插入"

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">广告插入</figure-callout> **750**, <figure-callout id="770" 

    label="图形模块" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">图形模块</figure-callout> **770**, <figure-callout id="775" 

    label="图形平面" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">图形平面</figure-callout> **775**, <figure-callout id="784" 

    label="视频平面" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">视频平面</figure-callout> **784**, <figure-callout id="780" 

    label="memory" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">内存</figure-callout> **780**, 和视频缩放以及 <figure-callout id="782" 

    label="FRC" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">FRC</figure-callout> **782**, 和 <figure-callout id="790" 

    label="显示面板" 

    filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" 

    state="{{state}}">显示面板</figure-callout> **790** of <figref idrefs="DRAWINGS">FIG. 

    **7**A</figref>.

+   <para-num num="[0084]"></para-num>

    在某些实施例中，<figure-callout id="900" label="显示设备" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">显示设备</figure-callout> **900** 可以通过不同于<figure-callout id="920" label="处理器" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">处理器</figure-callout> **920** 的相应处理器（未显示）执行一个或多个功能<figure-callout id="922" label="产品信息功能" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">产品信息功能</figure-callout> **922**、<figure-callout id="924" label="延迟模式功能" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">延迟模式功能</figure-callout> **924**、<figure-callout id="926" label="刷新率功能" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">刷新率功能</figure-callout> **926**、<figure-callout id="930" label="ACR" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">ACR</figure-callout> **930** 和/或<figure-callout id="940" label="计算机视觉" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">计算机视觉</figure-callout> **940**。在某些实施例中，一个或多个功能<figure-callout id="922" label="产品信息功能" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">产品信息功能</figure-callout> **922**、<figure-callout id="924" label="延迟模式功能" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">延迟模式功能</figure-callout> **924**、<figure-callout id="926" label="刷新率功能" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">刷新率功能</figure-callout> **926**、<figure-callout id="930" label="ACR" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">ACR</figure-callout> **930** 和/或<figure-callout id="940" label="计算机视觉" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">计算机视觉</figure-callout> **940** 可能由<figure-callout id="960" label="网络" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">网络</figure-callout> **960**、<figure-callout id="920" label="处理器" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">处理器</figure-callout> **920** 和/或不同的处理器（未显示）执行。

+   <para-num num="[0085]"></para-num>

    在某些实施例中，<figure-callout id="910" label="HDMI输入" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">HDMI输入</figure-callout> **910** 从<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 和/或 <figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 接收视频帧和控制信号。在某些实施例中，<figure-callout id="960" label="网络" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">网络</figure-callout> **960** 将视频帧提供给HDMI输入 **910**。

+   <para-num num="[0086]"></para-num>

    <figref idrefs="DRAWINGS">图 **4**</figref> 描述了根据某些实施例的显示设备的方法 **400**。仅供说明目的，不限于此，方法 **400** 可能参考披露的其他图中的元素进行描述。例如，方法 **400** 可由显示设备（例如，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 图 **1** 和图 **3** 的<figref idrefs="DRAWINGS">图 **7**A</figref>的<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705**，以及 <figure-callout id="900" label="显示设备" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">显示设备</figure-callout> **900** 的<figref idrefs="DRAWINGS">图 **9**</figref> ）执行。

+   <para-num num="[0087]"></para-num>

    在 **405**，显示设备可以确定CEC（例如，遥控器传递）功能是否已启用。当CEC功能启用时（例如，已设置），方法 **400** 进入 **410** 步骤。否则，方法 **400** 进入 **435** 步骤。

+   <para-num num="[0088]"></para-num>

    在 **410**，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 确定是否接收到暂停键信号。例如，<figure-callout id="132" label="用户" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">用户</figure-callout> **132** 可能在 <figure-callout id="110" label="遥控器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110** 上选择了暂停键。由于启用了CEC功能，暂停键信号经过媒体设备（例如，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106** 在<figref idrefs="DRAWINGS">图 **1**</figref>或<figref idrefs="DRAWINGS">图 **2**</figref>，<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600** 在<figref idrefs="DRAWINGS">图 **6**</figref>，和/或<figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800** 在<figref idrefs="DRAWINGS">图 **8**</figref>）到显示设备（例如，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 在<figref idrefs="DRAWINGS">图 **1**</figref>和<figref idrefs="DRAWINGS">图 **3**</figref>，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700** 在<figref idrefs="DRAWINGS">图 **7**A</figref>，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705** 在<figref idrefs="DRAWINGS">图 **7**B</figref>，或<figure-callout id="900" label="显示设备" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">显示设备</figure-callout> **900** 在<figref idrefs="DRAWINGS">图 **9**</figref>）通过HDMI连接（例如，<figure-callout id="107" label="HDMI连接" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">HDMI连接</figure-callout> **107** 在<figref idrefs="DRAWINGS">图 **1**</figref>）或<figure-callout id="710" label="HDMI输入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">HDMI输入</figure-callout> **710**，或<figure-callout id="910" label="HDMI输入" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">HDMI输入</figure-callout> **910**。当接收到暂停键信号时，显示设备确定检测到了暂停事件，方法 **400** 继续执行 **415** 和 **418**。否则，方法 **400** 返回到 **405**。

+   <para-num num="[0089]"></para-num>

    在 **415** 处，当检测到暂停事件时，显示设备对从媒体设备 **106** 接收到的一个或多个视频帧（例如，确定暂停事件的一个或多个视频帧）执行内容识别。在某些实施例中，当检测到暂停事件时，显示设备对从媒体设备 **106** 接收到的一个或多个音频帧（例如，确定暂停事件的一个或多个音频帧）执行内容识别。内容识别可以由一个 ACR 功能（例如，<figure-callout id="320" label="ACR 模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">ACR 模块</figure-callout> **320**, <figure-callout id="730" label="ACR" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">ACR</figure-callout> **730**, <figure-callout id="760" label="网络" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">网络</figure-callout> **760**, <figure-callout id="930" label="ACR" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">ACR</figure-callout> **930**，或网络 **960**）来确定相应的指纹和/或水印，或者由一个 CV 功能（例如，<figure-callout id="325" label="CV 模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">CV 模块</figure-callout> **325**, <figure-callout id="740" label="计算机视觉" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">计算机视觉</figure-callout> **740**, <figure-callout id="760" label="网络" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">网络</figure-callout> **760**, <figure-callout id="940" label="计算机视觉" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">计算机视觉</figure-callout> **940**，和/或网络 **960**）来确定相应的元数据。

+   <para-num num="[0090]"></para-num>

    在**418**，当检测到暂停事件时，显示设备根据接收到的控制信号执行源设备识别和/或应用程序识别，包括但不限于产品信息信号、延迟模式信号或刷新率信号。例如，产品信息功能使显示设备能够确定SPD，其中包括游戏设备或游戏平台的类型（例如，视频游戏机）和型号。延迟模式功能使显示设备能够确定ALLM，即正在使用的应用程序是否最适合低延迟（例如，<figure-callout id="618" label="游戏应用程序" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">游戏应用程序</figure-callout> **618**，<figure-callout id="816" label="输入应用程序" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">输入应用程序</figure-callout> **816**或媒体应用程序**817**）。刷新率功能使显示设备能够确定VRR，并且正在使用的应用程序可能是一个高级（例如，复杂的）游戏应用程序。

+   <para-num num="[0091]"></para-num>

    在**420**处，显示设备根据确定的指纹、水印、元数据和/或来自控制信号（例如，SPD、ALLM、VRR）的信息确定相应的广告。例如，广告插入功能（例如，<figure-callout id="330" label="广告插入模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330**，<figure-callout id="750" label="广告插入" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">广告插入</figure-callout> **750**，或者广告插入 **950**）可以确定与指纹、水印、元数据和/或来自控制信号（例如，SPD、ALLM、VRR）的信息相对应的适当广告。在某些实施例中，广告插入功能与网络通信（例如，通过<figure-callout id="114" label="通信设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">通信设备</figure-callout> **114**，<figure-callout id="760" label="网络" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">网络</figure-callout> **760**或者网络 **960**）以交换指纹、水印、元数据和/或来自控制信号（例如，SPD、ALLM、VRR）的信息和/或接收一个或多个相关广告以在显示设备（例如，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705**或者显示设备 **900**）期间展示。一个或多个相关广告可能包括静态或动态图像。

+   <para-num num="[0092]"></para-num>

    此篇文档包含了对视频帧和/或控制信号通过HDMI连接接收并传送到网络（例如<figure-callout id="118" label="network" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">网络</figure-callout> **118**, <figure-callout id="760" label="network" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">网络</figure-callout> **760**, 和/或网络 **960**）进行处理的某些实现方式。在某些实现中，网络执行一组活性识别(ACR)功能、色彩矢量化(CV)功能，以及控制信号分析，以此来识别从控制信号（如SPD、ALLM、VRR）中提取出的特征、水印、元数据和/或信息。此外，网络可以确定相应的广告（如SPD、ALLM、VRR）对应的广告。接着，网络向广告插入功能（如<figure-callout id="330" label="ad insertion module" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">广告插入模块</figure-callout> **330**, <figure-callout id="750" label="ad insertion" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">广告插入</figure-callout> **750**, 或广告插入 **950**）传输相对应的广告。

+   <para-num num="[0093]"></para-num>

    在某些实现中，网络负责执行活性识别(ACR)功能、色彩矢量化(CV)功能，以及控制信号分析，以此来识别从控制信号（如SPD、ALLM、VRR）中提取出的特征、水印、元数据和/或信息。显示设备上的处理器（如<figure-callout id="720" label="processor" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720**, <figure-callout id="920" label="processor" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">处理器</figure-callout> **920**，或显示设备上的其他处理器（未被显示））执行了网络未执行的其余功能，并将输出传输给网络（直接传输或通过广告插入功能进行传输）。网络依托从控制信号（如SPD、ALLM和/或VRR）中获得的特征、水印、元数据或相关信息，确定出适当的广告，并将这些广告传输至广告插入功能。

+   <para-num num="[0094]"></para-num>

    在某些实施例中，处理器（例如，<figure-callout id="720" label="processor" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">处理器</figure-callout> **720**，<figure-callout id="920" label="processor" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">处理器</figure-callout> **920**，或其他未显示的处理器）执行网络未执行的剩余功能，并将输出传输到广告插入功能。广告插入功能还接收来自控制信号（例如，SPD、ALLM和/或VRR）确定的一个或多个指纹、水印、元数据和/或信息。广告插入功能使用来自处理器和网络的接收数据确定一个或多个合适的广告。

+   <para-num num="[0095]"></para-num>

    在暂停事件期间，广告插入功能将一个或多个相关广告通信到显示设备上显示（例如，<figure-callout id="108" label="display device" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**，<figure-callout id="700" label="display device" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**，<figure-callout id="705" label="display device" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705**，或显示设备 **900**）。一个或多个相关广告可以包括静态或动态图像。

+   <para-num num="[0096]"></para-num>

    在 **425**，显示设备可以将一个或多个合适的广告添加到显示设备的图形平面中。例如，广告插入功能可以将一个或多个相关广告传输给图形模块（例如，<figure-callout id="360" label="graphics module" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">图形模块</figure-callout> **360**，<figure-callout id="770" label="graphics module" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">图形模块</figure-callout> **770**，图形模块 **970**），以添加到图形平面（例如，<figure-callout id="775" label="graphics plane" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">图形平面</figure-callout> **775**，图形平面 **975**）。

+   <para-num num="[0097]"></para-num>

    在**430**，显示设备在暂停事件期间呈现一个或多个适当的广告。例如，图形平面和视频平面可以混合（例如，对齐），并且显示模块（例如，<figure-callout id="370" label="显示模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">显示模块</figure-callout> **370**，<figure-callout id="790" label="显示面板" filenames="US20230388589A1-20231130-D00007.png,US20230388589A1-20231130-D00008.png" state="{{state}}">显示面板</figure-callout> **790**，显示面板 **990**）可以在显示设备的屏幕上呈现一个或多个适当的广告。在某些实施例中，显示设备确定暂停事件结束，并且停止在屏幕上呈现一个或多个适当的广告。确定暂停事件结束可以包括但不限于：从<figure-callout id="110" label="遥控器" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">遥控器</figure-callout> **110**接收播放键输入（例如，当CEC设置时）；检测到未收到静音音频或视频帧正在改变；和/或确定不再检测到暂停图标或视频帧中检测到运动（例如，通过CV功能）。

+   <para-num num="[0098]"></para-num>

    返回到**435**，当CEC功能未设置时，显示设备可以确定是否检测到静音音频信号。当检测到静音音频信号时，方法**400**继续到**440**。否则，方法**400**继续到**455**。

+   <para-num num="[0099]"></para-num>

    在**440**，当检测到静音音频时，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**确定一个或多个视频帧是否不变（例如，停止）。例如，<figure-callout id="340" label="处理模块" filenames="US20230388589A1-20231130-D00003.png" state="{{state}}">处理模块</figure-callout> **340**可以对接收到的视频帧执行定期采样。在某些示例中，改变的确定可以通过定期执行内容识别功能来完成。当确定视频帧未改变时，方法**400**继续到**415**。当视频帧正在改变时，方法**400**返回到**405**以检测暂停事件。

+   <para-num num="[0100]"></para-num>

    在**455**，显示设备确定是否在接收到的一个或多个视频帧中检测到暂停图标。例如，CV功能可以分析一个或多个视频帧的内容以识别暂停图标。当检测到暂停图标时，方法**400**继续到**460**。否则，方法**400**返回到**405**。

+   <para-num num="[0101]"></para-num>

    在 **460**，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108** 确定接收到的一个或多个音频和/或视频帧是否发生变化。例如，CV功能可以确定接收到的一个或多个视频帧的内容和/或上下文是否发生变化。在某些实施例中，ACR功能基于视频和/或音频帧确定接收到的视频和/或音频帧的内容是否发生变化。显示设备可以使用相应的指纹、水印和/或元数据来确定内容是否发生变化。在某些示例中，通过内容识别功能定期进行变化的确定。当未检测到音频和/或视频帧的变化时，方法 **400** 继续到 **415** 和/或 **418**。否则，方法 **400** 返回到 **405**。

<heading id="h-0010">示例计算机系统</heading>

+   <para-num num="[0102]"></para-num>

    各种实施例可以实施，例如，使用一个或多个众所周知的计算机系统，例如 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 如 <figref idrefs="DRAWINGS">图 **5**</figref> 所示。例如，<figure-callout id="106" label="媒体设备" filenames="US20230388589A1-20231130-D00001.png,US20230388589A1-20231130-D00002.png" state="{{state}}">媒体设备</figure-callout> **106**，<figure-callout id="600" label="媒体设备" filenames="US20230388589A1-20231130-D00006.png" state="{{state}}">媒体设备</figure-callout> **600**，<figure-callout id="800" label="媒体设备" filenames="US20230388589A1-20231130-D00009.png" state="{{state}}">媒体设备</figure-callout> **800**，<figure-callout id="108" label="显示设备" filenames="US20230388589A1-20231130-D00000.png,US20230388589A1-20231130-D00001.png" state="{{state}}">显示设备</figure-callout> **108**，<figure-callout id="700" label="显示设备" filenames="US20230388589A1-20231130-D00007.png" state="{{state}}">显示设备</figure-callout> **700**，<figure-callout id="705" label="显示设备" filenames="US20230388589A1-20231130-D00008.png" state="{{state}}">显示设备</figure-callout> **705**，或者 <figure-callout id="900" label="显示设备" filenames="US20230388589A1-20231130-D00010.png" state="{{state}}">显示设备</figure-callout> **900**，以及/或者方法 **400** 可以使用 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 的组合或子组合实施。另外或者，可以使用一个或多个计算机系统，例如 <figure-callout id="500" label="一个或多个计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">一个或多个计算机系统</figure-callout> **500**，例如，用于实施所述任何讨论在此的实施例及其组合和子组合。

+   <para-num num="[0103]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 可能包括一个或多个处理器（也称为中央处理单元或CPU），如 <figure-callout id="504" label="处理器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">处理器</figure-callout> **504**。 <figure-callout id="504" label="处理器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">处理器</figure-callout> **504** 可以连接到通信基础设施 **506**（可以是总线）。

+   <para-num num="[0104]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 也可能包括用户输入/输出设备 **503**，如显示器、键盘、指针设备等，这些设备可以通过用户输入/输出接口 **502** 与<figure-callout id="506" label="通信基础设施" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">通信基础设施</figure-callout> **506** 进行通信。

+   <para-num num="[0105]"></para-num>

    其中一个或多个<figure-callout id="504" label="处理器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">处理器</figure-callout> **504** 可能是图形处理单元（GPU）。在一个实施例中，GPU 可能是专门设计用于处理数学密集型应用的处理器。GPU 可能具有并行结构，适合处理大块数据的并行处理，例如计算机图形应用、图像、视频等中常见的数学密集数据。

+   <para-num num="[0106]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 也可能包括主存储器或<figure-callout id="508" label="主内存" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">主内存</figure-callout> **508**，如随机存取存储器（RAM）。<figure-callout id="508" label="主存储器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">主存储器</figure-callout> **508** 可能包括一个或多个级别的缓存。<figure-callout id="508" label="主存储器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">主存储器</figure-callout> **508** 可能存储有控制逻辑（即计算机软件）和/或数据。

+   <para-num num="[0107]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 还可能包括一个或多个次要存储设备或 <figure-callout id="510" label="内存" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">内存</figure-callout> **510**。 <figure-callout id="510" label="次要内存" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">次要内存</figure-callout> **510** 可包括例如 <figure-callout id="512" label="硬盘驱动器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">硬盘驱动器</figure-callout> **512** 和/或一个可移动存储设备或驱动器 **514**。 <figure-callout id="514" label="可移动存储驱动器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储驱动器</figure-callout> **514** 可以是软盘驱动器、磁带驱动器、光盘驱动器、光学存储设备、磁带备份设备和/或任何其他存储设备/驱动器。

+   <para-num num="[0108]"></para-num>

    <figure-callout id="514" label="可移动存储驱动器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储驱动器</figure-callout> **514** 可能会与 <figure-callout id="518" label="可移动存储单元" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储单元</figure-callout> **518** 进行交互。 <figure-callout id="518" label="可移动存储单元" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储单元</figure-callout> **518** 可包括存储有计算机软件（控制逻辑）和/或数据的计算机可用或可读存储设备。 <figure-callout id="518" label="可移动存储单元" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储单元</figure-callout> **518** 可以是软盘、磁带、光盘、DVD、光学存储盘和/或任何其他计算机数据存储设备。 <figure-callout id="514" label="可移动存储驱动器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储驱动器</figure-callout> **514** 可以从 <figure-callout id="518" label="可移动存储单元" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储单元</figure-callout> **518** 中读取和/或写入数据。

+   <para-num num="[0109]"></para-num>

    <figure-callout id="510" label="二级存储" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">二级存储</figure-callout> **510** 可能包括其他方法、设备、组件、工具或其他途径，用于允许计算机程序和/或其他指令和/或数据被 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 访问。这些方法、设备、组件、工具或其他途径可能包括可移动存储单元 **522** 和一个 <figure-callout id="520" label="接口" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">接口</figure-callout> **520**。可移动存储单元 **522** 和 <figure-callout id="520" label="接口" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">接口</figure-callout> **520** 的示例包括程序插卡和插卡接口（例如视频游戏设备中的那种）、可移动存储芯片（如EPROM或PROM）及其相关插座、存储卡和USB或其他端口、存储卡及其相关存储卡插槽，和/或任何其他可移动存储单元及其相关接口。

+   <para-num num="[0110]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 还可以包括通信或 <figure-callout id="524" label="网络接口" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">网络接口</figure-callout> **524**。 <figure-callout id="524" label="通信接口" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">通信接口</figure-callout> **524** 可以使 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 能够与任意组合的外部设备、外部网络、外部实体等（单独和集体以参考编号 **528** 引用）进行通信和交互。例如，<figure-callout id="524" label="通信接口" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">通信接口</figure-callout> **524** 可以允许 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 通过可能是有线和/或无线（或两者的组合）的 <figure-callout id="526" label="通信路径" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">通信路径</figure-callout> **526** 与外部或 <figure-callout id="528" label="远程设备" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">远程设备</figure-callout> **528** 进行通信。控制逻辑和/或数据可以通过 <figure-callout id="526" label="通信路径" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">通信路径</figure-callout> **526** 在 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 之间传输。

+   <para-num num="[0111]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 也可以是个人数字助理（PDA）、台式工作站、笔记本电脑或便携电脑、上网本、平板电脑、智能手机、智能手表或其他可穿戴设备、家电、物联网的一部分和/或嵌入式系统等，仅举几个非限制性的例子，或以上述任意组合为例。

+   <para-num num="[0112]"></para-num>

    <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 可能是客户端或服务器，通过任何交付范式访问或托管任何应用程序和/或数据，包括但不限于远程或分布式云计算解决方案；本地或本地软件（“本地”云解决方案）；“作为服务”模型（例如，内容作为服务（CaaS），数字内容作为服务（DCaaS），软件作为服务（SaaS），托管软件作为服务（MSaaS），平台作为服务（PaaS），桌面作为服务（DaaS），框架作为服务（FaaS），后端作为服务（BaaS），移动后端作为服务（MBaaS），基础设施作为服务（IaaS）等）；和/或混合模型，包括前述示例或其他服务或交付范式的任意组合。

+   <para-num num="[0113]"></para-num>

    在 <figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500** 中的任何适用数据结构、文件格式和模式可能源自标准，包括但不限于JavaScript对象表示法（JSON），可扩展标记语言（XML），另一种标记语言（YAML），可扩展超文本标记语言（XHTML），无线标记语言（WML），MessagePack，XML用户界面语言（XUL），或任何其他功能类似的表述，可以单独使用或组合使用。另外，也可以使用专有的数据结构、格式或模式，或者与已知或开放的标准组合使用。

+   <para-num num="[0114]"></para-num>

    在某些实施方式中，包括但不限于，具有存储在其中的控制逻辑（软件）的切实、非暂时的计算机可用或可读介质的实体或制造物也可以在本文中被称为计算机程序产品或程序存储设备。这包括<figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500**，<figure-callout id="508" label="主存储器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">主存储器</figure-callout> **508**，<figure-callout id="510" label="辅助存储器" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">辅助存储器</figure-callout> **510**，以及<figure-callout id="518" label="可移动存储单元" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">可移动存储单元</figure-callout> **518** 和 **522**，以及体现上述任何组合的切实制造物。这样的控制逻辑，当由一个或多个数据处理设备（例如<figure-callout id="500" label="计算机系统" filenames="US20230388589A1-20231130-D00005.png" state="{{state}}">计算机系统</figure-callout> **500**或处理器 **504**）执行时，可能导致这些数据处理设备按照本文描述的方式运行。

+   <para-num num="[0115]"></para-num>

    根据本公开内容所包含的教导，熟悉相关技术的人士可以明显地了解如何利用数据处理设备、计算机系统和/或与<figref idrefs="DRAWINGS">图**5**</figref>所示不同的计算机架构制造和使用本公开内容的实施方式。特别地，本实施方式可以与非本文所述的软件、硬件和/或操作系统实施方式一起运行。

<heading id="h-0011">结论</heading>

+   <para-num num="[0116]"></para-num>

    应理解详细说明部分用于解释权利要求，而不是任何其他部分。其他部分可以陈述一个或多个但不是所有的由发明者考虑到的示例实施方式，因此，并不打算以任何方式限制本公开内容或附属的权利要求。

+   <para-num num="[0117]"></para-num>

    尽管本公开描述了用于示例领域和应用的示例实施方式，但应理解本公开内容并不限于此。其他实施方式和对其的修改也是可能的，并且属于本公开内容的范围和精神。例如，并不限于本段的概括，实施方式不限于图中和/或本文所述的软件、硬件、固件和/或实体。此外，实施方式（无论是否在本文中明确描述）在超出本文所述示例的领域和应用中具有显著的实用性。

+   <para-num num="[0118]"></para-num>

    在此已描述的实施例中，借助功能性构建块描述了指定功能及其关系的实施方式。这些功能性构建块的边界在此仅为便于描述而任意定义。只要适当执行指定的功能和关系（或其等效物），可以定义替代边界。此外，替代实施例可以使用与此处描述的顺序不同的方式执行功能块、步骤、操作、方法等。

+   <para-num num="[0119]"></para-num>

    提及“一个实施例”，“一个实例实施例”或类似短语的引用表明所描述的实施例可能包括特定的特征、结构或特性，但并非每个实施例都必然包括这些特定的特征、结构或特性。此外，这些短语未必指的是同一实施例。而且，当描述特定的特征、结构或特性与某实施例相关时，相关领域的技术人员可以在其他实施例中融入这种特征、结构或特性，无论此处是否明确提及或描述。此外，有些实施例可以使用“耦合”和“连接”及其派生词来描述。这些术语未必是彼此的同义词。例如，有些实施例可以使用术语“连接”和/或“耦合”来指示两个或多个元素直接物理或电气接触。然而，“耦合”这个术语也可以表示两个或多个元素没有直接接触，但仍然相互协作或互动。

+   <para-num num="[0120]"></para-num>

    此公开的广度和范围不应仅限于上述示例实施例之一，而应仅按照以下权利要求及其等效物来定义。
