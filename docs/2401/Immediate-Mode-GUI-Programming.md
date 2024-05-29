<!--yml
category: 未分类
date: 2024-05-27 14:47:31
-->

# Immediate Mode GUI Programming

> 来源：[https://eliasnaur.com/blog/immediate-mode-gui-programming](https://eliasnaur.com/blog/immediate-mode-gui-programming)

# Immediate Mode GUI Programming

The initial focus of the [Gio](https://gioui.org) project was to create a simple cross-platform Go library for creating user interfaces for mobile and the desktop, avoiding the platform bound and often complex native toolkits. This is also the apparent selling point of Gio.

However, as work progressed it became clear that the immediate mode programming model is a key driver for longer term adoption of Gio. After all, there are plenty of cross-platform GUI toolkits available, but very few attempt the direct and stateless programming model of immediate mode libraries.

This article is an introduction to immediate mode programming, comparing Gio’s approach with popular retained mode toolkits.

## What is immediate mode?

In traditional *retained mode* UI toolkits, the state of the user interface is owned by the toolkit, and event handling, layout and animation is handled internally. The application can only indirectly manipulate the UI state, and responding to user input happen in callbacks.

The following JavaScript program implements a button and a click event handler:

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

	// UI state.
	var btn widget.Clickable
	var count int

	for e := range w.Events() {
		if e, ok := e.(system.FrameEvent); ok {
			gtx := layout.NewContext(&ops, e)

			for btn.Clicked() {
				count++
			}

			layout.W.Layout(gtx, func(gtx layout.Context) layout.Dimensions {
				return layout.Flex{Axis: layout.Vertical}.Layout(gtx,
					layout.Rigid(func(gtx layout.Context) layout.Dimensions {
						return material.H5(th, fmt.Sprintf("Number of clicks: %d", count)).Layout(gtx)
					}),
					layout.Rigid(func(gtx layout.Context) layout.Dimensions {
						return material.Button(th, &btn, "Click me!").Layout(gtx)
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

Notice the differences: the program is in control of the main event
loop, user input is handled as part of program flow, and the
program state is both minimal and cleanly separated. The counter
doesn’t respond to user input, and so is reduced to a (high-level)
drawing command. The same applies to layout; both the aligning and
listing of the two widgets are stateless.

Drawing

If a user interface is a form of I/O system, drawing is the output part.
Mixing custom drawing and animation with higher level widgets is where the
retained model typically struggles.

Here’s [a program](/include/files/bounce/main.go) that displays a
bouncing ball, controlled by a button.

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
btn.innerText = "Throw ball";
root.appendChild(btn);

var ballVisible = false;
var now;
var animHandle;

btn.addEventListener("click", function() {
	ballVisible = !ballVisible;
	if (ballVisible) {
		root.appendChild(ball);
		now = performance.now();
		btn.innerText = "Hide ball";
		animate(now);
	} else {
		cancelAnimationFrame(animHandle);
		btn.innerText = "Throw ball";
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
Several redundant state variables appear in the program: the
`innerText` property, the `ball` object, the callback handle
`animHandle`. As a result, the click listener must reset the extra
state and remember to unregister the callback. A lingering callback
may be invoked when the program is unprepared or leak memory.

Layout

The examples so far demonstrate some advantages of immediate mode
layouts such as centering, aligning and listing interface elements:
operations are stateless, and they’re specified and controlled with
regular programming constructs such as functions, loops and
conditions.

However, state for static layout is about as harmless as having extra
program constants in your program; the advantages are clearer for
dynamic layouts.

This program implements a scrollable list of many rows, with a button
to modify their content.

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
			const numRows = 1e6 // A million rows.
			return s.list.Layout(gtx, numRows, func(gtx layout.Context, i int) layout.Dimensions {
				txt := fmt.Sprintf("Row #%d", i*s.factor)
				return material.Body1(th, txt).Layout(gtx)
			})
		}),
		layout.Rigid(func(gtx layout.Context) layout.Dimensions {
			return layout.S.Layout(gtx, func(gtx layout.Context) layout.Dimensions {
				return material.Button(th, &s.btn, "Multiply by 10").Layout(gtx)
			})
		}),
	)
} 
```

```

User input

Perhaps the most useful effect of state minimization is the
elimination of event callbacks. Callbacks are the retained mode answer
to the question: given an input event such as a mouse click, where and
how should it be delivered? In Gio’s approach filters and unique
handler keys replace callbacks for convenient distribution of events
among widgets.

The following [implementation](/include/files/checkbox/main.go) of a basic checkbox
demonstrates low-level input handling:

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