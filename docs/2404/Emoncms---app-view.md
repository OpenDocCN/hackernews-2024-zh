<!--yml

category: 未分类

date: 2024-05-27 13:14:57

-->

# Emoncms - 应用视图

> 来源：[https://emoncms.org/app/view?name=MyBoilerIdealLogicH24OpenthermSAT&readkey=1d29c637a4817acdf6e6e271850c9026](https://emoncms.org/app/view?name=MyBoilerIdealLogicH24OpenthermSAT&readkey=1d29c637a4817acdf6e6e271850c9026)

## My Boiler

My Boiler 应用可用于探索锅炉的性能，包括燃料输入、电力消耗、热输出、效率和系统温度。

**自动配置：** 此应用可以根据右侧显示的名称自动配置连接到 emoncms feeds，或者可以通过点击编辑按钮选择 feeds。

使用 power_to_kwh 输入处理器可以从功率 feeds 创建**累计 kWh** feeds，该处理器将功率数据（以瓦特为单位）转换为能耗数据（以 kWh 为单位）。

**公开分享：** 如果要公开分享您的仪表板，请勾选“公开”复选框，并确保通过调整 feeds 页面上的设置使关联 feeds 也变为公开。

**开始日期：** 若要修改累计总燃料和电力消耗、热输出和效率的开始日期，请输入与您所需的起始日期和时间相对应的 Unix 时间戳。
