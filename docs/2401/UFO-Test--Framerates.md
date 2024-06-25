<!--yml

分类：未分类

日期：2024-05-27 14:25:23

-->

# UFO 测试：帧率

> 来源：[`testufo.com/`](https://testufo.com/)

| ![](img/8451d8425012a66e48b51e953ca58c1f.png) <select class="selectTestMenu setting" title="选择其中一个 Test UFO Motion Tests！" id="masterTests"><option value="" disabled="disabled">----------- 演示 -----------</option> <option value="framerates">比较帧速率：UFO</option> <option value="framerates-versus">比较帧速率：视频游戏运动</option> <option value="framerates-text">比较帧速率：垂直滚动</option> <option value="blackframes">黑帧插入</option> <option value="persistence">视觉坚持 - 光学幻觉</option> <option value="eyetracking">眼动追踪模糊 - 光学幻觉</option> <option value="mousearrow">幻影阵列效应 - 鼠标箭头</option> <option value="stutter">顿挫和撕裂</option> <option value="vrr">可变刷新率模拟</option> <option value="" disabled="disabled">----------- 测试 -----------</option> <option value="ghosting">鬼影/追踪摄像机</option> <option value="blurtrail">模糊轨迹/ PWM</option> <option value="photo">移动照片</option> <option value="chase">追逐方块</option> <option value="mprt">动态图片响应时间（MPRT）</option> <option value="inversion">倒置伪像（棋盘格模式）</option> <option value="aliasing-visibility">混叠可见性</option> <option value="" disabled="disabled">----------- 特殊工具 -----------</option> <option value="frameskipping">跳帧 - 用于显示器超频</option> <option value="refreshrate">带有小数位的刷新率</option> <option value="crosstalk">闪烁交叉 - 用于模糊减少</option> <option value="gtg-vs-mprt">GtG 与 MPRT</option> <option value="scanskew">扫描偏移 - 倾斜/果冻效应</option> <option value="blurbusterslaw">模糊猎人定律 - 运动模糊物理</option> <option value="rainboweffect">彩色连续彩虹效应</option> <option value="interlace">视频交错模拟</option> <option value="scanout">扫描输出 - 用于高速摄像机</option> <option value="flicker">闪烁 - 高速视频或示波器</option> <option value="animation-time-graph">浏览器动画定时精度图</option></select> ![](img/ba87ad3ea418835dc098157af2e1b7f0.png) |
| --- |

多个帧率

UFO 数量： <select title="count" id="count" class="setting" data-format="integer" data-default="3" data-min="1" data-max="8"><option value="1">1 架 UFO</option> <option value="2">2 架 UFO</option> <option value="3">3 架 UFO</option> <option value="4">4 架 UFO</option> <option value="5">5 架 UFO</option> <option value="6">6 架 UFO</option></select>

背景： <select title="background" id="background" class="setting" data-format="text" data-default="stars"><option value="none">无</option> <option value="stars">星星</option></select>

速度：<select title="pps" id="pps" class="setting" data-default="960" data-format="integer" data-min="60" data-max="10000"><option value="120">每秒 120 像素</option> <option value="240">每秒 240 像素</option> <option value="480">每秒 480 像素</option> <option value="720">每秒 720 像素</option> <option value="960">每秒 960 像素</option> <option value="1200">每秒 1200 像素</option> <option value="1440">每秒 1440 像素</option> <option value="1920">每秒 1920 像素</option> <option value="2560">每秒 2560 像素</option> <option value="2880">每秒 2880 像素</option> <option value="3840">每秒 3840 像素</option></select>

<canvas id="area51"></canvas>
