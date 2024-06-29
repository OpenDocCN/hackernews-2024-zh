<!--yml

类别：未分类

日期：2024-05-27 15:02:43

-->

# Python用户：BIPClip瞄准您的比特币钱包，通过PyPI - The New Stack

> 来源：[https://thenewstack.io/python-users-bibclip-is-after-your-bitcoin-wallet-via-pypi/](https://thenewstack.io/python-users-bibclip-is-after-your-bitcoin-wallet-via-pypi/)

随着比特币价值超过70,000美元，加密货币再次成为一个大事件。因此，网络安全公司ReversingLabs揭示了一个恶意的黑客攻击活动，旨在窃取加密货币钱包恢复短语。该攻击活动利用[Python](https://thenewstack.io/what-is-python/)作为其基础。

"BIPClip"（https://pypi.org/）巧妙地伪装成[有用的开源库](https://thenewstack.io/vendoring-why-you-still-have-overlooked-security-holes/)，利用[Python软件包索引（PyPI）](https://pypi.org/)。这个复杂的操作涉及到七个精心设计的开源软件包，每个软件包有多个版本。这些软件包的目标是基于[比特币改进提案39（BIP39）](https://trezor.io/learn/a/what-is-bip39)标准的助记短语。

## BIP39助记短语

BIP39短语是[加密钱包安全](https://thenewstack.io/github-iam-private-creds-are-being-cryptojacked-by-elektra-leak/)的基石。它们用于生成一个二进制种子，进而[创建确定性比特币](https://thenewstack.io/create-bitcoin-buy-and-sell-alerts-with-influxdb/)钱包。这些钱包可以在系统之间无缝共享，使用一组2048个词，这些词比传统的二进制或十六进制种子表示更容易记忆。

当然，仅有2048个单词，黑客迟早会利用它们。

ReversingLabs最初发现了两个PyPI软件包，*mnemonic_to_address*和*bip39_mnemonic_decrypt*，用于窃取用于保护加密货币钱包的敏感数据。*Mnemonic_to_address*充当“干净”软件包，其中恶意的*bip39_mnemonic_decrypt*被列为依赖项。*Mnemonic_to_address*实现了其宣传的功能，从用户的秘密助记短语中创建种子。它通过将BIP39数据转发到从合法项目[Ethereum](https://github.com/ethereum)的[eth-account](https://github.com/ethereum/eth-account)导入的功能来实现此功能。

到目前为止，一切都很顺利。但是，恶意文件依赖开始攻击。它通过使用Base64对提供的助记短语进行编码，并将其通过[HTTP POST请求](https://thenewstack.io/blockchain-direct-http-requests-via-the-internet-computer/)发送到数据泄露服务器。它进一步通过将助记短语隐藏在“许可证”数据字段中来伪装助记短语。

## 一个复杂的依赖网络

ReversingLabs 对 BIPClip 机制的深入分析揭示了一个旨在规避检测的复杂依赖网络。这次行动的隐秘性通过发现早在三月初的其他软件包，如 HashSnake，进一步扩展了 BIPClip 活动的影响范围。这种伪装策略，再加上有意使用一次性的 PyPI 管理员账户，突显了攻击者为掩盖真实意图而不惜一切的决心。

尽管这次攻击在隐藏其踪迹方面具有独创性，但总体影响似乎很快得到了缓解，恶意[软件包在其被发现后不久便从 PyPI 中被移除](https://thenewstack.io/poisoned-lolip0p-pypi-packages/)。然而，在其被移除之前的下载量表明可能比预期的传播更广泛，突显了[软件供应链攻击](https://thenewstack.io/lessons-learned-from-2021-software-supply-chain-attacks/)所带来的持续风险。

虽然这次或许幸免于难，但 ReversingLabs 的发现提醒我们，加密货币行业面临着持续的威胁。如果您从事加密货币相关工作，必须加强对这种阴险威胁的防范。

毕竟，就像据说20世纪20年代著名的银行劫匪威利·萨顿在被问及为什么抢劫银行时所说的那样，“因为那里有钱”。今天，[加密钱包才是钱所在的地方](https://thenewstack.io/crypto-and-nfts-dominate-the-headlines-but-smart-money-bets-on-web3/)。

*编者按：标题已更改以准确反映 BIPClip 的名称。*

<template class="youtube-subscribe-block-template"></template><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 68 31" version="1.1"><title>组</title> <desc>用草图创建。</desc></svg>
