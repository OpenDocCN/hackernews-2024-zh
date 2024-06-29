<!--yml

类别：未分类

日期：2024-05-27 12:57:57

-->

# faces.js - 一个用于生成基于向量的卡通面孔的 JavaScript 库

> 来源：[https://zengm.com/facesjs/](https://zengm.com/facesjs/)

## 一个用于生成基于向量的卡通面孔的 JavaScript 库

要加载新的随机面孔，请单击此处或按键盘上的“r”键。

faces.js 是一个生成和显示卡通面孔的 JavaScript 库，有点像任天堂的 Miis。面孔以 SVG 形式绘制，同时也用一个小的 JavaScript 对象表示，这使得你可以存储该对象，稍后再次绘制相同的面孔。

最初 faces.js 是为 [Basketball GM](https://basketball-gm.com/) 和 [ZenGM](https://zengm.com/) 的其他游戏开发的，但现在它被用于几个其他项目中。

# 对用户

你可以使用 [face editor](/facesjs/editor/) 生成一个定制的面孔，然后导出为 JSON/SVG/PNG 格式。

# 对开发者

（这只是来自

[GitHub 仓库的 README](https://github.com/zengm-games/facesjs)

.）

## 安装

```
npm install --save facesjs
```

## 使用

使用 ES 模块导入它：

```
import { display, generate } from "facesjs";
```

或者 CommonJS：

```
const { display, generate } = require("facesjs");
```

接下来，生成一个随机的面孔：

```
const face = generate();
```

然后显示它：

```
// Display in a div with id "my-div-id"
display("my-div-id", face);

// Display in a div you already have a reference to
const element = document.getElementById("my-div-id");
display(element, face);
```

如果你想要一个非随机的面孔，请查看 `face` 变量中的所有可用选项，手动构建面孔。

### 覆盖

`display` 和 `generate` 都接受一个可选参数，用来覆盖随机生成的面孔（对于 `generate`）或提供的面孔（对于 `display`）。例如：

```
# Generate a random face that always has blue skin
const face = generate({ body: { color: "blue" } });

# Display a face, but impose that it has blue skin
display("my-div-id", face, { body: { color: "blue" } });
```

### 选项

`generate` 函数接受第二个可选参数，这个参数以对象的形式提供额外的玩家创建参数。

生成一个女性/男性面孔（默认为男性）：

```
const face = generate(undefined, { gender: "female" });
```

分配一个种族属性，可以是白人、黑人、亚洲人或棕色人种（默认是随机的）：

```
const face = generate(undefined, { race: "white" });
```

或者一起使用：

```
const face = generate(undefined, { gender: "female", race: "asian" });
```

## 导出 SVG

### API

你可以使用 `faceToSvgString` 将面孔对象转换为 SVG 字符串。

```
import { faceToSvgString, generate } from "facesjs";

const face = generate();
const svg = faceToSvgString(face);
```

你还可以像 `display` 那样指定覆盖选项：

```
const svg = faceToSvgString(face, { body: { color: "blue" } });
```

`faceToSvgString` 旨在在 Node.js 中使用。如果你在客户端使用 JS，使用 `display` 将面孔渲染到 DOM 更为高效，然后像这样 [将其转换为 Blob](https://github.com/zengm-games/facesjs/blob/19ce236af6adbf76db29c4e669210b30e1de0e1a/public/editor/downloadFace.ts#L61-L64)。

### CLI

你也可以将 `facesjs` 用作 CLI 程序。从 `generate` 和 `display` 中提供的所有功能在 CLI 中也都可以使用。

#### 示例

输出一个随机面孔到 stdout：

```
$ npx facesjs
```

生成一个蓝色女性面孔并输出到 stdout：

```
$ npx facesjs -j '{"body":{"color":"blue"}}' -g female
```

生成一个白人男性面孔并保存为 test.svg：

```
$ npx facesjs -r white -o test.svg
```

#### 选项

```
-h, --help          Prints this help
-o, --output        Output filename to use rather than stdout
-f, --input-file    Path to a faces.js JSON file to convert to SVG
-j, --input-json    String faces.js JSON object to convert to SVG
-r, --race          Race - white/black/asian/brown, default is random
-g, --gender        Gender - male/female, default is male
```

`--input-file` 和 `--input-json` 可以指定一个完整的面部对象或部分面部对象。如果是部分面部对象，其他特征将是随机的。

## 开发

运行 `yarn run dev` 将执行几个操作：

1.  给你一个 URL，在浏览器中打开面孔编辑器 UI

1.  监视代码的变化

1.  监视面部特征 SVG 文件的变化

1.  当任何代码或 SVG 更改时更新面孔编辑器 UI

这使你在工作时能立即看到你的变化。

## 添加新的面部特征

每张面孔由多个 SVG 图形组装而成。你可以在“svg”文件夹内看到它们。如果你想添加另一个特征，只需创建一个 SVG 图形（使用像 [Inkscape](https://inkscape.org/) 这样的矢量图形编辑器），并将其放入适当的文件夹中。它应该会自动生效。如果没有，请告诉我！这可能是一个 bug。

在创建 SVG 图形时，假设画布大小为 400x600。对于大多数功能来说，在画布上绘制的位置并不重要，因为它会自动识别你的对象并将其放置在适当的位置。但对于头部和发型的 SVG 图形，位置确实很重要。对于这些图形，确保它们与现有的头部和发型 SVG 图形在一个 400x600 的画布上的正确位置相同是必须的。否则它不知道如何放置其他面部特征相对于头部和发型的位置。

如果你发现它没有将面部特征放置在你想要的确切位置，那是因为默认情况下它会找到眼睛/眉毛/嘴巴/鼻子 SVG 图形的中心并将其放置在特定位置。如果这对某个面部特征不合适，可以在代码中覆盖这种行为。例如，请查看 display.js 中“皮诺曹”鼻子的实现，它使用 SVG 的左侧而不是中心来放置它。

如果你想要一个全新的“类别”的面部特征（比如面部毛发、耳环或帽子），你需要在“svg”文件夹内创建一个新的子文件夹，并编辑代码以识别你的新特征。

如果你觉得有些地方令人困惑，请随时联系我寻求帮助！我很希望有人能帮助我制作更好看的面孔 :)

## 鸣谢

[dumbmatter](https://github.com/dumbmatter) 编写了大部分代码，[TravisJB89](https://github.com/TravisJB89) 制作了大部分图形，[Lia Cui](https://liacui.carrd.co/) 制作了大部分女性图形，[gurushida](https://github.com/gurushida) 编写了将面孔导出为 SVG 字符串的代码，[tomkennedy22](https://github.com/tomkennedy22) 编写了大部分编辑器 UI 代码。
