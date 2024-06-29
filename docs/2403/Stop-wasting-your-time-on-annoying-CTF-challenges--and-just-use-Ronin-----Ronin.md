<!--yml

类别: 未分类

日期: 2024-05-27 15:00:29

-->

# 放弃浪费时间在恼人的 CTF 挑战上（直接使用 Ronin）！| Ronin

> 来源：[https://ronin-rb.dev/blog/2024/03/15/stop-wasting-your-time-on-annoying-ctf-challenges.html](https://ronin-rb.dev/blog/2024/03/15/stop-wasting-your-time-on-annoying-ctf-challenges.html)

前几天我在 [nahamsec](https://nahamsec.com/) 的 Discord 服务器上发现有人在为解码加密字符串的 CTF 挑战而苦苦挣扎。他们已经尝试过 [CyberChef](https://cyberchef.org/)，但没有成功。其他用户鼓励他们“尽力而为”。

他们试图解密以下字符串：

```
pai1nYPe3tLR1IOH1YKH14XegtCDgtSH1oKAgNGF34eH19bSmw== 
```

显然这看起来是 base64 编码的，但如果你对字符串进行 base64 解码，你会得到二进制数据。用户表示挑战提示该字符串是经过 XOR 加密的，而明文的形式为 `CNS{md5hash}`。

在 `ronin irb` 控制台上，我能够轻松地用 *一行 Ruby 代码* 解决这个问题：

```
Chars.ascii.map { |key| "pai1nYPe3tLR1IOH1YKH14XegtCDgtSH1oKAgNGF34eH19bSmw==".
base64_decode.xor(key) }.find { |text| text.start_with?('CNS{') } 
```

从 CTF 挑战可能使用单个 ASCII 字符的 XOR 加密密钥的假设开始，我们只需使用 ASCII 字符集中的每个可能字符来暴力解密 base64 解码后的字符串，然后找到以 `CNS{` 开头的解密字符串。

运行上述代码给了我们答案：

```
CNS{e88472ea3da1c8d6ed2a0dff7c9aa104} 
```

就是这样了！

你不必“尽力而为”，只需使用 [Ronin](/install/)！

如果你对 Ronin 感兴趣或者喜欢我们的工作，请考虑捐赠给 Ronin

[GitHub](https://github.com/sponsors/ronin-rb)

,

[Patreon](https://patreon.com/roninrb)

，或者

[Open Collective](https://opencollective.com/ronin-rb)

所以我们可以继续构建高质量的免费开源安全工具和 Ruby 库。
