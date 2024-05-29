<!--yml
category: 未分类
date: 2024-05-27 14:40:28
-->

# pow

> 来源：[https://tropical.pages.dev/pow/](https://tropical.pages.dev/pow/)

**Update available, please** 

no message

# 

This is the current winning text

Change the top text for everyone by brute force

info

up to 32 chars: 0-9, a-z, A-Z, "-", "_", " ", "#", no extra spaces

#### What is this

*   Who has mined the hardest will have their text shown on the website
*   Proof of Work
*   Text of the smallest sha256(nonce + text) value seen in P2P network should be displayed on top of this page

#### Same idea, but with colors

Current winner: #### And with a grid

Pixel: click on grid New color:      

#### Custom PoW submit

My PoW is too slow? You can submit your results manually here:

For text: `nonce||text` (`||` = string concat)
where nonce is `^[0-9a-f]{32}$`
and text is `^(?=.{1,32}$)[0-9a-zA-Z_\-#]+(?: [0-9a-zA-Z_\-#]+)*$`

For colors: Prefix by key where key is 1 for red, 2 for green, 3 for blue, 4 for yellow.
Followed by "!", and "#" at the end.
Like this: `key||"!"||nonce||"#"` (without "")

For grid: It's 1024 pixels. Encode like this:
`key = hex(5+x_coord+y_coord*32)` then `"!"` and color is `^#[0-9a-f]{6}$`
Like this: `key||"!"||nonce||color`

#### Debug

peers: 0

best hash of text: none

color winner hash: none

best hash in grid: none

If there are problems, reloading the page can help

#### Log

#### Log 2

fix

#### Contact

3405691582@proton.me