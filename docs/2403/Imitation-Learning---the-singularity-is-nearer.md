<!--yml

类别：未分类

日期：2024-05-27 15:03:20

-->

# 模仿学习 | 单一性更近了

> 来源：[https://geohot.github.io//blog/jekyll/update/2023/11/18/imitation-learning.html](https://geohot.github.io//blog/jekyll/update/2023/11/18/imitation-learning.html)

7年前我开始了 [comma.ai](https://comma.ai)，从一个简单的想法开始。

1.  收集大量的人类驾驶数据，状态动作对：`(S_t, A_t)`

1.  训练一个监督模型 `f(S_t) -> A_t`

1.  用这个模型驾驶汽车。

最初的确切公式是一个模型，从图像中预测转向角，然后使用PID环路将方向盘调整到期望的角度。

`f_steerangle(img_t) -> steerangle_t`

结果表明这并不起作用，它甚至无法在高速公路上直线行驶。它可能会行驶大约10秒，但随后 [误差会累积](https://www.ri.cmu.edu/pub_files/2015/3/InvitationToImitation_3_1415.pdf)，车辆会向车道的一侧或另一侧漂移（有趣的是，它表现出不愿越过车道线的倾向，但作为ADAS系统是无用的）。

* * *

comma的第一个解决方案是预测车道位置的模型。

`f_lane(img_t) -> (left_lane_pos_t, right_lane_pos_t)`

尽管这本身不能驾驶汽车（尤其是在转弯时），但它作为转向角模型的一个无偏校正，其中 `α` 是校正因子。

`(f_steerangle(img_t) - α*f_lane(img_t).mean()) -> steerangle_t`

这基本上是在第一个版本的 [openpilot](https://github.com/commaai/openpilot) 中发布的。

* * *

一个主要问题是围绕着地面真实性的车道线模型。与测量转向角的简单传感器不同，“车道线”没有明确的定义。它们破坏了系统的端对端性。

我们将车道称为comma的“原罪”，并努力将其消除。我很遗憾地说，今天我们的地面真实性堆栈中仍然存在车道线，但是我们已经在 [消除它们方面取得了惊人的进展](https://blog.comma.ai/end-to-end-lateral-planning/)，以至于2020年的openpilot可以在没有任何车道线的情况下 [行驶在土路上](https://twitter.com/comma_ai/status/1309248079808229377?lang=en)。

然而，消除车道线是通过大量其他手工编码完成的。我们已将此扩展到在 [实验模式下消除显式使用的汽车](https://blog.comma.ai/090release/)，但与纵向情况相比，某些我们的手工编码假设在横向情况下更加脆弱。

* * *

足够有趣的是，事情已经完全回到起点，我们认为我们有了行为克隆的解决方案。我将尽力理解问题，并将解决方案留给读者自行解决。

想象一下随时间运行转向角模型。在每个时间步骤，任何模型都会产生 `ε` 误差。

```
f_steerangle(img_t0) + ε_t0 -> steerangle_t0
f_steerangle(img_t1) + ε_t1 -> steerangle_t1
f_steerangle(img_t2) + ε_t2 -> steerangle_t2
f_steerangle(img_t3) + ε_t3 -> steerangle_t3
... 
```

这个模型很容易训练，并且可以在保留集上实现非常低的损失。但是，它无法驾驶汽车，这是由于`ε`误差改变了下一帧图像。请注意，这些错误不会在训练或测试中改变下一帧图像，但在道路上看起来是这样的：

```
f_steerangle(img_t0) + ε_t0 -> steerangle_t0
f_steerangle(img_t1') + ε_t1 -> steerangle_t1
f_steerangle(img_t2'') + ε_t2 -> steerangle_t2
f_steerangle(img_t3''') + ε_t3 -> steerangle_t3
... 
```

如果它能够有效驾驶，取决于`img_t3'''`离`img_t3`有多远，这取决于极限情况下`ε_t0 + ε_t1 + ε_t2 + ε_t3 + ...`的形状。

`ε`是否相关？在最好的情况下，它们不相关，但在实践中它们几乎总是相关的。即使它们不相关，该误差仍会无限增长。你需要它们是**反相关的**。你需要这个和的极限为0。

* * *

你需要一个累积ε的估算器。在上面，我们使用`f_lane(img_t).mean()`，但想象一下其通用形式。

```
f_steerangle(img_t0) + ε_t0 -> steerangle_t0
f_steerangle(img_t1') + ε_t1 - α*ε_t0  -> steerangle_t1
f_steerangle(img_t2') + ε_t2 - α*(ε_t1 - α*ε_t0) -> steerangle_t2
f_steerangle(img_t3') + ε_t3 - α*(ε_t2 - α*(ε_t1 - α*ε_t0))-> steerangle_t3
... 
```

将时间t处的`α*`表达式替换为一个函数`f_correction(img_t)`，你就回到了上述的工作公式。

`(f_steerangle(img_t) - α*f_correction(img_t)) -> steerangle_t`

十亿美元的问题是，你如何端对端地确定`f_correction`的地面真值？
