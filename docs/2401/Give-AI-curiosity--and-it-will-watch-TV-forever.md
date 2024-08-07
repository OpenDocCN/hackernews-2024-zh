<!--yml

category: 未分类

日期：2024-05-27 15:23:05

-->

# 给予 AI 好奇心，它将永远观看电视。

> 来源：[`qz.com/1366484/give-ai-curiosity-and-it-will-watch-tv-forever`](https://qz.com/1366484/give-ai-curiosity-and-it-will-watch-tv-forever)

大多数用于翻译、在 Facebook 上标记照片和优化最佳导航路线的人工智能都依赖于人类提供的信息以启动。我们向算法展示哪些句子在其他语言中等价，不同照片中一个人的样子，以及如何规划汽车的最佳路径。

但一些 AI 研究人员正在探索如何赋予算法好奇心的能力，以便它们可以在没有任何人类指导的情况下学习。OpenAI 与伯克利大学和爱丁堡大学的研究人员合作，最新的[研究](https://pathak22.github.io/large-scale-curiosity/)发现，当 AI 算法被赋予简单的好奇心定义时，它可以在没有任何人类提供信息的情况下探索超过 50 款视频游戏，并且甚至能够击败其中一些。

但好奇心是有代价的。研究人员还发现，由于 AI 代理被奖励于看到新事物，有时它会故意去死以看到游戏结束画面，或者着迷于一个虚假的电视及其遥控器，不停地翻看频道，寻找新的内容。

## 什么是人工好奇心？

OpenAI 团队对人工好奇心的定义相对简单：算法试图预测环境在未来一帧的样子。当下一帧发生时，算法会根据其预测的准确性获得奖励。其理念在于，如果算法能够预测环境中会发生什么，那么它之前已经看到过。

这就是为什么 AI 代理在像超级马里奥这样的游戏中表现得如此出色——游戏的基础是向前探索并进入下一关。

## 电视有何特别之处？

OpenAI 研究员哈里·爱德华兹告诉 Quartz，让 AI 代理人翻看电视频道的想法来自于一个称为嘈杂电视问题的思想实验。电视上的静态噪音是极其随机的，因此一个好奇的 AI 代理永远无法真正预测接下来会发生什么，并被吸引看电视直至永远。在现实世界中，你可以把它想象成完全随机的事物，比如光线从瀑布上折射出的方式。

研究人员通过在 3D 环境中放置数字电视并允许代理按下按钮来更改频道来测试他们的理论。当代理找到电视并开始翻看频道时，新图像的流动使得电视变得不可抗拒。

Edwards 说，有时候 AI 可以从电视节目中解脱出来，但只有当 AI 周围的环境看起来比电视上的下一个节目更有趣时。

## 超越游戏

本研究的重点不仅在于用人工智能打败视频游戏，还在于理解算法如何更好地解释周围的世界。由于这些算法被证明在探索视频游戏的各个角落和缝隙方面效率很高，研究人员表示它们也可以被改进，以使代码调试更容易，或者通过视频游戏来确保没有任何故障。
