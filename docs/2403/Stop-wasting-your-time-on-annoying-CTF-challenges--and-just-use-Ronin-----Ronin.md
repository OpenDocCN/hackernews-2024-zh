<!--yml
category: 未分类
date: 2024-05-27 15:00:29
-->

# Stop wasting your time on annoying CTF challenges (and just use Ronin)! | Ronin

> 来源：[https://ronin-rb.dev/blog/2024/03/15/stop-wasting-your-time-on-annoying-ctf-challenges.html](https://ronin-rb.dev/blog/2024/03/15/stop-wasting-your-time-on-annoying-ctf-challenges.html)

The other day I spotted someone in [nahamsec](https://nahamsec.com/)’s Discord server struggling with a CTF challenge that involved decoding an encrypted string. They had already tried [CyberChef](https://cyberchef.org/), but no luck. Other users encouraged them to “try harder”.

They were trying to decrypt the following string:

```
pai1nYPe3tLR1IOH1YKH14XegtCDgtSH1oKAgNGF34eH19bSmw== 
```

Clearly it looks base64 encoded, but if you base64 decode the string you get binary data. The user said that the challenge hinted that the string was XOR encrypted and the plain-text was of the form `CNS{md5hash}`.

Using the `ronin irb` console I was able to easily solve this in *one line of Ruby*:

```
Chars.ascii.map { |key| "pai1nYPe3tLR1IOH1YKH14XegtCDgtSH1oKAgNGF34eH19bSmw==".
base64_decode.xor(key) }.find { |text| text.start_with?('CNS{') } 
```

Starting with the assumption that the CTF challenge probably used an XOR encryption key of a single ASCII character, all we have to do is bruteforce decrypt the base64 decoded string using every possible character in the ASCII character set, then find the decrypted string that starts with `CNS{`.

Running the above code gives us the answer:

```
CNS{e88472ea3da1c8d6ed2a0dff7c9aa104} 
```

That’s it!

You don’t have to “try harder”, just use [Ronin](/install/)!

If Ronin interests you or you like the work we do, consider donating to Ronin on

[GitHub](https://github.com/sponsors/ronin-rb)

,

[Patreon](https://patreon.com/roninrb)

, or

[Open Collective](https://opencollective.com/ronin-rb)

so we can continue building high-quality free and Open Source security tools and Ruby libraries.