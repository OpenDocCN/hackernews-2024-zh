<!--yml
category: 未分类
date: 2024-05-27 15:02:43
-->

# Python Users: BIPClip Is After Your Bitcoin Wallet, Via PyPI - The New Stack

> 来源：[https://thenewstack.io/python-users-bibclip-is-after-your-bitcoin-wallet-via-pypi/](https://thenewstack.io/python-users-bibclip-is-after-your-bitcoin-wallet-via-pypi/)

With Bitcoin values hovering over $70,000, the cryptocurrency has become a big deal again. So it can come as no surprise that cybersecurity firm ReversingLabs has [uncovered](https://www.reversinglabs.com/blog/bipclip-malicious-pypi-packages-target-crypto-wallet-recovery-passwords) a nefarious hacking campaign aimed at pilfering cryptocurrency wallet recovery phrases. And that campaign uses [Python](https://thenewstack.io/what-is-python/) as its base.

Dubbed “BIPClip,” it cleverly exploits the [Python Package Index (PyPI)](https://pypi.org/) by masquerading as a [useful open source library](https://thenewstack.io/vendoring-why-you-still-have-overlooked-security-holes/). This sophisticated operation involves seven ingeniously crafted open source packages, each with multiple versions. These packages target the mnemonic phrases based on the [Bitcoin Improvement Proposal 39 (BIP39)](https://trezor.io/learn/a/what-is-bip39) standard.

## BIP39 Mnemonic Phrases

The BIP39 phrases are a cornerstone of [crypto wallet security](https://thenewstack.io/github-iam-private-creds-are-being-cryptojacked-by-elektra-leak/). They’re used to generate a binary seed that, in turn, [creates deterministic Bitcoin](https://thenewstack.io/create-bitcoin-buy-and-sell-alerts-with-influxdb/) wallets. These wallets can be seamlessly shared across systems, using a set of 2,048 words that are easier for humans to remember than traditional binary or hexadecimal seed representations.

Of course, with only 2,048 words, it was only a matter of time before hackers exploited them.

ReversingLabs initially discovered two PyPI packages, *mnemonic_to_address* and *bip39_mnemonic_decrypt*, which are used to exfiltrate sensitive data used to protect cryptocurrency wallets. *Mnemonic_to_address* acts as a “clean” package with the malicious *bip39_mnemonic_decrypt* listed as a dependency. *Mnemonic_to_address* implements its advertised functionality of creating a seed from the user’s secret mnemonic phrase. It does this by forwarding the BIP39 data to functions imported from a legitimate project: [Ethereum](https://github.com/ethereum)‘s [eth-account](https://github.com/ethereum/eth-account).

So far, so good. But then malicious file dependency comes into attack. It does this by encoding the provided mnemonic passphrase using Base64 and then sending it to the exfiltration server using an [HTTP POST request](https://thenewstack.io/blockchain-direct-http-requests-via-the-internet-computer/). It further disguises the passphrase by hiding it in the “license” data field.

## A Complex Web of Dependencies

ReversingLabs’ deep dive into the mechanics of BIPClip revealed a complex web of dependencies designed to evade detection. The operation’s stealthiness is underscored by the discovery of additional packages, such as HashSnake, in early March, further expanding the BIPClip campaign’s reach. This strategy of camouflage, coupled with the deliberate use of throwaway PyPI maintainer accounts, underscores the lengths to which these attackers will go to mask their true intentions.

Despite this attack’s ingenuity in hiding its tracks the overall impact appears to have been mitigated swiftly, with the malicious [packages removed from PyPI](https://thenewstack.io/poisoned-lolip0p-pypi-packages/) shortly after their discovery. However, the number of downloads prior to their removal suggests a potentially wider spread than anticipated, highlighting the ongoing risks posed by [software supply chain attacks](https://thenewstack.io/lessons-learned-from-2021-software-supply-chain-attacks/).

While you may have dodged a bullet this time, ReversingLabs’ findings serve as a stark reminder of the persistent threats facing the cryptocurrency sector. If you work with cryptocurrency, you must bolster your defenses against such insidious threats.

After all, just like the famous 1920s bank robber Willie Sutton supposedly said when he was asked why he robbed banks, “Because that’s where the money is.” Today, [crypto wallets are where the money](https://thenewstack.io/crypto-and-nfts-dominate-the-headlines-but-smart-money-bets-on-web3/) is.

*Editor’s note: The headline has been changed to accurately reflect the name of BIPClip. *

<template class="youtube-subscribe-block-template"></template><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 68 31" version="1.1"><title>Group</title> <desc>Created with Sketch.</desc></svg>