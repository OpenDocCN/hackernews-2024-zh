<!--yml

category: 未分类

date: 2024-05-27 14:47:31

-->

# 即时模式 GUI 编程

> 来源：[https://eliasnaur.com/blog/immediate-mode-gui-programming](https://eliasnaur.com/blog/immediate-mode-gui-programming)

# 即时模式 GUI 编程

[Gio](https://gioui.org) 项目最初的关注点是创建一个简单的跨平台 Go 库，用于创建移动设备和桌面的用户界面，避免了平台绑定和通常复杂的本机工具包。这也是 Gio 的明显卖点。

然而，随着工作的进展，很明显，即时模式编程模型是 Gio 长期采用的关键驱动因素。毕竟，有很多跨平台 GUI 工具包可用，但很少有尝试即时模式库的直接和无状态编程模型。

本文是对即时模式编程的介绍，比较 Gio 的方法与流行的保留模式工具包。

## 什么是即时模式？

在传统的*保留模式* UI 工具包中，用户界面的状态由工具包拥有，并且事件处理、布局和动画在内部处理。应用程序只能间接操作 UI 状态，并且对用户输入的响应发生在回调中。

下面的 JavaScript 程序实现了一个按钮和一个点击事件处理程序：

```
var count = 0;

var root = document.getElementById("clickcounter");

var label = document.createElement("h3");
label.innerText = "Number of clicks: 0";
root.appendChild(label);

var btn = document.createElement("button");
btn.innerText = "Click Me!";
btn.addEventListener("click", function() {
	count++;
	label.innerText = "Number of clicks:" + count;
});
root.appendChild(btn); 
```

```
In retained mode the label and button must explicitly be created,
added to a widget hierrarchy (the HTML DOM) and the click handler must
be registered. The handler callback is invoked at the discretion
of the UI library, not when convenient for the program.

In the *immediate mode* model, library owned state is minimized, and the
program is responsible for drawing, layout and event handling,
supported by the facilities offered by the UI library.

A [complete Gio version](/include/files/clickcounter/main.go) of the
click counter is reproduced below (import statements omitted).

```

func main() {

    go func() {

        w := app.NewWindow()

        loop(w)

    }()

    app.Main()

}

func loop(w *app.Window) {

    th := material.NewTheme(gofont.Collection())

    var ops op.Ops

    // UI 状态。

    var btn widget.Clickable

    var count int

    for e := range w.Events() {

        if e, ok := e.(system.FrameEvent); ok {

            gtx := layout.NewContext(&ops, e)

            for btn.Clicked() {

                count++

            }

            layout.W.Layout(gtx, func(gtx layout.Context) layout.Dimensions {

                返回 layout.Flex{Axis: layout.Vertical}.Layout(gtx,

                    layout.Rigid(func(gtx layout.Context) layout.Dimensions {

                        return material.H5(th, fmt.Sprintf("点击次数：%d", count)).Layout(gtx)

                    }),

                    layout.Rigid(func(gtx layout.Context) layout.Dimensions {

                        return material.Button(th, &btn, "点击我！").Layout(gtx)

                    }),

                )

            })

            e.Frame(gtx.Ops)

        }

    }

}

```

If your browser supports WebAssembly and WebGL, you can run the
program by clicking the `run` button. It may take a little while to
load.

```

注意区别：程序控制主要事件。

循环，用户输入作为程序流的一部分处理，和

程序状态既简洁又清晰地分离。计数器

不响应用户输入，因此被简化为（高级别的）

绘图命令。布局也是如此；对齐和

两个小部件的列表是无状态的。

绘图

如果用户界面是一种 I/O 系统，绘图就是输出部分。

将自定义绘图和动画与更高级别的小部件混合使用是

保留模型通常会遇到困难。

这是[一个程序](/include/files/bounce/main.go)，显示

由按钮控制的弹跳球。

```
type state struct {
	btn         widget.Clickable
	ballVisible bool
	time        time.Time
}

var programState state

func (s *state) layout(gtx layout.Context, th *material.Theme) {
	for s.btn.Clicked() {
		s.ballVisible = !s.ballVisible
		s.time = gtx.Now
	}

	if s.ballVisible {
		s.drawBall(gtx)
	}
	layout.Center.Layout(gtx, func(gtx layout.Context) layout.Dimensions {
		label := "Throw ball"
		if s.ballVisible {
			label = "Hide ball"
		}
		return material.Button(th, &s.btn, label).Layout(gtx)
	})
}

func (s *state) drawBall(gtx layout.Context) {
	defer op.Save(gtx.Ops).Load()
	const size = 50

	// Animate bounce.
	const (
		v0 = 50.0
		g  = -9.81
	)
	now := gtx.Now
	t := now.Sub(s.time).Seconds()
	t *= 10
	t1 := -v0 / (.5 * g)
	t = math.Mod(t, t1)
	y := v0*t + .5*g*t*t

	// Position.
	bottom := float32(gtx.Constraints.Max.Y)
	op.Offset(f32.Point{X: 20, Y: bottom - size - float32(y)}).Add(gtx.Ops)

	// Draw.
	paint.ColorOp{Color: color.NRGBA{A: 0xff, G: 0xff}}.Add(gtx.Ops)
	clip.RRect{
		Rect: f32.Rectangle{Max: f32.Point{X: size, Y: size}},
		NE:   size * .5, NW: size * .5, SE: size * .5, SW: size * .5,
	}.Op(gtx.Ops).Add(gtx.Ops)
	paint.PaintOp{}.Add(gtx.Ops)

	// Request immediate redraw.
	op.InvalidateOp{}.Add(gtx.Ops)
} 
```

```

See how the high-level button and its centering is naturally mixed
with the low-level animation and drawing of the sphere. There is no
separate animation system; rather, the program draws and requests
redrawing as long as the ball is bouncing, and simply stops when it is
not. The required state is minimized to just the `ballVisible` boolean
and the throwing `time`.

Compare the retained mode version:

```

var root = document.getElementById("bounce");

root.style = "display: flex; position: relative; height: 200px";

var ball = document.createElement("div");

ball.style = "background-color: green; position: absolute; width: 40px; height: 40px; border-radius: 50%";

ball.style.bottom = 0;

var btn = document.createElement("button");

btn.style = "margin: auto";

btn.innerText = "扔球";

root.appendChild(btn);

var ballVisible = false;

var now;

var animHandle;

btn.addEventListener("click", function() {

    ballVisible = !ballVisible;

    if (ballVisible) {

        root.appendChild(ball);

        now = performance.now();

        btn.innerText = "隐藏球";

        animate(now);

    } else {

        cancelAnimationFrame(animHandle);

        btn.innerText = "扔球";

        ball.remove();

    }

});

function animate(time) {

    var v0 = 50.0;

    var g = -9.81;

    var t = (time - now)/1000;

    t *= 10;

    var t1 = -v0 / (.5 * g);

    var t = t % t1;

    var y = v0*t + .5*g*t*t;

    ball.style.bottom = y+"px";

    animHandle = window.requestAnimationFrame(animate);

}

```

```

程序中出现了几个多余的状态变量：包括

`innerText` 属性，`ball` 对象，回调句柄

`animHandle`。因此，点击监听器必须重置额外

状态并记得取消注册回调。悬而未决的回调

可能在程序未准备好或泄漏内存时调用。

布局

到目前为止的示例演示了即时模式的一些优势

诸如居中、对齐和列表化界面元素的布局：

操作是无状态的，并且它们是指定和受控的

常规的编程构造，如函数、循环和

条件。

但是，对于静态布局的状态就像有额外的一样无害

程序中的常量；对于静态布局的优势更为明显

动态布局。

该程序实现了一个可滚动的包含许多行的列表，带有一个按钮

修改它们的内容。

```
var root = document.getElementById("list");

var list = document.createElement("div");
list.style = "overflow-y: scroll; max-height: 200px;"
root.appendChild(list);

var factor = 1;

function addRows() {
	for (var i = 0; i < 1000; i++) {
		var row = document.createElement("p");
		row.innerText = "Row #" + i*factor;
		list.appendChild(row);
	}
}
addRows()

var btn = document.createElement("button");
btn.innerText = "Multiply by 10";
btn.addEventListener("click", function() {
	while (list.firstChild) {
		list.firstChild.remove();
	}
	factor *= 10;
	addRows();
});
root.appendChild(btn); 
```

```
Because every row has a persistent representation to the UI in the
form of `<p>` elements, the program is left with two options for
updating the elements: either go through them one by one, or remove
them all and recreate them. Both options perform worse as the number
of elements grows.

The equivalent [Gio program](/include/files/list/main.go) has just two
variables: the integer factor adjusted by the button, and the list
object that tracks the scroll position. There is no need for a
persistent representation of each row and the list spans a million rows
without impacting performance.

```

type state struct {

    btn    widget.Clickable

    factor int

    list   layout.List

}

var programState state

func (s *state) layout(gtx layout.Context, th *material.Theme) {

    for s.btn.Clicked() {

        s.factor *= 10

    }

    layout.Flex{Axis: layout.Vertical}.Layout(gtx,

        layout.Flexed(1, func(gtx layout.Context) layout.Dimensions {

            const numRows = 1e6 // 一百万行。

            return s.list.Layout(gtx, numRows, func(gtx layout.Context, i int) layout.Dimensions {

                txt := fmt.Sprintf("第 #%d 行", i*s.factor)

                return material.Body1(th, txt).Layout(gtx)

            })

        }),

        layout.Rigid(func(gtx layout.Context) layout.Dimensions {

            return layout.S.Layout(gtx, func(gtx layout.Context) layout.Dimensions {

                return material.Button(th, &s.btn, "乘以 10").Layout(gtx)

            })

        }),

    )

}

```

```

用户输入

或许状态最小化最有用的效果是

消除事件回调。回调是保留模式的答案

对于问题：给定输入事件，如鼠标点击，何时何地

应该如何传递呢？在 Gio 的方法中，过滤器和唯一

处理器键取代了回调，方便地分发事件

在小部件之间。

下面是基本复选框的[实现](/include/files/checkbox/main.go)

展示了低级输入处理：

```
type checkbox struct {
	checked bool
}

var boxes [10]checkbox

func draw(gtx layout.Context) {
	var children []layout.FlexChild
	for i := range boxes {
		box := &boxes[i]
		children = append(children,
			layout.Rigid(func(gtx layout.Context) layout.Dimensions {
				return layout.UniformInset(unit.Dp(10)).Layout(gtx,
					box.layout,
				)
			}),
		)
	}
	layout.Flex{}.Layout(gtx, children...)
}

func (c *checkbox) layout(gtx layout.Context) layout.Dimensions {
	const size = 50

	// Process events using the key, c.
	for _, e := range gtx.Events(c) {
		if e, ok := e.(pointer.Event); ok {
			if e.Type == pointer.Press {
				c.checked = !c.checked
			}
		}
	}

	st := op.Save(gtx.Ops) // Save operation state.

	// Confine input to the area covered by the checkbox.
	pointer.Rect(image.Rectangle{Max: image.Point{
		X: size,
		Y: size,
	}}).Add(gtx.Ops)
	// Declare the filter with the key, c.
	pointer.InputOp{Tag: c, Types: pointer.Press}.Add(gtx.Ops)

	col := color.NRGBA{A: 0xff, R: 0xff} // Red.
	if c.checked {
		col = color.NRGBA{A: 0xff, G: 0xff} // Green.
	}

	// Draw checkbox. Red for unchecked, green for checked.
	paint.ColorOp{Color: col}.Add(gtx.Ops)
	clip.Rect{
		Max: image.Point{
			X: size,
			Y: size,
		},
	}.Add(gtx.Ops)
	paint.PaintOp{}.Add(gtx.Ops)

	st.Load() // Restore operation state.

	// Specify layout dimensions.
	return layout.Dimensions{
		Size: image.Point{
			X: size, Y: size,
		},
	}
} 
```

```

First, notice the simple interface of the checkbox; input handling,
drawing and layout are all contained in a single method.

Second, the input protocol. To separate events belonging to different
widgets, handler keys that uniquely identify the handlers are used.
Keys must be declared in the previous frame, along with any predicates
on the input (hit area for pointer input).

Finally, the convenience. The program is free to process events
anywhere during the frame, and there is no callback or state to
unregister. Key declarations are automatically discarded before
the beginning of each frame.

Resources

Sponsoring

I work on open source projects such as [Gio](https://gioui.org),
supported by sponsorships. If you find my work useful, please consider
[sponsoring me](https://eliasnaur.com/sponsor).

```

```

```

```

```

```

```
