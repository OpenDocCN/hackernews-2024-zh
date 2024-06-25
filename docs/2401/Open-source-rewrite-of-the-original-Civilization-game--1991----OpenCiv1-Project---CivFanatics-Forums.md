<!--yml

分类：未分类

日期：2024-05-27 15:15:56

-->

# Civilization原作（1991年）的开源重写 - OpenCiv1 项目 | CivFanatics 论坛

> 来源：[https://forums.civfanatics.com/threads/open-source-rewrite-of-civilization-1-source-code-openciv1-project.682623/](https://forums.civfanatics.com/threads/open-source-rewrite-of-civilization-1-source-code-openciv1-project.682623/)

据我们所知，实际的游戏机制非常接近，如果不是相同的（虽然我确定我们的居住专家逆向工程师

[@darkpanda](https://forums.civfanatics.com/members/130195/)

，

[@tupi](https://forums.civfanatics.com/members/212558/)

其他人会喜欢将源代码的细节与他们自己的反向DOS代码进行比较），主要是表现让它失望了

-在保存逻辑中存在一个严重的错误，这意味着如果你在你的回合中的某些单位仍有行动力时保存，当你恢复时，所有单位都将在同一回合内恢复所有行动力（DOS实际上也有这个错误，但它只影响到了自动保存，所以要触发它，你必须手动将一个保存移动到自动保存槽，然后加载它）。Windows在每次保存时都会这样做。

-很多城市菜单中的功能都转移到了Windows菜单栏，我想大多数人都更喜欢在城市界面完成所有工作，就像在DOS中一样。

-Windows使用相同的共享音乐作为标题界面和介绍，大部分介绍音乐都被切断，意味着你在大部分介绍中都是在沉默中观看（有关详情，请参阅此帖

[https://forums.civfanatics.com/thre...dows-complete-soundtrack-overhaul-mod.633237/](https://forums.civfanatics.com/threads/civilization-1-for-windows-complete-soundtrack-overhaul-mod.633237/)

）

-在实际外交开始时，民族特有的音乐在外交期间立即被切断，在DOS中，它会继续播放（同样，与标题界面一样，这些音轨非常短，除非你对它们进行修改）

-出于某种原因，Windows版本的保存文件不保存单位名称。这意味着你不能使用自定义单位名称来创建场景，但在DOS中可以。

-没有沿岸、空中花园和其他城市界面水功能的动画。动态海岸线和河流仅在256色模式下存在。

图形显然是主观的，但很多人更喜欢DOS版本的地形和单位图形/颜色方案。当然，这可以是一个可选项。特别是城市界面选择的酒红色和黄色，在我看来有点让人震惊！

编辑 - 还有一个，Windows版本使用了版本3补丁的幸福模型，引入了'非常不快乐的市民'，在DOS中，它们在城市菜单上被标记为红色，在Windows中，没有视觉线索来区分它们与'普通'市民（参见此帖讨论

[https://forums.civfanatics.com/threads/civil-disorder-why.638921/post-16425928](https://forums.civfanatics.com/threads/civil-disorder-why.638921/post-16425928)

）
