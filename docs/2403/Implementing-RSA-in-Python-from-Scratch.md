<!--yml

category: 未分类

date: 2024-05-27 14:32:39

-->

# 在Python中从零开始实现RSA

> 来源：[https://coderoasis.com/implementing-rsa-from-scratch-in-python/](https://coderoasis.com/implementing-rsa-from-scratch-in-python/)

请注意，我必须强调这里介绍的代码和技术仅供教育目的，不应在现实世界的应用中使用，除非经过慎重考虑和专家指导。

同时，理解RSA加密的原理，并探索各种实现方式，对于教育目的和理解如何编写加密方法，构建安全加密系统非常有价值，实际使用需要遵循行业标准、最佳实践和彻底的安全评估。不恰当的实现或不正确的加密技术使用可能导致严重的漏洞，危及敏感数据的保密性和完整性。

* * *

我相信许多程序员，特别是网页开发人员都听说过RSA加密系统。RSA是一种非对称加密系统，意味着一个密钥用于加密，另一个用于解密。我看过很多解释非对称加密一般原理的文章，但没有看到能够简单易懂地解释这些算法背后数学原理的文章，因此我决定写这篇文章。

## **RSA背后的数学**

首先，正如我在上面的段落中提到的，要传输一个RSA加密的消息，你需要两个密钥。一个只能加密，一个只能解密。让加密密钥表示为`e`，解密密钥将表示为`d`，消息将表示为`m`。RSA背后的关键原理是：

```
(m^e)^d ≡ m (mod n)
```

如果这些数被适当地选择（如下文所示），那么根据已知的`e`和`n`找到`d`的困难可能非常大。

但首先，我们需要介绍一些数论中的新概念。

+   `a ≡ b (mod n)`意味着存在整数`x`使得`a + nx = b`。更直观的解释是`a / n`的余数等于`b / n`的余数。

+   `gcd(a, b)`是同时除以`a`和`b`的最大数。

+   `lcm(a, b)`是既是`a`又是`b`的倍数的最小数。

+   `λ(n)`是这样一个数，对于任何使`gcd(x, n) = 1`的`x`，都有`x^λ(n) ≡ 1 (mod n)`。由此可推出，如果`a ≡ b (mod λ(n))`，则`x^a ≡ x^b (mod n)`

## 生成密钥

让我们令`n = pq`，其中`p`和`q`是两个素数。由于`λ(p) = p - 1`和`λ(q) = q - 1`（查阅费马小定理的证明），所以`λ(n) = (p - 1) * (q - 1)`。

现在我们必须解决`ed ≡ 1 (mod λ(n))`的问题。`e`可以是某个素数（通常是65537），`d`可以使用扩展欧几里得算法计算（将在代码部分中编写和解释）从方程`ed + xλ(n) = gcd(e, λ(n))`中得到（`d`和`x`是要计算的系数）。

`对（e，n）`用于加密（`(m^e) % n`），而`对（d，n）`用于解密（`(m^d) % n`）

计算完密钥后，可以丢弃p和q。请注意，没有p和q，找到`λ(n)`将意味着分解n，这对于通常使用的值高达2^2048的n来说并不是易事。

## **在Python中实现RSA**

首先列出过程及其步骤：

#### 密钥生成：

+   找到两个随机素数，`p`和`q`

+   计算`n = p * q`和`λ(n) = (p - 1) * (q - 1)`

+   使`e`等于某个质数，例如`e = 35537`

+   使用扩展欧几里得算法从方程`ed + λ(n)x = gcd(e，λ(n)) = 1`计算`d`（从这一点开始我们将其称为`eucalg`）

##### 加密：

+   将消息分成部分（如果n为2048位，则为256位）

+   每个部分（表示为`m`）加密为`(m^e) % n`

#### 解密：

+   将消息分成几部分，如上所述

+   每个部分（表示为`m`）解密为`(m^d) % n`

## 扩展欧几里得算法

该算法依赖于以下事实：如果`x`同时除以`a`和`b`，那么将会有一对系数`(k，l)`，满足

`a * k + b * l = x`

该算法通过重复从较大的参数中减去较小的参数，直到较小的参数变为0来找到这些系数（和`x`）。与重复减法不同，您可以计算`b`可以从`a`中减去多少次（`k = a // b`），然后减去`b*k`。这种优化使得算法的运行时间为O(log max(a, b))，这比以前要快得多。

下面是Python中算法的实现：

```
def eucalg(a, b):
	# make a the bigger one and b the lesser one
	swapped = False
	if a < b:
		a, b = b, a
		swapped = True
	# ca and cb store current a and b in form of
	# coefficients with initial a and b
	# a' = ca[0] * a + ca[1] * b
	# b' = cb[0] * a + cb[1] * b
	ca = (1, 0)
	cb = (0, 1)
	while b != 0:
		# k denotes how many times number b
		# can be substracted from a
		k = a // b
		# a  <- b
		# b  <- a - b * k
		# ca <- cb
		# cb <- (ca[0] - k * cb[0], ca[1] - k * cb[1])
		a, b, ca, cb = b, a-b*k, cb, (ca[0]-k*cb[0], ca[1]-k*cb[1])
	if swapped:
		return (ca[1], ca[0])
	else:
		return ca 
```

注意，上述函数可以为系数产生负数，因此如果发生这种情况，我们只需将`d`设置为`λ(n) - d`。

## 快速指数运算与模

有些人可能建议只使用`(b ** e) % n`，但这种方法在时间和内存上并不是很好，因为`b ** e`可能非常大，尽管只需要最后的大约2000位。解决方案是在每次除法后重新实现带模指数运算。

下面的指数实现具有对数时间复杂度。它不是从1到`e`迭代，并通过`r`（结果）乘以`b`，而是从`e`的最高有效位到最低有效位迭代，并在每位上进行`r = r*r + bit`。这是因为如果`r`等于`b^x`，并且您正在向x的末尾附加一个位，则新的x将是`x * 2 + bit`。

```
def modpow(b, e, n):
	# find length of e in bits
	tst = 1
	siz = 0
	while e >= tst:
		tst <<= 1
		siz += 1
	siz -= 1
	# calculate the result
	r = 1
	for i in range(siz, -1, -1):
		r = (r * r) % n
		if (e >> i) & 1: r = (r * b) % n
	return r 
```

## 生成/加密/解密

密钥生成、加密和解密都已在数学部分解释过，下面的代码只是该过程的实现。

```
def keysgen(p, q):
	n = p * q
	lambda_n = (p - 1) * (q - 1)
	e = 35537
	d = eucalg(e, lambda_n)[0]
	if d < 0: d += lambda_n
        # both private and public key must have n stored with them
	return {'priv': (d, n), 'pub': (e, n)}

def numencrypt(m, pub):
	return modpow(m, pub[0], pub[1])

def numdecrypt(m, priv):
	return modpow(m, priv[0], priv[1]) 
```

## 代码测试

我将代码存储在名为rsa.py的文件中，并在Python shell中运行以下代码：

```
>>> import rsa
>>> keys = rsa.keysgen(31337, 31357)
>>> keys
{'priv': (720926705, 982634309), 'pub': (35537, 982634309)}
>>> priv = keys['priv']
>>> pub = keys['pub']
>>> msg = 80087
>>> enc = rsa.numencrypt(msg, priv)
>>> enc
34604568
>>> dec = rsa.numdecrypt(enc, pub)
>>> dec
80087
>>> 
```

## **结论**

写完这篇文章后，我意识到实现一个可用的Python RSA程序比我最初推测的要复杂得多。因此，我决定将这个主题分成多篇文章。在本文中提供的代码中，您已经编写了RSA的所有核心部分。但是，您仍然需要一个随机素数生成器和明文加密（`numencrypt`和`numdecrypt`仅适用于小于`n`的数字`m`）。所有这些内容将包含在即将发布的下一篇文章中。

* * *

[Python从零开始实现RSA（第4部分）](https://coderoasis.com/implementing-rsa-from-scratch-in-python-part-4/)，如果您想继续阅读，我们还有第2部分甚至第3部分供那些对从零开始实现Python中的RSA感兴趣的人阅读。

[Python从零开始实现RSA（第2部分）

[Python从零开始实现RSA](https://coderoasis.com/implementing-rsa-from-scratch-in-python-part-2/)，我相信许多程序员，特别是Web开发人员，都听说过RSA加密系统。RSA是一种非对称加密系统，意味着一个密钥用于加密，另一个用于解密。我看过很多解释一般原理的文章

两篇文章主要关注RSA的思想和实现。因此，它们忽略了现代加密中的一个相关主题。侧信道攻击利用我们通常忽略的数据，例如硬件声音、电磁波和时间。时序攻击是最

我要强调的是，这里提供的代码和技术仅用于教育目的，并且在没有仔细考虑和专家指导的情况下，不应用于现实世界的应用程序。同时，理解RSA加密原理并探索各种

* * *

> 您是否喜欢从CoderOasis科技博客上阅读的内容？我们建议阅读我们的下一篇选择[***Hacktivism: Social Justice by Data Leaks and Defacements***](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/)。

[Hacktivism: Social Justice by Data Leaks and Defacements

大约在二月底，自称为JaXpArO和My Little Anonymous Revival Project的黑客活动分子侵入了名为Gab的极右社交媒体平台。他们成功地从后端数据库获取了70GB的数据。数据包含用户配置文件、私人帖子、聊天消息等等 - 很多

* * *

> 我们有一个名为[**社区论坛**](https://forums.coderoasis.com/?ref=coderoasis.com)和[**Discord 服务器**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)的地方，我们邀请每个人加入我们。想要与我们社区的其他成员讨论本文吗？想要加入一个轻松的地方，讨论编程、网络安全、网页开发和Linux等主题吗？考虑今天加入我们！

[加入 CoderOasis.com 的 Discord 服务器！

CoderOasis提供关于编程、安全、网页开发、Linux、系统管理等技术新闻文章。 | 112 名成员](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)  [CoderOasis 论坛

CoderOasis 社区论坛，我们的成员可以在此讨论技术并共享资源。](https://forums.coderoasis.com/?ref=coderoasis.com)
