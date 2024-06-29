<!--yml

分类：未分类

date: 2024-05-27 14:40:49

-->

# UFO 测试：帧率

> 来源：[https://www.testufo.com/](https://www.testufo.com/)

| ![](img/8451d8425012a66e48b51e953ca58c1f.png) <select class="selectTestMenu setting" title="选择多个测试中的一个 Test UFO Motion Tests!" id="masterTests"><option value="" disabled="disabled">----------- 演示 -----------</option> <option value="framerates">比较帧率：UFO</option> <option value="framerates-versus">比较帧率：视频游戏动作</option> <option value="framerates-text">比较帧率：垂直滚动</option> <option value="blackframes">黑帧插入</option> <option value="persistence">视觉持续性 - 光学错觉</option> <option value="eyetracking">眼球跟踪运动模糊 - 光学错觉</option> <option value="mousearrow">幻影阵列效应 - 鼠标箭头</option> <option value="stutter">顿挫和撕裂</option> <option value="vrr">可变刷新率模拟</option> <option value="" disabled="disabled">----------- 测试 -----------</option> <option value="ghosting">幽灵影像 / 追踪摄像机</option> <option value="blurtrail">模糊轨迹 / PWM</option> <option value="photo">移动照片</option> <option value="chase">追逐方块</option> <option value="mprt">动态图像响应时间（MPRT）</option> <option value="inversion">反向伪影（棋盘图案）</option> <option value="aliasing-visibility">混叠可见性</option> <option value="" disabled="disabled">----------- 特殊工具 -----------</option> <option value="frameskipping">跳帧 - 用于显示器超频</option> <option value="refreshrate">带有小数位的刷新率</option> <option value="crosstalk">闪烁交错以减少模糊</option> <option value="gtg-vs-mprt">GtG 与 MPRT 比较</option> <option value="scanskew">扫描偏斜 - 倾斜 / 凝胶效应</option> <option value="blurbusterslaw">模糊猎犬定律 - 运动模糊物理</option> <option value="rainboweffect">彩色顺序虹效应</option> <option value="interlace">视频间隔模拟</option> <option value="scanout">扫描输出 - 用于高速摄像机</option> <option value="flicker">闪烁 - 高速视频或示波器</option> <option value="animation-time-graph">浏览器动画时间精度图表</option></select> ![](img/ba87ad3ea418835dc098157af2e1b7f0.png) |
| --- |

多种帧率

UFO 数量：<select title="count" id="count" class="setting" data-format="integer" data-default="3" data-min="1" data-max="8"><option value="1">1 个 UFO</option> <option value="2">2 个 UFO</option> <option value="3">3 个 UFO</option> <option value="4">4 个 UFO</option> <option value="5">5 个 UFO</option> <option value="6">6 个 UFO</option></select>

背景：<select title="background" id="background" class="setting" data-format="text" data-default="stars"><option value="none">无</option> <option value="stars">星星</option></select>

Speed: <select title="pps" id="pps" class="setting" data-default="960" data-format="integer" data-min="60" data-max="10000"><option value="120">每秒120像素</option> <option value="240">每秒240像素</option> <option value="480">每秒480像素</option> <option value="720">每秒720像素</option> <option value="960">每秒960像素</option> <option value="1200">每秒1200像素</option> <option value="1440">每秒1440像素</option> <option value="1920">每秒1920像素</option> <option value="2560">每秒2560像素</option> <option value="2880">每秒2880像素</option> <option value="3840">每秒3840像素</option></select>

`<canvas id="area51"></canvas>`
