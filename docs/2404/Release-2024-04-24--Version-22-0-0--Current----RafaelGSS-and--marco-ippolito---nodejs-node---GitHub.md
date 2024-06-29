<!--yml

category: 未分类

date: 2024-05-27 13:29:37

-->

# Release 2024-04-24, Version 22.0.0 (Current), @RafaelGSS and @marco-ippolito · nodejs/node · GitHub

> 来源：[https://github.com/nodejs/node/releases/tag/v22.0.0](https://github.com/nodejs/node/releases/tag/v22.0.0)

我们很高兴地宣布 Node.js 22 版本的发布！

亮点包括 require() ESM 图形，WebSocket 客户端，更新的 V8 JavaScript 引擎等等！

作为提醒，Node.js 22 将在十月进入长期支持（LTS），但在那之前，它将是接下来六个月的“当前”版本。

鼓励您探索这个最新版本带来的新功能和优势，并评估它们对您的应用程序可能产生的影响。

### Other Notable Changes

+   [[`25c79f3331`](https://github.com/nodejs/node/commit/25c79f3331)] - **esm**: drop support for import assertions (Nicolò Ribaudo) [#52104](https://github.com/nodejs/node/pull/52104)

+   [[`818c10e86d`](https://github.com/nodejs/node/commit/818c10e86d)] - **lib**: improve perf of `AbortSignal` creation (Raz Luvaton) [#52408](https://github.com/nodejs/node/pull/52408)

+   [[`4f68c7c1c9`](https://github.com/nodejs/node/commit/4f68c7c1c9)] - **watch**: mark as stable (Moshe Atlow) [#52074](https://github.com/nodejs/node/pull/52074)

+   [[`02b0bc01fe`](https://github.com/nodejs/node/commit/02b0bc01fe)] - **(SEMVER-MAJOR)** **deps**: update V8 to 12.4.254.14 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`c975384264`](https://github.com/nodejs/node/commit/c975384264)] - **(SEMVER-MAJOR)** **lib**: enable WebSocket by default (Aras Abbasi) [#51594](https://github.com/nodejs/node/pull/51594)

+   [[`1abff07392`](https://github.com/nodejs/node/commit/1abff07392)] - **(SEMVER-MAJOR)** **stream**: bump default highWaterMark (Robert Nagy) [#52037](https://github.com/nodejs/node/pull/52037)

+   [[`1a5acd0638`](https://github.com/nodejs/node/commit/1a5acd0638)] - **(SEMVER-MAJOR)** **v8**: enable maglev on supported architectures (Keyhan Vakil) [#51360](https://github.com/nodejs/node/pull/51360)

+   [[`128c60d906`](https://github.com/nodejs/node/commit/128c60d906)] - **(SEMVER-MINOR)** **cli**: implement `node --run <script-in-package-json>` (Yagiz Nizipli) [#52190](https://github.com/nodejs/node/pull/52190)

+   [[`151d365ad1`](https://github.com/nodejs/node/commit/151d365ad1)] - **(SEMVER-MINOR)** **fs**: expose glob and globSync (Moshe Atlow) [#51912](https://github.com/nodejs/node/pull/51912)

+   [[`5f7fad2605`](https://github.com/nodejs/node/commit/5f7fad2605)] - **(SEMVER-MINOR)** **module**: support require()ing synchronous ESM graphs (Joyee Cheung) [#51977](https://github.com/nodejs/node/pull/51977)

### Semver-Major Commits

+   [[`2b1e7c2fcb`](https://github.com/nodejs/node/commit/2b1e7c2fcb)] - **(SEMVER-MAJOR)** **build**: compile with C++20 support on Windows (StefanStojanovic) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`12d00f1479`](https://github.com/nodejs/node/commit/12d00f1479)] - **(SEMVER-MAJOR)** **build**: 重置嵌入字符串为 "-node.0" (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`5f08e11a3c`](https://github.com/nodejs/node/commit/5f08e11a3c)] - **(SEMVER-MAJOR)** **build**: 重置嵌入字符串为 "-node.0" (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`94f0369d1d`](https://github.com/nodejs/node/commit/94f0369d1d)] - **(SEMVER-MAJOR)** **build**: 重置嵌入字符串为 "-node.0" (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`58674cd1d8`](https://github.com/nodejs/node/commit/58674cd1d8)] - **(SEMVER-MAJOR)** **build**: 重置嵌入字符串为 "-node.0" (Michaël Zasso) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`60e836427e`](https://github.com/nodejs/node/commit/60e836427e)] - **(SEMVER-MAJOR)** **console**: 在 console.assert() 中将非字符串视为单独参数 (Jacob Hummer) [#49722](https://github.com/nodejs/node/pull/49722)

+   [[`d62ab3a1ef`](https://github.com/nodejs/node/commit/d62ab3a1ef)] - **(SEMVER-MAJOR)** **crypto**: 运行时弃用 hmac 构造函数 (Marco Ippolito) [#52071](https://github.com/nodejs/node/pull/52071)

+   [[`de0602d190`](https://github.com/nodejs/node/commit/de0602d190)] - **(SEMVER-MAJOR)** **crypto**: 运行时弃用 Hash 构造函数 (Marco Ippolito) [#51880](https://github.com/nodejs/node/pull/51880)

+   [[`215f4d04b7`](https://github.com/nodejs/node/commit/215f4d04b7)] - **(SEMVER-MAJOR)** **crypto**: 将 createCipher 和 createDecipher 移至 eol (Marco Ippolito) [#50973](https://github.com/nodejs/node/pull/50973)

+   [[`30801b8aaf`](https://github.com/nodejs/node/commit/30801b8aaf)] - **(SEMVER-MAJOR)** **deps**: V8: 拣选 cd10ad7cdbe5 (Joyee Cheung) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`521b629ab1`](https://github.com/nodejs/node/commit/521b629ab1)] - **(SEMVER-MAJOR)** **deps**: V8: 撤销 CL 5331688 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`3795e97e6c`](https://github.com/nodejs/node/commit/3795e97e6c)] - **(SEMVER-MAJOR)** **deps**: 补丁 V8 以支持 MSVC 编译 (StefanStojanovic) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`5bde9e677d`](https://github.com/nodejs/node/commit/5bde9e677d)] - **(SEMVER-MAJOR)** **deps**: 屏蔽内部 V8 废弃警告 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`46e628c6f2`](https://github.com/nodejs/node/commit/46e628c6f2)] - **(SEMVER-MAJOR)** **deps**: 补丁 V8 以避免重复的 zlib 符号 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`f824e40a82`](https://github.com/nodejs/node/commit/f824e40a82)] - **(SEMVER-MAJOR)** **deps**: 移除 V8 中对 C++20 特性的使用 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`d2c84c9a13`](https://github.com/nodejs/node/commit/d2c84c9a13)] - **(SEMVER-MAJOR)** **deps**: 避免 ASan 编译错误 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`95d6045bdb`](https://github.com/nodejs/node/commit/95d6045bdb)] - **(SEMVER-MAJOR)** **deps**: 禁用 V8 的并发 sparkplug 编译 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`00f55f5743`](https://github.com/nodejs/node/commit/00f55f5743)] - **(SEMVER-MAJOR)** **deps**: 屏蔽无关的 V8 警告 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`764085aa66`](https://github.com/nodejs/node/commit/764085aa66)] - **(SEMVER-MAJOR)** **deps**: 总是将 V8_EXPORT_PRIVATE 定义为无操作 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`02b0bc01fe`](https://github.com/nodejs/node/commit/02b0bc01fe)] - **(SEMVER-MAJOR)** **deps**: 更新 V8 至 12.4.254.14 (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`0ec50a19dd`](https://github.com/nodejs/node/commit/0ec50a19dd)] - **(SEMVER-MAJOR)** **deps**: V8: cherry-pick cd10ad7cdbe5 (Joyee Cheung) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`021b0b7dee`](https://github.com/nodejs/node/commit/021b0b7dee)] - **(SEMVER-MAJOR)** **deps**: V8: 后移 c4be0a97f981 (Richard Lau) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`681aaf85c7`](https://github.com/nodejs/node/commit/681aaf85c7)] - **(SEMVER-MAJOR)** **deps**: 屏蔽内部 V8 弃用警告 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`c563a1c4e4`](https://github.com/nodejs/node/commit/c563a1c4e4)] - **(SEMVER-MAJOR)** **deps**: 修补 V8 以支持 MSVC 编译 (Stefan Stojanovic) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`11e94b9987`](https://github.com/nodejs/node/commit/11e94b9987)] - **(SEMVER-MAJOR)** **deps**: 修补 V8 以避免重复的 zlib 符号 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`856163e23c`](https://github.com/nodejs/node/commit/856163e23c)] - **(SEMVER-MAJOR)** **deps**: 移除 V8 中的 C++20 特性 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`b530214127`](https://github.com/nodejs/node/commit/b530214127)] - **(SEMVER-MAJOR)** **deps**: 避免 ASan 编译错误 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`8054f69dd9`](https://github.com/nodejs/node/commit/8054f69dd9)] - **(SEMVER-MAJOR)** **deps**: 禁用 V8 的并发 sparkplug 编译 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`dee908be42`](https://github.com/nodejs/node/commit/dee908be42)] - **(SEMVER-MAJOR)** **deps**: 屏蔽无关的 V8 警告 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`cf069414ee`](https://github.com/nodejs/node/commit/cf069414ee)] - **(SEMVER-MAJOR)** **deps**: 总是将 V8_EXPORT_PRIVATE 定义为无操作 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`cc5792dd85`](https://github.com/nodejs/node/commit/cc5792dd85)] - **(SEMVER-MAJOR)** **deps**: 更新 V8 到 12.3.219.16 (Michaël Zasso) [#52293](https://github.com/nodejs/node/pull/52293)

+   [[`61a0d3b4c4`](https://github.com/nodejs/node/commit/61a0d3b4c4)] - **(SEMVER-MAJOR)** **deps**: V8: 后移 c4be0a97f981 (Richard Lau) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`f55380a725`](https://github.com/nodejs/node/commit/f55380a725)] - **(SEMVER-MAJOR)** **deps**: V8: 樱桃选取 f8d5e576b814 (Richard Lau) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`b9d806a2dd`](https://github.com/nodejs/node/commit/b9d806a2dd)] - **(SEMVER-MAJOR)** **deps**: 补丁 V8 以支持 MSVC 编译 (StefanStojanovic) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`63b58bc17b`](https://github.com/nodejs/node/commit/63b58bc17b)] - **(SEMVER-MAJOR)** **deps**: 补丁 V8 避免重复的 zlib 符号 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`86056353c4`](https://github.com/nodejs/node/commit/86056353c4)] - **(SEMVER-MAJOR)** **deps**: 从 V8 中删除 C++20 特性的使用 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`2e0efc1c8d`](https://github.com/nodejs/node/commit/2e0efc1c8d)] - **(SEMVER-MAJOR)** **deps**: 避免 ASan 编译错误 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`59e6f62e34`](https://github.com/nodejs/node/commit/59e6f62e34)] - **(SEMVER-MAJOR)** **deps**: 禁用 V8 并行 sparkplug 编译 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`0423f7e27e`](https://github.com/nodejs/node/commit/0423f7e27e)] - **(SEMVER-MAJOR)** **deps**: 沉默无关的 V8 警告 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`f36620806d`](https://github.com/nodejs/node/commit/f36620806d)] - **(SEMVER-MAJOR)** **deps**: 总是将 V8_EXPORT_PRIVATE 定义为无操作 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`09a8440b45`](https://github.com/nodejs/node/commit/09a8440b45)] - **(SEMVER-MAJOR)** **deps**: 更新 V8 到 12.2.281.27 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`0da3beebfc`](https://github.com/nodejs/node/commit/0da3beebfc)] - **(SEMVER-MAJOR)** **deps**: V8: 樱桃选取 de611e69ad51 (Keyhan Vakil) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`b982335637`](https://github.com/nodejs/node/commit/b982335637)] - **(SEMVER-MAJOR)** **deps**: V8: 樱桃选取 0fd478bcdabd (Joyee Cheung) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`481a90116c`](https://github.com/nodejs/node/commit/481a90116c)] - **(SEMVER-MAJOR)** **deps**: V8: 樱桃选取 0f9ebbc672c7 (Chengzhong Wu) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`782addbdc3`](https://github.com/nodejs/node/commit/782addbdc3)] - **(SEMVER-MAJOR)** **deps**: V8: 拾取 8f0b94671ddb（Lu Yahan）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`b682e7f540`](https://github.com/nodejs/node/commit/b682e7f540)] - **(SEMVER-MAJOR)** **deps**: V8: 拾取 f7d000a7ae7b（Luke Albao）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`a60090c52f`](https://github.com/nodejs/node/commit/a60090c52f)] - **(SEMVER-MAJOR)** **deps**: V8: 拾取 25902244ad1a（Joyee Cheung）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`8441d1fc18`](https://github.com/nodejs/node/commit/8441d1fc18)] - **(SEMVER-MAJOR)** **deps**: 修复 V8 中重复 zlib 符号的问题（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`e8e9bbd7a9`](https://github.com/nodejs/node/commit/e8e9bbd7a9)] - **(SEMVER-MAJOR)** **deps**: 从 V8 中移除对 C++20 特性的使用（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`785d5cd006`](https://github.com/nodejs/node/commit/785d5cd006)] - **(SEMVER-MAJOR)** **deps**: 避免 ASan 编译错误（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`7071c1dafd`](https://github.com/nodejs/node/commit/7071c1dafd)] - **(SEMVER-MAJOR)** **deps**: 禁用 V8 并发 sparkplug 编译（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`d1d60b297d`](https://github.com/nodejs/node/commit/d1d60b297d)] - **(SEMVER-MAJOR)** **deps**: 消除不相关的 V8 警告（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`5b240c62f9`](https://github.com/nodejs/node/commit/5b240c62f9)] - **(SEMVER-MAJOR)** **deps**: 始终将 V8_EXPORT_PRIVATE 定义为无操作（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`d8c97e4857`](https://github.com/nodejs/node/commit/d8c97e4857)] - **(SEMVER-MAJOR)** **deps**: 更新 V8 至 11.9.169.7（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`b9df88a8c2`](https://github.com/nodejs/node/commit/b9df88a8c2)] - **(SEMVER-MAJOR)** **doc**: 运行时废弃标志 --trace-atomics-wait（marco-ippolito）[#51179](https://github.com/nodejs/node/pull/51179)

+   [[`9ba5df30b4`](https://github.com/nodejs/node/commit/9ba5df30b4)] - **(SEMVER-MAJOR)** **doc**: 将 FreeBSD 实验性支持升级至 13.2（Michaël Zasso）[#51231](https://github.com/nodejs/node/pull/51231)

+   [[`900d79caf2`](https://github.com/nodejs/node/commit/900d79caf2)] - **(SEMVER-MAJOR)** **doc**: 为废弃的 utils 添加迁移路径（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`8206f6bb7f`](https://github.com/nodejs/node/commit/8206f6bb7f)] - **(SEMVER-MAJOR)** **fs**: 运行时废弃 fs.Stats 构造函数（Marco Ippolito）[#52067](https://github.com/nodejs/node/pull/52067)

+   [[`c14133503a`](https://github.com/nodejs/node/commit/c14133503a)] - **(SEMVER-MAJOR)** **fs**: 使用私有字段而非符号来处理 `Dir`（Jungku Lee）[#51037](https://github.com/nodejs/node/pull/51037)

+   [[`abbdc3efaa`](https://github.com/nodejs/node/commit/abbdc3efaa)] - **(SEMVER-MAJOR)** **fs**: 将 stats 的日期字段设为延迟加载（Yagiz Nizipli）[#50908](https://github.com/nodejs/node/pull/50908)

+   [[`4b76ccea95`](https://github.com/nodejs/node/commit/4b76ccea95)] - **(SEMVER-MAJOR)** **http**: 在 setHeader 调用后保留原始头部的重复值（Tim Perry）[#50394](https://github.com/nodejs/node/pull/50394)

+   [[`c975384264`](https://github.com/nodejs/node/commit/c975384264)] - **(SEMVER-MAJOR)** **lib**: 默认启用 WebSocket（Aras Abbasi）[#51594](https://github.com/nodejs/node/pull/51594)

+   [[`351495e938`](https://github.com/nodejs/node/commit/351495e938)] - **(SEMVER-MAJOR)** **lib,test**: 处理新的 Iterator 全局对象（Michaël Zasso）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`a8b21fdc90`](https://github.com/nodejs/node/commit/a8b21fdc90)] - **(SEMVER-MAJOR)** **process**: 在打印结果之前等待 `'exit'` 事件（Antoine du Hamel）[#52172](https://github.com/nodejs/node/pull/52172)

+   [[`582ff5037c`](https://github.com/nodejs/node/commit/582ff5037c)] - **(SEMVER-MAJOR)** **src**: 更新 NODE_MODULE_VERSION 到 127（Michaël Zasso）[#52465](https://github.com/nodejs/node/pull/52465)

+   [[`c5c4b50260`](https://github.com/nodejs/node/commit/c5c4b50260)] - **(SEMVER-MAJOR)** **src**: 更新 NODE_MODULE_VERSION 到 126（Michaël Zasso）[#52293](https://github.com/nodejs/node/pull/52293)

+   [[`d248639285`](https://github.com/nodejs/node/commit/d248639285)] - **(SEMVER-MAJOR)** **src**: 使用支持的 API 获取停滞的 TLA 消息（Michaël Zasso）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`d34b02db4c`](https://github.com/nodejs/node/commit/d34b02db4c)] - **(SEMVER-MAJOR)** **src**: 更新默认的 V8 平台，以重写具有位置的函数（Etienne Pierre-Doray）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`d9c47e9b5f`](https://github.com/nodejs/node/commit/d9c47e9b5f)] - **(SEMVER-MAJOR)** **src**: 添加缺失的 TryCatch（Michaël Zasso）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`5cddd3b2d8`](https://github.com/nodejs/node/commit/5cddd3b2d8)] - **(SEMVER-MAJOR)** **src**: 更新 NODE_MODULE_VERSION 到 124（Michaël Zasso）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`1528846ada`](https://github.com/nodejs/node/commit/1528846ada)] - **(SEMVER-MAJOR)** **src**: 使用非废弃的 v8::Uint8Array::kMaxLength（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`7166986626`](https://github.com/nodejs/node/commit/7166986626)] - **(SEMVER-MAJOR)** **src**: 适应 v8::Exception API 的变化（Michaël Zasso）[#50115](https://github.com/nodejs/node/pull/50115)

+   [[`4782818020`](https://github.com/nodejs/node/commit/4782818020)] - **(SEMVER-MAJOR)** **src**: 使用不再被弃用的 CreateSyntheticModule 版本 (Michaël Zasso) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`2cff0ce411`](https://github.com/nodejs/node/commit/2cff0ce411)] - **(SEMVER-MAJOR)** **src**: 将 NODE_MODULE_VERSION 更新为 122 (Michaël Zasso) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`1abff07392`](https://github.com/nodejs/node/commit/1abff07392)] - **(SEMVER-MAJOR)** **stream**: 提升默认 highWaterMark (Robert Nagy) [#52037](https://github.com/nodejs/node/pull/52037)

+   [[`9efc84a2cb`](https://github.com/nodejs/node/commit/9efc84a2cb)] - **(SEMVER-MAJOR)** **test**: 将 test-worker-arraybuffer-zerofill 标记为 flaky (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`84c2e712eb`](https://github.com/nodejs/node/commit/84c2e712eb)] - **(SEMVER-MAJOR)** **test**: 将一些与 GC 相关的测试标记为 flaky (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`cdc4437b87`](https://github.com/nodejs/node/commit/cdc4437b87)] - **(SEMVER-MAJOR)** **test**: 在内存泄漏测试中允许稍微更大的差异 (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`515b007fae`](https://github.com/nodejs/node/commit/515b007fae)] - **(SEMVER-MAJOR)** **test**: 用 alway-turbofan 替换 always-opt 标志 (Michaël Zasso) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`2341805eb2`](https://github.com/nodejs/node/commit/2341805eb2)] - **(SEMVER-MAJOR)** **test**: 移除创建非常大缓冲区的测试 (Michaël Zasso) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`941cef5636`](https://github.com/nodejs/node/commit/941cef5636)] - **(SEMVER-MAJOR)** **test**: 适应新的 V8 受信任内存空间 (Michaël Zasso) [#50115](https://github.com/nodejs/node/pull/50115)

+   [[`29de7f82cd`](https://github.com/nodejs/node/commit/29de7f82cd)] - **(SEMVER-MAJOR)** **test_runner**: 从输出中省略被过滤的测试 (Colin Ihrig) [#52221](https://github.com/nodejs/node/pull/52221)

+   [[`00dc6d9d97`](https://github.com/nodejs/node/commit/00dc6d9d97)] - **(SEMVER-MAJOR)** **test_runner**: 改进 `--test-name-pattern` 以允许匹配单个测试 (Michał Drobniak) [#51577](https://github.com/nodejs/node/pull/51577)

+   [[`5def8019d5`](https://github.com/nodejs/node/commit/5def8019d5)] - **(SEMVER-MAJOR)** **tools**: 为 12.4 更新 V8 gypfiles (Michaël Zasso) [#52465](https://github.com/nodejs/node/pull/52465)

+   [[`c22793d050`](https://github.com/nodejs/node/commit/c22793d050)] - **(SEMVER-MAJOR)** **tools**: 将 v8_abseil 大致移植到 gyp (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`ffb0302f0c`](https://github.com/nodejs/node/commit/ffb0302f0c)] - **(SEMVER-MAJOR)** **tools**: 为 12.2 更新 V8 gypfiles (Michaël Zasso) [#51362](https://github.com/nodejs/node/pull/51362)

+   [[`aadea12440`](https://github.com/nodejs/node/commit/aadea12440)] - **(SEMVER-MAJOR)** **tools**: 更新 V8 的 gypfiles 到 12.1 版本（Michaël Zasso）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`7784773967`](https://github.com/nodejs/node/commit/7784773967)] - **(SEMVER-MAJOR)** **tools**: 更新 V8 的 gypfiles 到 12.0 版本（Michaël Zasso）[#51362](https://github.com/nodejs/node/pull/51362)

+   [[`9fe0424baa`](https://github.com/nodejs/node/commit/9fe0424baa)] - **(SEMVER-MAJOR)** **trace_events**: 使用私有字段而不是符号来进行 `Tracing`（Jungku Lee）[#51180](https://github.com/nodejs/node/pull/51180)

+   [[`e96cd25007`](https://github.com/nodejs/node/commit/e96cd25007)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.log`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`6cf20d5e43`](https://github.com/nodejs/node/commit/6cf20d5e43)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isUndefined`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`09e424921f`](https://github.com/nodejs/node/commit/09e424921f)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isSymbol`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`80b6bfd4e9`](https://github.com/nodejs/node/commit/80b6bfd4e9)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isString`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`d419edded9`](https://github.com/nodejs/node/commit/d419edded9)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isRegExp`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`e0b8de78ed`](https://github.com/nodejs/node/commit/e0b8de78ed)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isPrimitive`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`5478e1129a`](https://github.com/nodejs/node/commit/5478e1129a)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isObject`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`b05b1dd541`](https://github.com/nodejs/node/commit/b05b1dd541)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isNumber`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`5af9bf5f6a`](https://github.com/nodejs/node/commit/5af9bf5f6a)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isNullOrUndefined`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`860a10e10e`](https://github.com/nodejs/node/commit/860a10e10e)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isNull`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`70330f5c2b`](https://github.com/nodejs/node/commit/70330f5c2b)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 `util.isFunction`（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`7c69c33acc`](https://github.com/nodejs/node/commit/7c69c33acc)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 util.isError（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`a0c5b871a9`](https://github.com/nodejs/node/commit/a0c5b871a9)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 util.isDate（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`3c670cb15d`](https://github.com/nodejs/node/commit/3c670cb15d)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 util.isBuffer（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`c17a448ca9`](https://github.com/nodejs/node/commit/c17a448ca9)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 util.isBoolean（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`fbb2f891aa`](https://github.com/nodejs/node/commit/fbb2f891aa)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 util.isArray（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`22d8062e42`](https://github.com/nodejs/node/commit/22d8062e42)] - **(SEMVER-MAJOR)** **util**: 运行时弃用 util._extend（Marco Ippolito）[#50488](https://github.com/nodejs/node/pull/50488)

+   [[`1a5acd0638`](https://github.com/nodejs/node/commit/1a5acd0638)] - **(SEMVER-MAJOR)** **v8**: 在支持的架构上启用 maglev（Keyhan Vakil）[#51360](https://github.com/nodejs/node/pull/51360)

### Semver-Minor Commits

+   [[`128c60d906`](https://github.com/nodejs/node/commit/128c60d906)] - **(SEMVER-MINOR)** **cli**: 实现 `node --run <script-in-package-json>`（Yagiz Nizipli）[#52190](https://github.com/nodejs/node/pull/52190)

+   [[`f69946b905`](https://github.com/nodejs/node/commit/f69946b905)] - **(SEMVER-MINOR)** **deps**: 更新 simdutf 到 5.0.0（Daniel Lemire）[#52138](https://github.com/nodejs/node/pull/52138)

+   [[`828ad42eee`](https://github.com/nodejs/node/commit/828ad42eee)] - **(SEMVER-MINOR)** **deps**: 更新 undici 到 6.3.0（Node.js GitHub Bot）[#51462](https://github.com/nodejs/node/pull/51462)

+   [[`05f8172188`](https://github.com/nodejs/node/commit/05f8172188)] - **(SEMVER-MINOR)** **deps**: 更新 undici 到 6.2.1（Node.js GitHub Bot）[#51278](https://github.com/nodejs/node/pull/51278)

+   [[`a0c466810a`](https://github.com/nodejs/node/commit/a0c466810a)] - **(SEMVER-MINOR)** **doc**: 废弃 fs.Stats 的公共构造函数（Marco Ippolito）[#51879](https://github.com/nodejs/node/pull/51879)

+   [[`151d365ad1`](https://github.com/nodejs/node/commit/151d365ad1)] - **(SEMVER-MINOR)** **fs**: 公开 glob 和 globSync（Moshe Atlow）[#51912](https://github.com/nodejs/node/pull/51912)

+   [[`5f7fad2605`](https://github.com/nodejs/node/commit/5f7fad2605)] - **(SEMVER-MINOR)** **module**: 支持同步 ESM 图的 require()（Joyee Cheung）[#51977](https://github.com/nodejs/node/pull/51977)

+   [[`009665fb56`](https://github.com/nodejs/node/commit/009665fb56)] - **(SEMVER-MINOR)** **report**: 添加 `--report-exclude-network` 选项 (Ethan Arrowood) [#51645](https://github.com/nodejs/node/pull/51645)

+   [[`80f86e5d02`](https://github.com/nodejs/node/commit/80f86e5d02)] - **(SEMVER-MINOR)** **src**: 添加 C++ ProcessEmitWarningSync() (Joyee Cheung) [#51977](https://github.com/nodejs/node/pull/51977)

+   [[`78be0d0f1c`](https://github.com/nodejs/node/commit/78be0d0f1c)] - **(SEMVER-MINOR)** **src**: 添加 uv_get_available_memory 报告和处理 (theanarkh) [#52023](https://github.com/nodejs/node/pull/52023)

+   [[`b34512e38e`](https://github.com/nodejs/node/commit/b34512e38e)] - **(SEMVER-MINOR)** **src**: 为 Environment 预加载函数 (Cheng Zhao) [#51539](https://github.com/nodejs/node/pull/51539)

+   [[`7d258db1d7`](https://github.com/nodejs/node/commit/7d258db1d7)] - **(SEMVER-MINOR)** **stream**: 支持类型化数组 (IlyasShabi) [#51866](https://github.com/nodejs/node/pull/51866)

+   [[`5276c0d5d4`](https://github.com/nodejs/node/commit/5276c0d5d4)] - **(SEMVER-MINOR)** **test_runner**: 添加 suite() (Colin Ihrig) [#52127](https://github.com/nodejs/node/pull/52127)

+   [[`84de97a61e`](https://github.com/nodejs/node/commit/84de97a61e)] - **(SEMVER-MINOR)** **test_runner**: 支持强制退出 (Colin Ihrig) [#52038](https://github.com/nodejs/node/pull/52038)

+   [[`aac5ad901d`](https://github.com/nodejs/node/commit/aac5ad901d)] - **(SEMVER-MINOR)** **test_runner**: 添加 `test:complete` 事件以反映执行顺序 (Moshe Atlow) [#51909](https://github.com/nodejs/node/pull/51909)

+   [[`9a1e01c4ce`](https://github.com/nodejs/node/commit/9a1e01c4ce)] - **(SEMVER-MINOR)** **util**: 在 util.styleText 中支持格式数组 (Marco Ippolito) [#52040](https://github.com/nodejs/node/pull/52040)

+   [[`7f2d61f82a`](https://github.com/nodejs/node/commit/7f2d61f82a)] - **(SEMVER-MINOR)** **v8**: 为内存泄漏回归测试实现 v8.queryObjects() (Joyee Cheung) [#51927](https://github.com/nodejs/node/pull/51927)

+   [[`d1d5da22e4`](https://github.com/nodejs/node/commit/d1d5da22e4)] - **(SEMVER-MINOR)** **vm**: 强化模块类型检查 (Chengzhong Wu) [#52162](https://github.com/nodejs/node/pull/52162)

### Semver-Patch Commits

+   [[`a760dadec3`](https://github.com/nodejs/node/commit/a760dadec3)] - **benchmark**: 添加 AbortSignal.abort 基准测试 (Raz Luvaton) [#52408](https://github.com/nodejs/node/pull/52408)

+   [[`47c934e464`](https://github.com/nodejs/node/commit/47c934e464)] - **benchmark**: 根据需要使用 spawn 和 taskset 进行 CPU 钉扎 (Ali Hassan) [#52253](https://github.com/nodejs/node/pull/52253)

+   [[`dde0cffb2e`](https://github.com/nodejs/node/commit/dde0cffb2e)] - **benchmark**: 添加 toNamespacedPath 基准测试 (Rafael Gonzaga) [#52236](https://github.com/nodejs/node/pull/52236)

+   [[`bda66ad711`](https://github.com/nodejs/node/commit/bda66ad711)] - **benchmark**: 添加 style-text 基准测试 (Rafael Gonzaga) [#52004](https://github.com/nodejs/node/pull/52004)

+   [[`21211a3fa9`](https://github.com/nodejs/node/commit/21211a3fa9)] - **buffer**: 改进 `btoa` 性能 (Yagiz Nizipli) [#52427](https://github.com/nodejs/node/pull/52427)

+   [[`6f504b71ac`](https://github.com/nodejs/node/commit/6f504b71ac)] - **buffer**: 使用 simdutf 实现 `atob` (Yagiz Nizipli) [#52381](https://github.com/nodejs/node/pull/52381)

+   [[`0ce7365856`](https://github.com/nodejs/node/commit/0ce7365856)] - **build**: 暂时禁用 ubsan (Rafael Gonzaga) [#52560](https://github.com/nodejs/node/pull/52560)

+   [[`4e278f0253`](https://github.com/nodejs/node/commit/4e278f0253)] - **build**: 加快某些 V8 文件的编译速度 (Michaël Zasso) [#52083](https://github.com/nodejs/node/pull/52083)

+   [[`ba06c5c509`](https://github.com/nodejs/node/commit/ba06c5c509)] - **build,tools**: 添加 test-ubsan ci (Rafael Gonzaga) [#46297](https://github.com/nodejs/node/pull/46297)

+   [[`562369f348`](https://github.com/nodejs/node/commit/562369f348)] - **child_process**: 使用内部 addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`8f61b658de`](https://github.com/nodejs/node/commit/8f61b658de)] - **crypto**: 废弃隐式截短的 GCM 标签 (Tobias Nießen) [#52345](https://github.com/nodejs/node/pull/52345)

+   [[`08609b5222`](https://github.com/nodejs/node/commit/08609b5222)] - **crypto**: 使 Uint8Array 的 timingSafeEqual 更快 (Tobias Nießen) [#52341](https://github.com/nodejs/node/pull/52341)

+   [[`9f939f5af7`](https://github.com/nodejs/node/commit/9f939f5af7)] - **crypto**: 在 Sign/Verify 原型中拒绝 Ed25519/Ed448 (Filip Skokan) [#52340](https://github.com/nodejs/node/pull/52340)

+   [[`2241e8c5b3`](https://github.com/nodejs/node/commit/2241e8c5b3)] - **crypto**: 在 subtle.sign 和 subtle.verify 中验证 RSA-PSS saltLength (Filip Skokan) [#52262](https://github.com/nodejs/node/pull/52262)

+   [[`6dd1c75f4a`](https://github.com/nodejs/node/commit/6dd1c75f4a)] - **crypto**: 修复 `crypto.hash` 中 `input` 的验证 (Antoine du Hamel) [#52070](https://github.com/nodejs/node/pull/52070)

+   [[`a1d48f4a26`](https://github.com/nodejs/node/commit/a1d48f4a26)] - **deps**: 将 simdutf 更新到 5.2.4 (Node.js GitHub Bot) [#52473](https://github.com/nodejs/node/pull/52473)

+   [[`08ff4a0c9d`](https://github.com/nodejs/node/commit/08ff4a0c9d)] - **deps**: 将 nghttp2 更新到 1.61.0 (Node.js GitHub Bot) [#52395](https://github.com/nodejs/node/pull/52395)

+   [[`cf629366b9`](https://github.com/nodejs/node/commit/cf629366b9)] - **deps**: 将 simdutf 更新到 5.2.3 (Yagiz Nizipli) [#52381](https://github.com/nodejs/node/pull/52381)

+   [[`ad86a12964`](https://github.com/nodejs/node/commit/ad86a12964)] - **deps**: 将 npm 升级到 10.5.1 (npm team) [#52351](https://github.com/nodejs/node/pull/52351)

+   [[`45cc32c9c6`](https://github.com/nodejs/node/commit/45cc32c9c6)] - **deps**: 将 c-ares 更新到 1.28.1 (Node.js GitHub Bot) [#52285](https://github.com/nodejs/node/pull/52285)

+   [[`38161c38d9`](https://github.com/nodejs/node/commit/38161c38d9)] - **deps**: 更新 zlib 至 1.3.0.1-motley-24c07df (Node.js GitHub 机器人) [#52199](https://github.com/nodejs/node/pull/52199)

+   [[`1264414700`](https://github.com/nodejs/node/commit/1264414700)] - **deps**: 更新 simdjson 至 3.8.0 (Node.js GitHub 机器人) [#52124](https://github.com/nodejs/node/pull/52124)

+   [[`f6996ee150`](https://github.com/nodejs/node/commit/f6996ee150)] - **deps**: V8: 回溯 c4be0a97f981 (Richard Lau) [#52183](https://github.com/nodejs/node/pull/52183)

+   [[`0d4bc4c40e`](https://github.com/nodejs/node/commit/0d4bc4c40e)] - **deps**: V8: 拣选 f8d5e576b814 (Richard Lau) [#52183](https://github.com/nodejs/node/pull/52183)

+   [[`70a05103c8`](https://github.com/nodejs/node/commit/70a05103c8)] - **deps**: 更新 zlib 至 1.3.0.1-motley-24342f6 (Node.js GitHub 机器人) [#52123](https://github.com/nodejs/node/pull/52123)

+   [[`4c3e9659ed`](https://github.com/nodejs/node/commit/4c3e9659ed)] - **deps**: 更新 corepack 至 0.26.0 (Node.js GitHub 机器人) [#52027](https://github.com/nodejs/node/pull/52027)

+   [[`0b4cdb4b42`](https://github.com/nodejs/node/commit/0b4cdb4b42)] - **deps**: 更新 ada 至 2.7.7 (Node.js GitHub 机器人) [#52028](https://github.com/nodejs/node/pull/52028)

+   [[`b241a1d0ae`](https://github.com/nodejs/node/commit/b241a1d0ae)] - **deps**: 更新 simdutf 至 4.0.9 (Node.js GitHub 机器人) [#51655](https://github.com/nodejs/node/pull/51655)

+   [[`36dcd399c0`](https://github.com/nodejs/node/commit/36dcd399c0)] - **deps**: 升级 libuv 至 1.48.0 (Santiago Gimeno) [#51697](https://github.com/nodejs/node/pull/51697)

+   [[`8cf313cd72`](https://github.com/nodejs/node/commit/8cf313cd72)] - **deps**: 更新 undici 至 6.6.0 (Node.js GitHub 机器人) [#51630](https://github.com/nodejs/node/pull/51630)

+   [[`dd4767f99f`](https://github.com/nodejs/node/commit/dd4767f99f)] - **deps**: 更新 undici 至 6.4.0 (Node.js GitHub 机器人) [#51527](https://github.com/nodejs/node/pull/51527)

+   [[`8362caa7d8`](https://github.com/nodejs/node/commit/8362caa7d8)] - **dgram**: 使用内部 addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`4f3cf4e89a`](https://github.com/nodejs/node/commit/4f3cf4e89a)] - **diagnostics_channel**: 提前退出跟踪通道的 trace 方法 (Stephen Belanger) [#51915](https://github.com/nodejs/node/pull/51915)

+   [[`204018bba6`](https://github.com/nodejs/node/commit/204018bba6)] - **doc**: 废弃 --experimental-policy (RafaelGSS) [#52602](https://github.com/nodejs/node/pull/52602)

+   [[`d32a914ac7`](https://github.com/nodejs/node/commit/d32a914ac7)] - **doc**: 在 BUILDING.md 中添加 lint-js-fix (jakecastelli) [#52290](https://github.com/nodejs/node/pull/52290)

+   [[`411503bacd`](https://github.com/nodejs/node/commit/411503bacd)] - **doc**: 移除 BUILDING.md 中对 Internet Explorer 的提及 (Rich Trott) [#52455](https://github.com/nodejs/node/pull/52455)

+   [[`e9ccf5aba2`](https://github.com/nodejs/node/commit/e9ccf5aba2)] - **doc**: 适应即将更严格的 .md 语法检查（Rich Trott）[#52454](https://github.com/nodejs/node/pull/52454)

+   [[`b4186ec2c1`](https://github.com/nodejs/node/commit/b4186ec2c1)] - **doc**: 将 Rafael 添加到管理员列表中（Rafael Gonzaga）[#52452](https://github.com/nodejs/node/pull/52452)

+   [[`7b01bfb2be`](https://github.com/nodejs/node/commit/7b01bfb2be)] - **doc**: 修正 C++ 风格指南中的命名约定（Mohammed Keyvanzadeh）[#52424](https://github.com/nodejs/node/pull/52424)

+   [[`c82f3c9e80`](https://github.com/nodejs/node/commit/c82f3c9e80)] - **doc**: 更新 `process.execArg` 示例以更有用（Jacob Smith）[#52412](https://github.com/nodejs/node/pull/52412)

+   [[`655b327a4d`](https://github.com/nodejs/node/commit/655b327a4d)] - **doc**: 强调 http(s).globalAgent 的默认设置（mathis-west-1）[#52392](https://github.com/nodejs/node/pull/52392)

+   [[`2c77be5488`](https://github.com/nodejs/node/commit/2c77be5488)] - **doc**: 更新 `build_with_cmake` 的位置（Emmanuel Ferdman）[#52356](https://github.com/nodejs/node/pull/52356)

+   [[`7dd514f2db`](https://github.com/nodejs/node/commit/7dd514f2db)] - **doc**: 保留 125 作为 Electron 31 的保留字段（Shelley Vohr）[#52379](https://github.com/nodejs/node/pull/52379)

+   [[`756acd0877`](https://github.com/nodejs/node/commit/756acd0877)] - **doc**: 使用一致的“index”复数形式（Rich Trott）[#52373](https://github.com/nodejs/node/pull/52373)

+   [[`ba07e4e5e6`](https://github.com/nodejs/node/commit/ba07e4e5e6)] - **doc**: 修正 cli.md 中的拼写错误（Daeyeon Jeong）[#52388](https://github.com/nodejs/node/pull/52388)

+   [[`461d9d665d`](https://github.com/nodejs/node/commit/461d9d665d)] - **doc**: 将 Rafael 添加到安全发布管理员中（Rafael Gonzaga）[#52354](https://github.com/nodejs/node/pull/52354)

+   [[`d0c364a844`](https://github.com/nodejs/node/commit/d0c364a844)] - **doc**: 文档 events.on 的缺失选项（Chemi Atlow）[#52080](https://github.com/nodejs/node/pull/52080)

+   [[`a63261cf2c`](https://github.com/nodejs/node/commit/a63261cf2c)] - **doc**: 添加缺失的空格（Augustin Mauroy）[#52360](https://github.com/nodejs/node/pull/52360)

+   [[`dd711d221a`](https://github.com/nodejs/node/commit/dd711d221a)] - **doc**: 添加关于 vcpkg 导致 Windows 构建失败的提示（Cong Zhang）[#52181](https://github.com/nodejs/node/pull/52181)

+   [[`4df34cf6dd`](https://github.com/nodejs/node/commit/4df34cf6dd)] - **doc**: 将“below”替换为“following”（Rich Trott）[#52315](https://github.com/nodejs/node/pull/52315)

+   [[`d9aa33fdbf`](https://github.com/nodejs/node/commit/d9aa33fdbf)] - **doc**: 修正电子邮件模式，使用 `<<` 而不是单个 `<`（Raz Luvaton）[#52284](https://github.com/nodejs/node/pull/52284)

+   [[`903f28e684`](https://github.com/nodejs/node/commit/903f28e684)] - **doc**: 更新发布 GPG 密钥服务器（marco-ippolito）[#52257](https://github.com/nodejs/node/pull/52257)

+   [[`fd55458770`](https://github.com/nodejs/node/commit/fd55458770)] - **doc**: 为 marco-ippolito 添加发布密钥 (marco-ippolito) [#52257](https://github.com/nodejs/node/pull/52257)

+   [[`27493a1dd7`](https://github.com/nodejs/node/commit/27493a1dd7)] - **doc**: 修复 HTML 版本中箭头的垂直对齐问题 (Akash Yeole) [#52193](https://github.com/nodejs/node/pull/52193)

+   [[`af48641993`](https://github.com/nodejs/node/commit/af48641993)] - **doc**: 将 TSC 成员从常规移至名誉成员 (Michael Dawson) [#52209](https://github.com/nodejs/node/pull/52209)

+   [[`fa13ed6d79`](https://github.com/nodejs/node/commit/fa13ed6d79)] - **doc**: 添加解释待办测试的部分 (Colin Ihrig) [#52204](https://github.com/nodejs/node/pull/52204)

+   [[`312ebd97c2`](https://github.com/nodejs/node/commit/312ebd97c2)] - **doc**: 编辑 `ChildProcess` `'message'` 事件文档 (theanarkh) [#52154](https://github.com/nodejs/node/pull/52154)

+   [[`f1635f442f`](https://github.com/nodejs/node/commit/f1635f442f)] - **doc**: 引用 test_runner 的 glob 参数 (Fabian Meyer) [#52201](https://github.com/nodejs/node/pull/52201)

+   [[`fc029181df`](https://github.com/nodejs/node/commit/fc029181df)] - **doc**: 在加速部分添加 mold 链接 (Cong Zhang) [#52179](https://github.com/nodejs/node/pull/52179)

+   [[`8bd3cb2f8c`](https://github.com/nodejs/node/commit/8bd3cb2f8c)] - **doc**: 修正 http 事件顺序 (wh0) [#51464](https://github.com/nodejs/node/pull/51464)

+   [[`a7f170e45a`](https://github.com/nodejs/node/commit/a7f170e45a)] - **doc**: 将 gabrielschulhof 移至 TSC 名誉成员 (Gabriel Schulhof) [#52192](https://github.com/nodejs/node/pull/52192)

+   [[`305375ac16`](https://github.com/nodejs/node/commit/305375ac16)] - **doc**: 修复 `--env-file` 文档中用于定义值的有效引号 (Gabriel Bota) [#52157](https://github.com/nodejs/node/pull/52157)

+   [[`3fcaf7b900`](https://github.com/nodejs/node/commit/3fcaf7b900)] - **doc**: 澄清 NODE_OPTIONS 中的支持内容 (Michael Dawson) [#52076](https://github.com/nodejs/node/pull/52076)

+   [[`4fe87357f3`](https://github.com/nodejs/node/commit/4fe87357f3)] - **doc**: 修正 maintaining-dependencies.md 中的拼写错误 (RoboSchmied) [#52160](https://github.com/nodejs/node/pull/52160)

+   [[`f1949ac1ae`](https://github.com/nodejs/node/commit/f1949ac1ae)] - **doc**: 添加包含模块语法规范 (Geoffrey Booth) [#52059](https://github.com/nodejs/node/pull/52059)

+   [[`707155424b`](https://github.com/nodejs/node/commit/707155424b)] - **doc**: 优化 Unix 抽象套接字的文档 (theanarkh) [#52043](https://github.com/nodejs/node/pull/52043)

+   [[`8a191e4e6a`](https://github.com/nodejs/node/commit/8a191e4e6a)] - **doc**: 更新 pnpm 链接 (Superchupu) [#52113](https://github.com/nodejs/node/pull/52113)

+   [[`454d0806a1`](https://github.com/nodejs/node/commit/454d0806a1)] - **doc**: 从 crypto 中移除能力主义语言 (Jamie King) [#52063](https://github.com/nodejs/node/pull/52063)

+   [[`dafe004703`](https://github.com/nodejs/node/commit/dafe004703)] - **doc**: 更新协作者电子邮件（Ruy Adorno）[#52088](https://github.com/nodejs/node/pull/52088)

+   [[`8824adb031`](https://github.com/nodejs/node/commit/8824adb031)] - **doc**: 指出移除 npm 不是目标（Geoffrey Booth）[#51951](https://github.com/nodejs/node/pull/51951)

+   [[`b360532f1a`](https://github.com/nodejs/node/commit/b360532f1a)] - **doc**: 在 RafaelGSS 的管理员列表中提到 NodeSource（Rafael Gonzaga）[#52057](https://github.com/nodejs/node/pull/52057)

+   [[`57d2e4881c`](https://github.com/nodejs/node/commit/57d2e4881c)] - **doc**: 从 crypto.hash() 数据参数类型中移除 ArrayBuffer（fengmk2）[#52069](https://github.com/nodejs/node/pull/52069)

+   [[`e11c1d2315`](https://github.com/nodejs/node/commit/e11c1d2315)] - **doc**: 添加一些常用标签到前端（Michael Dawson）[#52006](https://github.com/nodejs/node/pull/52006)

+   [[`8f9f5db1e8`](https://github.com/nodejs/node/commit/8f9f5db1e8)] - **doc**: 记录 `const c2 = vm.createContext(c1); c1 === c2` 为真（Daniel Kaplan）[#51960](https://github.com/nodejs/node/pull/51960)

+   [[`d78a565713`](https://github.com/nodejs/node/commit/d78a565713)] - **doc**: 澄清内容管理问题的定义（Antoine du Hamel）[#51990](https://github.com/nodejs/node/pull/51990)

+   [[`4cac07c931`](https://github.com/nodejs/node/commit/4cac07c931)] - **doc**: 将 Hemanth HM 添加到 v21.7.0 变更日志中（Rafael Gonzaga）[#52008](https://github.com/nodejs/node/pull/52008)

+   [[`73025c4dec`](https://github.com/nodejs/node/commit/73025c4dec)] - **doc**: 添加 UlisesGascon 作为协作者（Ulises Gascón）[#51991](https://github.com/nodejs/node/pull/51991)

+   [[`999c6b34fb`](https://github.com/nodejs/node/commit/999c6b34fb)] - **doc**: 为 CLI 选项进行测试（Aras Abbasi）[#51623](https://github.com/nodejs/node/pull/51623)

+   [[`edd6190836`](https://github.com/nodejs/node/commit/edd6190836)] - **doc**: 废弃 hmac 公共构造函数（Marco Ippolito）[#51881](https://github.com/nodejs/node/pull/51881)

+   [[`25c79f3331`](https://github.com/nodejs/node/commit/25c79f3331)] - **esm**: 放弃对导入断言的支持（Nicolò Ribaudo）[#52104](https://github.com/nodejs/node/pull/52104)

+   [[`d619aab575`](https://github.com/nodejs/node/commit/d619aab575)] - **events**: 为了一致性重命名高水位线和低水位线（Chemi Atlow）[#52080](https://github.com/nodejs/node/pull/52080)

+   [[`e263946c2e`](https://github.com/nodejs/node/commit/e263946c2e)] - **events**: 为安全内部使用提取 addAbortListener（Chemi Atlow）[#52081](https://github.com/nodejs/node/pull/52081)

+   [[`40ef2da8d6`](https://github.com/nodejs/node/commit/40ef2da8d6)] - **events**: 在 `on` 中从信号中移除 abort 监听器（Neal Beeken）[#51091](https://github.com/nodejs/node/pull/51091)

+   [[`61e5de1268`](https://github.com/nodejs/node/commit/61e5de1268)] - **fs**: 重构 maybeCallback 函数（Yagiz Nizipli）[#52129](https://github.com/nodejs/node/pull/52129)

+   [[`39f1b899cd`](https://github.com/nodejs/node/commit/39f1b899cd)] - **fs**: 修复 readFileSync utf8 快速路径中的边缘情况 (Richard Lau) [#52101](https://github.com/nodejs/node/pull/52101)

+   [[`639c096004`](https://github.com/nodejs/node/commit/639c096004)] - **fs**: 在 `fchown` 上从 cpp 验证 fd (Yagiz Nizipli) [#52051](https://github.com/nodejs/node/pull/52051)

+   [[`9ac1fe05d7`](https://github.com/nodejs/node/commit/9ac1fe05d7)] - **fs**: 在 `close` 上从 cpp 验证 fd (Yagiz Nizipli) [#52051](https://github.com/nodejs/node/pull/52051)

+   [[`3ec20f25df`](https://github.com/nodejs/node/commit/3ec20f25df)] - **fs**: 从 cpp 验证文件模式 (Yagiz Nizipli) [#52050](https://github.com/nodejs/node/pull/52050)

+   [[`8c0b723ccb`](https://github.com/nodejs/node/commit/8c0b723ccb)] - **fs,permission**: 使缓冲区处理一致 (Tobias Nießen) [#52348](https://github.com/nodejs/node/pull/52348)

+   [[`3fc8d2200e`](https://github.com/nodejs/node/commit/3fc8d2200e)] - **http2**: 修复 h2-over-h2 连接代理 (Tim Perry) [#52368](https://github.com/nodejs/node/pull/52368)

+   [[`b9d8a14a03`](https://github.com/nodejs/node/commit/b9d8a14a03)] - **http2**: 使用内部 addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`818c10e86d`](https://github.com/nodejs/node/commit/818c10e86d)] - **lib**: 改进 `AbortSignal` 创建的性能 (Raz Luvaton) [#52408](https://github.com/nodejs/node/pull/52408)

+   [[`3f5ff8dc20`](https://github.com/nodejs/node/commit/3f5ff8dc20)] - **lib**: 在未传递文件时添加适当的错误消息 (Thomas Mauran) [#52225](https://github.com/nodejs/node/pull/52225)

+   [[`0a252c23d9`](https://github.com/nodejs/node/commit/0a252c23d9)] - **lib**: 修复 _refreshLine 的类型错误 (Jackson Tian) [#52133](https://github.com/nodejs/node/pull/52133)

+   [[`14de082ab4`](https://github.com/nodejs/node/commit/14de082ab4)] - **lib**: 当调用 listen 两次时仅发出一次监听事件 (theanarkh) [#52119](https://github.com/nodejs/node/pull/52119)

+   [[`4e9ce7c035`](https://github.com/nodejs/node/commit/4e9ce7c035)] - **lib**: 确保在 http 服务器中清除旧定时器 (theanarkh) [#52118](https://github.com/nodejs/node/pull/52118)

+   [[`20525f14b9`](https://github.com/nodejs/node/commit/20525f14b9)] - **lib**: 在集群工作者中使用 handle 修复 listen (theanarkh) [#52056](https://github.com/nodejs/node/pull/52056)

+   [[`8df54481f4`](https://github.com/nodejs/node/commit/8df54481f4)] - **meta**: 将 actions/download-artifact 从 4.1.3 升级到 4.1.4 (dependabot[bot]) [#52314](https://github.com/nodejs/node/pull/52314)

+   [[`bcc102147a`](https://github.com/nodejs/node/commit/bcc102147a)] - **meta**: 将 rtCamp/action-slack-notify 从 2.2.1 升级到 2.3.0 (dependabot[bot]) [#52313](https://github.com/nodejs/node/pull/52313)

+   [[`4e7e0ef9c3`](https://github.com/nodejs/node/commit/4e7e0ef9c3)] - **meta**: 将 github/codeql-action 从 3.24.6 升级到 3.24.9 (dependabot[bot]) [#52312](https://github.com/nodejs/node/pull/52312)

+   [[`14a39881b8`](https://github.com/nodejs/node/commit/14a39881b8)] - **meta**: 将 actions/cache 从 4.0.1 升级至 4.0.2 (dependabot[bot]) [#52311](https://github.com/nodejs/node/pull/52311)

+   [[`2f8f90dadb`](https://github.com/nodejs/node/commit/2f8f90dadb)] - **meta**: 将 actions/setup-python 从 5.0.0 升级至 5.1.0 (dependabot[bot]) [#52310](https://github.com/nodejs/node/pull/52310)

+   [[`95efdaf01a`](https://github.com/nodejs/node/commit/95efdaf01a)] - **meta**: 将 codecov/codecov-action 从 4.1.0 升级至 4.1.1 (dependabot[bot]) [#52308](https://github.com/nodejs/node/pull/52308)

+   [[`24c1a8e739`](https://github.com/nodejs/node/commit/24c1a8e739)] - **meta**: 将一名或多名协作者移至名誉退休状态 (Node.js GitHub Bot) [#52300](https://github.com/nodejs/node/pull/52300)

+   [[`60dcfad91e`](https://github.com/nodejs/node/commit/60dcfad91e)] - **meta**: 将 Codecov 上传令牌传递给 codecov action (Michaël Zasso) [#51982](https://github.com/nodejs/node/pull/51982)

+   [[`db1746182b`](https://github.com/nodejs/node/commit/db1746182b)] - **module**: 禁止在循环中从 require(esm) 导致的 CJS <-> ESM 边缘 (Joyee Cheung) [#52264](https://github.com/nodejs/node/pull/52264)

+   [[`d6b57f6629`](https://github.com/nodejs/node/commit/d6b57f6629)] - **module**: 将 SourceTextModule 编译集中于内置加载器 (Joyee Cheung) [#52291](https://github.com/nodejs/node/pull/52291)

+   [[`f4a0a3b04b`](https://github.com/nodejs/node/commit/f4a0a3b04b)] - **module**: 在无类型包中检测时发出警告 (Geoffrey Booth) [#52168](https://github.com/nodejs/node/pull/52168)

+   [[`8bc745944e`](https://github.com/nodejs/node/commit/8bc745944e)] - **module**: 消除检测 CJS 入口性能开销 (Geoffrey Booth) [#52093](https://github.com/nodejs/node/pull/52093)

+   [[`63d04d4d80`](https://github.com/nodejs/node/commit/63d04d4d80)] - **module**: 修复 detect-module 对仅 CJS 错误不重试为 esm 的问题 (Geoffrey Booth) [#52024](https://github.com/nodejs/node/pull/52024)

+   [[`575ced8139`](https://github.com/nodejs/node/commit/575ced8139)] - **module**: 打印未决顶级 await 在入口点的位置 (Joyee Cheung) [#51999](https://github.com/nodejs/node/pull/51999)

+   [[`075c95f61f`](https://github.com/nodejs/node/commit/075c95f61f)] - **module**: 重构 ESM 加载器初始化和入口点处理 (Joyee Cheung) [#51999](https://github.com/nodejs/node/pull/51999)

+   [[`45f0dd0192`](https://github.com/nodejs/node/commit/45f0dd0192)] - **module,win**: 修复长路径解析问题 (Stefan Stojanovic) [#51097](https://github.com/nodejs/node/pull/51097)

+   [[`d89fc73d45`](https://github.com/nodejs/node/commit/d89fc73d45)] - **net**: 使用内部 addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`f0e6acde2d`](https://github.com/nodejs/node/commit/f0e6acde2d)] - **node-api**: 再次让 tsfn 接受 napi_finalize (Gabriel Schulhof) [#51801](https://github.com/nodejs/node/pull/51801)

+   [[`ff93f3e1a8`](https://github.com/nodejs/node/commit/ff93f3e1a8)] - **readline**: 使用内部的 addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`4a6ca7a1d4`](https://github.com/nodejs/node/commit/4a6ca7a1d4)] - **src**: 移除错误的 CVE-2024-27980 回滚选项 (Tobias Nießen) [#52543](https://github.com/nodejs/node/pull/52543)

+   [[`64b67779f7`](https://github.com/nodejs/node/commit/64b67779f7)] - **src**: 禁止直接生成 .bat 和 .cmd 文件 (Ben Noordhuis) [nodejs-private/node-private#560](https://github.com/nodejs-private/node-private/pull/560)

+   [[`9ef724bc81`](https://github.com/nodejs/node/commit/9ef724bc81)] - **src**: 更新 node_revert.h 中的分支名称 (Tobias Nießen) [#52390](https://github.com/nodejs/node/pull/52390)

+   [[`ec1550407b`](https://github.com/nodejs/node/commit/ec1550407b)] - **src**: 停止使用 `v8::BackingStore::Reallocate` (Michaël Zasso) [#52292](https://github.com/nodejs/node/pull/52292)

+   [[`681b0a3df3`](https://github.com/nodejs/node/commit/681b0a3df3)] - **src**: 处理 module_wrap.cc 中的 coverity 警告 (Michael Dawson) [#52143](https://github.com/nodejs/node/pull/52143)

+   [[`04319228e0`](https://github.com/nodejs/node/commit/04319228e0)] - **src**: 修复 coverity 报告的使用后移动问题 (Michael Dawson) [#52141](https://github.com/nodejs/node/pull/52141)

+   [[`0eb2b727f6`](https://github.com/nodejs/node/commit/0eb2b727f6)] - **src**: 始终从 process.constrainedMemory() 返回一个数字 (Chengzhong Wu) [#52039](https://github.com/nodejs/node/pull/52039)

+   [[`bec9b5fccc`](https://github.com/nodejs/node/commit/bec9b5fccc)] - **src**: 使用专用的例程为内置的 CJS 加载器编译函数 (Joyee Cheung) [#52016](https://github.com/nodejs/node/pull/52016)

+   [[`1f193165b9`](https://github.com/nodejs/node/commit/1f193165b9)] - **src**: 修复在 Blob[De]serializer 中读取空字符串视图的问题 (Joyee Cheung) [#52000](https://github.com/nodejs/node/pull/52000)

+   [[`fb356b3305`](https://github.com/nodejs/node/commit/fb356b3305)] - **src**: 重构出 FormatErrorMessage 以进行错误格式化 (Joyee Cheung) [#51999](https://github.com/nodejs/node/pull/51999)

+   [[`1a8ae9d6c0`](https://github.com/nodejs/node/commit/1a8ae9d6c0)] - **src**: 在 Blob 中使用基于回调的数组迭代 (Joyee Cheung) [#51758](https://github.com/nodejs/node/pull/51758)

+   [[`5cd2ec8bd5`](https://github.com/nodejs/node/commit/5cd2ec8bd5)] - **src**: 使用新的基于回调的 API 实现 v8 数组迭代 (Joyee Cheung) [#51758](https://github.com/nodejs/node/pull/51758)

+   [[`89a26b451e`](https://github.com/nodejs/node/commit/89a26b451e)] - **src**: 修复 node_version.h (Joyee Cheung) [#50375](https://github.com/nodejs/node/pull/50375)

+   [[`c02de658a1`](https://github.com/nodejs/node/commit/c02de658a1)] - **stream**: 让 Duplex 从 Writable 继承 destroy 方法 (Luigi Pinca) [#52318](https://github.com/nodejs/node/pull/52318)

+   [[`63391e749d`](https://github.com/nodejs/node/commit/63391e749d)] - **stream**: add `new` when constructing `ERR_MULTIPLE_CALLBACK` (haze) [#52110](https://github.com/nodejs/node/pull/52110)

+   [[`a9528e87b9`](https://github.com/nodejs/node/commit/a9528e87b9)] - **stream**: use internal addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`ee4fa77624`](https://github.com/nodejs/node/commit/ee4fa77624)] - **test**: fix watch test with require not testing pid (Raz Luvaton) [#52353](https://github.com/nodejs/node/pull/52353)

+   [[`05cb16dc1a`](https://github.com/nodejs/node/commit/05cb16dc1a)] - **test**: simplify ASan build checks (Michaël Zasso) [#52430](https://github.com/nodejs/node/pull/52430)

+   [[`eb53121b77`](https://github.com/nodejs/node/commit/eb53121b77)] - **test**: fix Windows compiler warnings in overlapped-checker (Michaël Zasso) [#52405](https://github.com/nodejs/node/pull/52405)

+   [[`7dfa4750af`](https://github.com/nodejs/node/commit/7dfa4750af)] - **test**: add test for skip+todo combinations (Colin Ihrig) [#52204](https://github.com/nodejs/node/pull/52204)

+   [[`5905596719`](https://github.com/nodejs/node/commit/5905596719)] - **test**: fix incorrect test fixture (Colin Ihrig) [#52185](https://github.com/nodejs/node/pull/52185)

+   [[`bae14b7914`](https://github.com/nodejs/node/commit/bae14b7914)] - **test**: do not set concurrency on parallelized runs (Antoine du Hamel) [#52177](https://github.com/nodejs/node/pull/52177)

+   [[`0b676736a0`](https://github.com/nodejs/node/commit/0b676736a0)] - **test**: add missing cctest/test_path.cc (Yagiz Nizipli) [#52148](https://github.com/nodejs/node/pull/52148)

+   [[`c714cda9a7`](https://github.com/nodejs/node/commit/c714cda9a7)] - **test**: add `spawnSyncAndAssert` util (Antoine du Hamel) [#52132](https://github.com/nodejs/node/pull/52132)

+   [[`978d5a26c9`](https://github.com/nodejs/node/commit/978d5a26c9)] - **test**: reduce flakiness of test-runner-output.mjs (Colin Ihrig) [#52146](https://github.com/nodejs/node/pull/52146)

+   [[`afaf889775`](https://github.com/nodejs/node/commit/afaf889775)] - **test**: skip test for dynamically linked OpenSSL (Richard Lau) [#52542](https://github.com/nodejs/node/pull/52542)

+   [[`be75821a12`](https://github.com/nodejs/node/commit/be75821a12)] - **test**: add test for using `--print` with promises (Antoine du Hamel) [#52137](https://github.com/nodejs/node/pull/52137)

+   [[`4e109e5958`](https://github.com/nodejs/node/commit/4e109e5958)] - **test**: un-set test-emit-after-on-destroyed as flaky (Abdirahim Musse) [#51995](https://github.com/nodejs/node/pull/51995)

+   [[`3f8cc88009`](https://github.com/nodejs/node/commit/3f8cc88009)] - **test_runner**: fix clearing final timeout in own callback (Ben Richeson) [#52332](https://github.com/nodejs/node/pull/52332)

+   [[`52f8dcfccc`](https://github.com/nodejs/node/commit/52f8dcfccc)] - **test_runner**: make end of work check stricter (Colin Ihrig) [#52326](https://github.com/nodejs/node/pull/52326)

+   [[`433bd1b04d`](https://github.com/nodejs/node/commit/433bd1b04d)] - **test_runner**: 修复递归运行问题（Moshe Atlow）[#52322](https://github.com/nodejs/node/pull/52322)

+   [[`e57992ffb2`](https://github.com/nodejs/node/commit/e57992ffb2)] - **test_runner**: 在规范报告器中没有错误时隐藏换行（Moshe Atlow）[#52297](https://github.com/nodejs/node/pull/52297)

+   [[`ac9e5e7527`](https://github.com/nodejs/node/commit/ac9e5e7527)] - **test_runner**: 改进 describe.only 行为（Moshe Atlow）[#52296](https://github.com/nodejs/node/pull/52296)

+   [[`2c024cd24d`](https://github.com/nodejs/node/commit/2c024cd24d)] - **test_runner**: 禁用 TestsStream 上的 highWatermark（Colin Ihrig）[#52287](https://github.com/nodejs/node/pull/52287)

+   [[`7c02486f1f`](https://github.com/nodejs/node/commit/7c02486f1f)] - **test_runner**: 按正确顺序运行每个 afterEach 钩子（Colin Ihrig）[#52239](https://github.com/nodejs/node/pull/52239)

+   [[`6af4049810`](https://github.com/nodejs/node/commit/6af4049810)] - **test_runner**: 简化测试结束时间跟踪（Colin Ihrig）[#52182](https://github.com/nodejs/node/pull/52182)

+   [[`878047be0b`](https://github.com/nodejs/node/commit/878047be0b)] - **test_runner**: 简化测试启动时间跟踪（Colin Ihrig）[#52182](https://github.com/nodejs/node/pull/52182)

+   [[`4648c83dbc`](https://github.com/nodejs/node/commit/4648c83dbc)] - **test_runner**: 每个测试不再等待相同的承诺（Colin Ihrig）[#52185](https://github.com/nodejs/node/pull/52185)

+   [[`f9755f6f79`](https://github.com/nodejs/node/commit/f9755f6f79)] - **test_runner**: 在观察模式耗尽时发出诊断（Moshe Atlow）[#52130](https://github.com/nodejs/node/pull/52130)

+   [[`4ba9f45d99`](https://github.com/nodejs/node/commit/4ba9f45d99)] - **test_runner**: 运行套件时忽略待办标记（Colin Ihrig）[#52117](https://github.com/nodejs/node/pull/52117)

+   [[`6f4d6011ea`](https://github.com/nodejs/node/commit/6f4d6011ea)] - **test_runner**: 跳过已跳过测试的每个钩子（Colin Ihrig）[#52115](https://github.com/nodejs/node/pull/52115)

+   [[`05db979c01`](https://github.com/nodejs/node/commit/05db979c01)] - **test_runner**: 在微任务中运行顶层测试（Colin Ihrig）[#52092](https://github.com/nodejs/node/pull/52092)

+   [[`97b2c5344d`](https://github.com/nodejs/node/commit/97b2c5344d)] - **test_runner**: 移除多余的报告调用（Colin Ihrig）[#52089](https://github.com/nodejs/node/pull/52089)

+   [[`780d030bdf`](https://github.com/nodejs/node/commit/780d030bdf)] - **test_runner**: 使用内部 addAbortListener（Chemi Atlow）[#52081](https://github.com/nodejs/node/pull/52081)

+   [[`814fa1ae74`](https://github.com/nodejs/node/commit/814fa1ae74)] - **test_runner**: 报告覆盖率时使用源映射（Moshe Atlow）[#52060](https://github.com/nodejs/node/pull/52060)

+   [[`3c5764a0e2`](https://github.com/nodejs/node/commit/3c5764a0e2)] - **test_runner**: 处理未定义的测试位置（Colin Ihrig）[#52036](https://github.com/nodejs/node/pull/52036)

+   [[`328642bbb9`](https://github.com/nodejs/node/commit/328642bbb9)] - **test_runner**: 使用路径来定位测试位置（Colin Ihrig）[#52010](https://github.com/nodejs/node/pull/52010)

+   [[`6d625fe616`](https://github.com/nodejs/node/commit/6d625fe616)] - **test_runner**: 支持源映射的测试位置（Colin Ihrig）[#52010](https://github.com/nodejs/node/pull/52010)

+   [[`592c6907bf`](https://github.com/nodejs/node/commit/592c6907bf)] - **test_runner**: 避免覆盖根起始时间（Colin Ihrig）[#52020](https://github.com/nodejs/node/pull/52020)

+   [[`29b231763e`](https://github.com/nodejs/node/commit/29b231763e)] - **test_runner**: 在异步错误时中止未完成的测试（Colin Ihrig）[#51996](https://github.com/nodejs/node/pull/51996)

+   [[`5d13419dbd`](https://github.com/nodejs/node/commit/5d13419dbd)] - **test_runner**: 如果测试已启动，则立即运行 before hook（Moshe Atlow）[#52003](https://github.com/nodejs/node/pull/52003)

+   [[`8451990668`](https://github.com/nodejs/node/commit/8451990668)] - **test_runner**: 添加对 null 和日期值输出的支持（Malthe Borch）[#51920](https://github.com/nodejs/node/pull/51920)

+   [[`423ad47e0f`](https://github.com/nodejs/node/commit/423ad47e0f)] - **tools**: 将不活动限制更改为 12 个月（Yagiz Nizipli）[#52425](https://github.com/nodejs/node/pull/52425)

+   [[`0d1e64f64c`](https://github.com/nodejs/node/commit/0d1e64f64c)] - **tools**: 更新过期的机器人消息（Wes Todd）[#52423](https://github.com/nodejs/node/pull/52423)

+   [[`5bae73df90`](https://github.com/nodejs/node/commit/5bae73df90)] - **tools**: 更新 lint-md-dependencies 到 rollup@4.14.0（Node.js GitHub Bot）[#52398](https://github.com/nodejs/node/pull/52398)

+   [[`468cb99ba4`](https://github.com/nodejs/node/commit/468cb99ba4)] - **tools**: 更新 Ruff 到 v0.3.4（Michaël Zasso）[#52302](https://github.com/nodejs/node/pull/52302)

+   [[`67b9dda003`](https://github.com/nodejs/node/commit/67b9dda003)] - **tools**: 在 ubuntu-latest 上运行 test-ubsan（Michaël Zasso）[#52375](https://github.com/nodejs/node/pull/52375)

+   [[`f1f32d89e0`](https://github.com/nodejs/node/commit/f1f32d89e0)] - **tools**: 更新 lint-md-dependencies 到 rollup@4.13.2（Node.js GitHub Bot）[#52286](https://github.com/nodejs/node/pull/52286)

+   [[`d7aa8fc9da`](https://github.com/nodejs/node/commit/d7aa8fc9da)] - ***撤销*** "**tools**: 仅在源更改时运行 `build-windows` 工作流"（Michaël Zasso）[#52320](https://github.com/nodejs/node/pull/52320)

+   [[`a3b1fc3f27`](https://github.com/nodejs/node/commit/a3b1fc3f27)] - **tools**: 在 GitHub Actions 工作流中使用 Python 3.12（Michaël Zasso）[#52301](https://github.com/nodejs/node/pull/52301)

+   [[`021cf91208`](https://github.com/nodejs/node/commit/021cf91208)] - **tools**: 允许本地更新 llhttp（Paolo Insogna）[#52085](https://github.com/nodejs/node/pull/52085)

+   [[`4d8602046e`](https://github.com/nodejs/node/commit/4d8602046e)] - **tools**: 在 Windows 上安装 npm PowerShell 脚本 (Luke Karrys) [#52009](https://github.com/nodejs/node/pull/52009)

+   [[`081319d762`](https://github.com/nodejs/node/commit/081319d762)] - **tools**: 更新 lint-md-dependencies 至 rollup@4.13.0 (Node.js GitHub Bot) [#52122](https://github.com/nodejs/node/pull/52122)

+   [[`c43a944231`](https://github.com/nodejs/node/commit/c43a944231)] - **tools**: 修复 js2c.cc 中 coverity 报告的错误 (Michael Dawson) [#52142](https://github.com/nodejs/node/pull/52142)

+   [[`f05b241f07`](https://github.com/nodejs/node/commit/f05b241f07)] - **tools**: 同步 ubsan 工作流程至 asan (Michaël Zasso) [#52152](https://github.com/nodejs/node/pull/52152)

+   [[`a21b15a14e`](https://github.com/nodejs/node/commit/a21b15a14e)] - **tools**: 更新 github_reporter 至 1.7.0 (Node.js GitHub Bot) [#52121](https://github.com/nodejs/node/pull/52121)

+   [[`d60a871db2`](https://github.com/nodejs/node/commit/d60a871db2)] - **tools**: 删除 gyp-next 的 .github 文件夹 (Marco Ippolito) [#52064](https://github.com/nodejs/node/pull/52064)

+   [[`6ad5353764`](https://github.com/nodejs/node/commit/6ad5353764)] - **tools**: 更新 gyp-next 至 0.16.2 (Node.js GitHub Bot) [#52062](https://github.com/nodejs/node/pull/52062)

+   [[`dab85bdc06`](https://github.com/nodejs/node/commit/dab85bdc06)] - **tools**: 为 FreeBSD 安装 manpage 到 share/man (Po-Chuan Hsieh) [#51791](https://github.com/nodejs/node/pull/51791)

+   [[`cde37e7b63`](https://github.com/nodejs/node/commit/cde37e7b63)] - **tools**: 自动化更新 gyp-next (Marco Ippolito) [#52014](https://github.com/nodejs/node/pull/52014)

+   [[`925a464cb8`](https://github.com/nodejs/node/commit/925a464cb8)] - **url**: 移除 URLSearchParams 中的 #context (Matt Cowley) [#51520](https://github.com/nodejs/node/pull/51520)

+   [[`893e2cf22b`](https://github.com/nodejs/node/commit/893e2cf22b)] - **watch**: 修复未传递给监视进程的某些 node 参数 (Raz Luvaton) [#52358](https://github.com/nodejs/node/pull/52358)

+   [[`fec7e505fc`](https://github.com/nodejs/node/commit/fec7e505fc)] - **watch**: 使用内部 addAbortListener (Chemi Atlow) [#52081](https://github.com/nodejs/node/pull/52081)

+   [[`4f68c7c1c9`](https://github.com/nodejs/node/commit/4f68c7c1c9)] - **watch**: 标记为稳定 (Moshe Atlow) [#52074](https://github.com/nodejs/node/pull/52074)

+   [[`257f32296d`](https://github.com/nodejs/node/commit/257f32296d)] - **watch**: 批量文件重启 (Moshe Atlow) [#51992](https://github.com/nodejs/node/pull/51992)
