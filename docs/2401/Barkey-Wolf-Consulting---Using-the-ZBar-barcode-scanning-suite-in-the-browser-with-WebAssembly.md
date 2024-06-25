<!--yml

类别：未分类

日期：2024-05-27 14:50:20

-->

# Barkey Wolf Consulting - 使用 ZBar 条形码扫描套件在浏览器中使用 WebAssembly

> 来源：[https://barkeywolf.consulting/posts/barcode-scanner-webassembly/#meet-zbar](https://barkeywolf.consulting/posts/barcode-scanner-webassembly/#meet-zbar)

**我没时间看你干瘪的散文，给我看代码！**

如果您在代码或本文中发现任何错误，请务必告知我！您可以[在 github 仓库中创建一个问题](https://github.com/jjhbw/barcode-scanner-webassembly)或发送邮件。

## 目标

因此，我希望为涉及库存管理的研究项目开发一个跨平台的条形码扫描器。理想情况下，它将成为一个简单的库，可用于中档智能手机的浏览器中。一些约束条件：

+   扫描器需要能够在一个摄像头帧中识别多个条形码

+   它需要能够处理 QR 码，也需要能够处理更广泛的一维条形码（如 EAN-13/UPC-A，UPC-E，EAN-8 等），最好使用相同的代码库，无需用户配置。

## 遇见 ZBar

为了满足上述要求，我查看了几个 Javascript 库。有几个明显的竞争者：

+   [QuaggaJs](https://serratus.github.io/quaggaJS/)

    +   一个非常好的条形码扫描器，但是原生不支持 QR 码。

+   [jsQR](https://github.com/cozmo/jsQR)

    +   Performant，纯 JS QR 码扫描器，带有易于使用的 API。仅扫描 QR 码（每张图片只扫描一个？）。

+   [Instascan](https://github.com/schmich/instascan)

    +   不错的 *Promise* 基础 API，但仅扫描 QR 码。由 [ZXing](https://github.com/zxing/zxing) 的 Emscripten 移植驱动，理论上应该能够通过一些修改处理多种一维码格式。但是，默认情况下不支持。

当我放宽‘仅限 Javascript’的限制，扩大寻找库时，我很快就遇到了[ZBar](https://github.com/ZBar/ZBar)，这是一个经过多次测试的 C 库，广泛用于条形码扫描。它可以处理多种类型的一维和二维（QR）条形码，并且可以在单个帧中检测到多个码。

考虑到 ZBar 是一个 C 库，我很快发现一些人已经开始将其编译为 Javascript 以供在浏览器中使用。[ZBarjs](https://github.com/yurydelendik/zbarjs) 是一个很好的例子，尽管文档不多。ZBarjs 使用了令人惊叹的 [emscripten](http://kripken.github.io/emscripten-site/) 编译器，将从 ZBar 的 C 代码中得到的 LLVM 位码编译为 [asm.js](http://asmjs.org/)，这是一种旨在提高性能的 Javascript 子集。

显然，这是一个‘研究’项目，我觉得不能就此罢手。我为什么要错过利用新的强大技术进行过度工程的机会呢？

遇见 WebAssembly。

## WebAssembly

WebAssembly 是[新](https://webassembly.org/)且*非常*闪亮的。WebAssembly 是浏览器的低级语言，可用作 LLVM 语言（如 C、C++、Rust）的编译目标，但也可用于 TypeScript（使用[Assemblyscript/Binaryen](https://github.com/AssemblyScript/assemblyscript)），同时承诺接近本地执行性能。所有主要浏览器[最近开始支持它](https://webassembly.org/roadmap/)。

我认为 WebAssembly 最棒的事情之一是，它有潜力使 Web 开发不那么依赖于 Javascript，从而将浏览器的领域开放给 LLVM 生态系统中的广泛专业知识和经过时间检验的库。

请参见[此处](https://hacks.mozilla.org/2017/02/what-makes-webassembly-fast/)，了解 Mozilla 关于 WebAssembly 及其潜力的一系列有趣文章。

## 在 WebAssembly 中使用 ZBar

在浏览器中使用 ZBar 的需求是尝试 WebAssembly 并探索其周围新兴生态系统的一个*绝佳*借口。此外，我可能会比使用 ZBar 的 asm.js 版本获得一些性能提升（本文不会涉及任何性能基准，也许以后会）。

编译一个使用 ZBar 检测条形码的 C 函数到 WebAssembly 模块只比编译到 asm.js 稍微复杂一点，这要归功于 Emscripten 的魔法。

请注意，Emscripten 还会生成用于与您的 WebAssembly 代码进行交互所需的 JS/WebAssembly 样板。这使得 Emscripten 不仅仅是一个简单的 C -> WebAssembly 编译器。

## 搭建我们的开发环境

我们需要在系统上安装 emscripten (`emcc`)。您可以按照[这些说明](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm)在本地安装 emscripten，或者，如果您更愿意像我一样使用快速解决方案：将 Docker 容器用作构建环境。我发现 `trzeci/emscripten` 用于此目的非常方便。

启动容器，将您的本地工作目录挂载为一个卷（可在 `src` 中访问），然后进入 bash shell：

```
docker run -v $(pwd):/src -it trzeci/emscripten:1.39.0 /bin/bash 
```

在编译 ZBar 之前，我们需要快速安装一些构建依赖项。因为我只需要编译 ZBar 一次，所以我不在乎这些安装是否会被保留。

在 `trzeci/emscripten` shell 中运行：

```
apt-get update && apt-get install autoconf libtool gettext autogen imagemagick libmagickcore-dev -y 
```

然后我们终于可以开始从源代码编译 ZBar（部分指令修改自[zbarjs](https://github.com/yurydelendik/zbarjs)）。

```
# cd into the mounted host directory
cd /src

# clone the latest ZBar code from the Github repo
git clone https://github.com/ZBar/ZBar

# cd into the directory
cd ZBar

# Delete all -Werror strings from configure.ac
# Don't treat warnings as errors!
sed -i "s/ -Werror//" $(pwd)/configure.ac

# Generate automake files
autoreconf -i

# Configure: disable all unneccesary features
# This may produce red error messages, but it is safe to ignore them (it assumes that
# emscripten is GCC and uses invalid parameters on it)
emconfigure ./configure --without-x --without-jpeg --without-imagemagick --without-npapi --without-gtk --without-python --without-qt --without-xshm --disable-video --disable-pthread

# Compile ZBar
emmake make 
```

那应该就可以了，您会得到一个编译好的库。请注意，您应该**保持此容器 shell 活动**，因为我们以后会需要它。

## 编写用于检测条形码的 C 代码

让我们编写使用 ZBar 库在图像中检测条形码的 C 代码。我们希望稍后从浏览器的 JavaScript 环境调用这些函数。名为 `scan.c` 的 C 文件如下所示：

```
#include <stdlib.h>  #include <stdio.h>  #include <stdint.h>  #include <zbar.h>  #include "emscripten.h"   // External javascript function to pass the retrieved data to. extern void js_output_result(const char *symbolName, const char *data, const int *polygon, const unsigned polysize);

zbar_image_scanner_t *scanner = NULL;

EMSCRIPTEN_KEEPALIVE
int scan_image(uint8_t *raw, int width, int height)
{
	// create the scanner
 scanner = zbar_image_scanner_create();

    // set the scanner density (function will have nonzero return code on error, check your browser console)
 printf("%d \n", zbar_image_scanner_set_config(scanner, 0, ZBAR_CFG_X_DENSITY, 1));
    printf("%d \n", zbar_image_scanner_set_config(scanner, 0, ZBAR_CFG_Y_DENSITY, 1));

	// hydrate a zbar image struct with the image data.
 zbar_image_t *image = zbar_image_create();
    zbar_image_set_format(image, zbar_fourcc('Y', '8', '0', '0'));
    zbar_image_set_size(image, width, height);
    zbar_image_set_data(image, raw, width * height, zbar_image_free_data);

	// scan the image for barcodes
 int n = zbar_scan_image(scanner, image);

	// Iterate over each detected barcode and extract its data and location
 const zbar_symbol_t *symbol = zbar_image_first_symbol(image);
    for (; symbol; symbol = zbar_symbol_next(symbol))
    {
        // Get the data encoded in the detected barcode.
 zbar_symbol_type_t typ = zbar_symbol_get_type(symbol);
        const char *data = zbar_symbol_get_data(symbol);

        // get the polygon describing the bounding box of the barcode in the image
 unsigned poly_size = zbar_symbol_get_loc_size(symbol);

        // return the polygon as a flat array.
 // (Parsing two-dimensional arrays from the wesm heap introduces unnecessary upstream code complexity)
 int poly[poly_size * 2];
        unsigned u = 0;
        for (unsigned p = 0; p < poly_size; p++)
        {
            poly[u] = zbar_symbol_get_loc_x(symbol, p);
            poly[u + 1] = zbar_symbol_get_loc_y(symbol, p);
            u += 2;
        }

        // Output the result to the javascript environment
 js_output_result(zbar_get_symbol_name(typ), data, poly, poly_size);
    }

	// clean up
 zbar_image_destroy(image);
    zbar_image_scanner_destroy(scanner);

    return (0);
}

// this function can be used from the javascript environment to // allocate a buffer on the WebAssembly heap to accomodate the image that is to be scanned. EMSCRIPTEN_KEEPALIVE
uint8_t *create_buffer(int width, int height)
{
    return malloc(width * height * 4 * sizeof(uint8_t));
}

// this function can be used from the javascript environment to free an image buffer. EMSCRIPTEN_KEEPALIVE
void destroy_buffer(uint8_t *p)
{
    free(p);
} 
```

关于这段代码需要注意的几件事：

+   `EMSCRIPTEN_KEEPALIVE` 宏确保函数不会被编译器消除，因为它们在技术上是死代码（即无法从诸如 `main()` 函数之类的地方到达的代码）。

+   我们使用 `extern` 将 `js_output_result` 函数标记为外部符号。此函数将在 JavaScript 粘合代码中定义，并且 Emscripten 编译器将在链接时插入对此函数的引用。

+   有很多优化的空间，比如在多个图像上重用扫描器。现在我们不去讨论这些。

+   这段代码展示了从浏览器 JS 环境中将数据输入和输出到 WebAssembly 环境的方法之一。在 [Emscripten 文档中](https://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/index.html) 可以找到几种替代方法。

## JavaScript 的‘粘合’代码

我们还将创建一个名为 `library.js` 的文件，其中定义了应该从 WebAssembly 环境中调用的 JavaScript 函数。Emscripten 为我们处理了繁琐的部分。如果您想了解更多，请查阅[这个主题的 emscripten 文档](https://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html#implement-a-c-api-in-javascript)。请注意，在下面的代码中，`Module` 和 `LibraryManager` 对象以及 `mergeInto` 函数都是由 Emscripten 提供的全局对象。

我们定义了一个单一的函数：用于提取检测到的条形码数据的函数。需要注意的是：在调用此函数之前，应在 `Module` 对象的 `processResult` 键上设置一个函数。这并不是一种非常优雅的与自己代码交互的方式，但目前足够了。

我们的 `library.js` 文件包含：

```
mergeInto(LibraryManager.library, {
    js_output_result: function (symbol, data, polygon, polygon_size) {
        // function provided by Emscripten to convert WASM heap string pointers to JS strings.
 const Pointer_stringify = Module["UTF8ToString"];

        // Note: new TypedArray(someBuffer) will create a new view onto the same memory chunk, 
 // while new TypedArray(someTypedArray) will copy the data so the original can be freed.
 const resultView = new Int32Array(
            Module.HEAP32.buffer,
            polygon,
            polygon_size * 2
        );
        const coordinates = new Int32Array(resultView);

        // call the downstream processing function that should have been set by the client code
 const downstreamProcessor = Module["processResult"];
        if (downstreamProcessor == null) {
            throw new Error("No downstream processing function set");
        }
        downstreamProcessor(
            Pointer_stringify(symbol),
            Pointer_stringify(data),
            coordinates
        );
    }
}); 
```

## 将我们的代码编译成 WebAssembly 模块

从您的构建容器 shell（`trzeci/emscripten`，请参见上文）运行以下命令：

```
emcc -O3 -s WASM=1 \ --js-library ./library.js \ -s EXTRA_EXPORTED_RUNTIME_METHODS='["cwrap", "UTF8ToString"]' \ -I `pwd`/ZBar/include ./scan.c ./ZBar/zbar/.libs/libzbar.a 
```

对此命令的快速概述：

+   `emcc` 调用 emscripten

+   `-O3` 标志告诉编译器生成高度优化的 WebAssembly 和 JavaScript 代码。

+   `-s WASM=1` 告诉 `emcc` 生成 WebAssembly

+   `—js-library` 告诉 emscripten 在哪里可以找到您的 JavaScript 粘合代码

+   `-s EXTRA_EXPORTED_RUNTIME_METHODS` 告诉 Emscripten 在 JavaScript 环境中注入 `Module` 全局对象，以及这两个实用函数。`crwap` 允许我们调用我们的 C 函数，`Pointer_stringify` 将有助于将数据从 WebAssembly 运行时移动到 JS 中。

+   `-I` 告诉它要编译哪些 C 库/代码

这应该产生以下文件：

+   `a.out.js`，包含了精心打包的 JavaScript 粘合代码（感谢 emscripten！）

+   包含 WebAssembly 模块的 `a.out.wasm`

## 测试我们的条形码扫描器

为了测试我们的条形码扫描模块，让我们编写一个简单的 `index.html`，打开网络摄像头并将图像帧传递给我们的扫描器模块，当找到条形码时，在画布上绘制其边界框和条形码内容。

请注意，我还包括了一个将图像数据转换为灰度的步骤，使用了一个快速的位移技巧。我期望这个操作可以在 WebAssembly 中以更高效的方式实现，但现在我们不必担心这个。

您需要运行一个快速的即时静态文件服务器来在浏览器中测试所有内容。我非常喜欢极简工具[serve](https://www.npmjs.com/package/serve)。

**重要提示：** 确保您的 Web 服务器正确设置了`.wasm`文件的`Content-Type`头。Firefox 58 显然严格遵循 WebAssembly 规范的这一部分，并会拒绝编译`.wasm`文件，导致 Emscripten 兼容层退回到一个较慢的 polyfill。最新版本的[serve](https://www.npmjs.com/package/serve)能够正确处理这一点。

`index.html`

```
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="UTF-8">
	<title>barcode scanner wasm</title>
</head>

<!-- We create some DOM elements necessary for grabbing webcam frames -->
<body>
	<div>
		<!-- note that we hide the live video element using 'display:none', that way only the canvas is rendered -->
		<video id="live" width="320" height="240" autoplay style="border:5px solid #000000; display:none;"> </video>
		<canvas id="canvas" style="border:5px solid #000000"> </canvas>
	</div>

</body>

<!-- Import the javascript bundle produced by Emscripten-->
<script src="/a.out.js"></script>

<!-- The main 'application code' tying it all together -->
<script>

// Execute the application code when the WebAssembly module is ready. Module.onRuntimeInitialized = async _ => {

// wrap all C functions using cwrap. Note that we have to provide crwap with the function signature. const api = {
	scan_image: Module.cwrap('scan_image', '', ['number', 'number', 'number']),
	create_buffer: Module.cwrap('create_buffer', 'number', ['number', 'number']),
	destroy_buffer: Module.cwrap('destroy_buffer', '', ['number']),
};

const video = document.getElementById("live");
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext('2d');
const desiredWidth = 1280;
const desiredHeight = 720;

// settings for the getUserMedia call const constraints = {
	video: {
		// the browser will try to honor this resolution, but it may end up being lower.  width: desiredWidth,
		height: desiredHeight
	}
};

// open the webcam stream navigator.mediaDevices.getUserMedia(constraints).then((stream) => {
	// stream is a MediaStream object  video.srcObject = stream;
	video.play();

	// tell the canvas which resolution we ended up getting from the webcam  const track = stream.getVideoTracks()[0];
	const actualSettings = track.getSettings();
	console.log(actualSettings.width, actualSettings.height)
	canvas.width = actualSettings.width;
	canvas.height = actualSettings.height;

	// every k milliseconds, we draw the contents of the video to the canvas and run the detector.  const timer = setInterval(detectSymbols, 250);

}).catch((e) => {
	throw e
});

function detectSymbols() {
	// grab a frame from the media source and draw it to the canvas  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

	// get the image data from the canvas  const image = ctx.getImageData(0, 0, canvas.width, canvas.height)

	// convert the image data to grayscale  const grayData = []
	const d = image.data;
	for (var i = 0, j = 0; i < d.length; i += 4, j++) {
		grayData[j] = (d[i] * 66 + d[i + 1] * 129 + d[i + 2] * 25 + 4096) >> 8;
	}

	// put the data into the allocated buffer on the wasm heap.  const p = api.create_buffer(image.width, image.height);
	Module.HEAP8.set(grayData, p);

	// call the scanner function  api.scan_image(p, image.width, image.height)

	// clean up 
 //(this is not really necessary in this example as we could reuse the buffer, but is used to demonstrate how you can manage Wasm heap memory from the js environment)  api.destroy_buffer(p);

}

function drawPoly(ctx, poly) {
// drawPoly expects a flat array of coordinates forming a polygon (e.g. [x1,y1,x2,y2,... etc])  ctx.beginPath();
	ctx.moveTo(poly[0], poly[1]);
	for (item = 2; item < poly.length - 1; item += 2) { ctx.lineTo(poly[item], poly[item + 1]) }

	ctx.lineWidth = 2;
	ctx.strokeStyle = "#FF0000";
	ctx.closePath();
	ctx.stroke();
}

// render the string contained in the barcode as text on the canvas function renderData(ctx, data, x, y) {
	ctx.font = "15px Arial";
	ctx.fillStyle = "red";
	ctx.fillText(data, x, y);
}

// set the function that should be called whenever a barcode is detected Module['processResult'] = (symbol, data, polygon) => {
	console.log("Data liberated from WASM heap:")
	console.log(symbol)
	console.log(data)
	console.log(polygon)

	// draw the bounding polygon  drawPoly(ctx, polygon)

	// render the data at the first coordinate of the polygon  renderData(ctx, data, polygon[0], polygon[1] - 10)
}

}
</script>

</html> 
```

## 结论

我们编译了一些使用 ZBar 到 WebAssembly 的 C 代码，编写了一些 Javascript 粘合代码，最终得到了一个在浏览器中很好的条形码扫描器。由于 ZBar 的支持，我们的扫描器兼容多种条形码类型，并能在单个图像帧中检测到多个代码。

但是，还有一些需要改进的地方：

+   目前性能还不算理想。然而，有一些非常明显的优化可以应用，而且我可能会在下一篇文章中专门优化扫描器。简而言之，我首先会看一下：

    +   WebAssembly 层面的更好的内存管理（销毁缓冲区，重用扫描器。）

    +   在更复杂的应用中，我肯定会尽可能地将工作外包（例如 WebAssembly/JS 互操作）给 web worker 处理。

    +   将网络摄像头帧传递到 WebAssembly 堆可以进行优化。首先将每帧渲染到画布似乎有些复杂，但我还没有真正量化这个开销。目前这是我知道的唯一方法...

    +   尝试所有（组合）编译器优化。Emscripten 在各个级别都提供了[相当严格的一套编译器优化](http://kripken.github.io/emscripten-site/docs/optimizing/Optimizing-Code.html)。

    +   减少输入图像的分辨率大大提高了性能。我想知道这对能够检测 QR 码的距离有何影响。

+   扫描器需要条形码具有相当高的分辨率才能检测到它们。这限制了仅限于靠近摄像头的条形码检测。我不认为其他实现在这一点上能超过我们的实现，因为我认为我们正在遇到自信的 QR 码检测的基本分辨率要求。也许 ZBar 内部有一些旋钮可以通过牺牲解码保真度来减少分辨率要求，但我还没有时间去研究。

+   即使有 Emscripten 的帮助，Javascript 和 WebAssembly 之间的互操作性仍然有相当大的学习曲线。然而，随着技术的成熟，我预计会开发出许多良好的抽象层。

+   未来，[从 WebAssembly 访问 DOM API](https://github.com/WebAssembly/host-bindings/blob/master/proposals/host-bindings/Overview.md) 可能会消除对性能关键的 JavaScript 粘合代码的需求。这将是一个改变游戏规则的事件。

总的来说，这个项目是探索使用 WebAssembly 基础知识的一个很好的途径。此外，它展示了一个未来的一瞥，即我们可以利用 WebAssembly 来利用 C（和其他 LLVM 语言）的庞大生态系统，构建更丰富、性能更好的 Web 应用程序。

如果你在代码或本文中发现任何错误，请务必让我知道！你可以[在 github 仓库中创建一个问题](https://github.com/jjhbw/barcode-scanner-webassembly) 或者给我发送邮件。
