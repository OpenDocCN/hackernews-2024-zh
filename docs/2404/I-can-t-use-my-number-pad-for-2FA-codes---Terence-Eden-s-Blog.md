<!--yml
category: 未分类
date: 2024-05-27 13:17:01
-->

# I can’t use my number pad for 2FA codes – Terence Eden’s Blog

> 来源：[https://shkspr.mobi/blog/2024/04/i-cant-use-my-number-pad-for-2fa-codes/](https://shkspr.mobi/blog/2024/04/i-cant-use-my-number-pad-for-2fa-codes/)

This has to be the most infuriating bug report I've ever submitted.

I went to type in my 2FA code on a website - but no numbers appeared on screen. Obviously, I was an idiot and had forgotten to press the NumLock button. D'oh! I toggled it on and typed again. No numbers appeared. I switched to another tab, my numbers appeared when I typed them. So I was reasonably confident that my keyboard was working.

I swapped back to the 2FA entry and tried again. Still nothing. Then I tried typing the numbers using the number row on my keyboard. My 2FA code appeared.

WHAT IN THE SAINTED NAME OF ALPHONSE CHAPANIS IS GOING ON?!?!?

Developers often use JavaScript to "improve" the standard features of HTML. For example, using `<input type="number">` has some [accessibility concerns](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/number#accessibility_concerns) and using [`inputmode="numeric"`](https://css-tricks.com/everything-you-ever-wanted-to-know-about-inputmode/#aa-numeric) is great for showing a number key board on mobile, but not much else.

So a developer wants a reliable way to make sure a user can *only* type numbers. Fair enough.

There are two ways to do this - a right way and a wrong way - using [`KeyboardEvent`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent).

One way is to [listen for the character being sent from the keyboard](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key) - known as the `key`.

The other is to [listen for the *button* being pressed on the keyboard](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code) - known as the `code`.

A good demo of this is at [keyjs.dev](https://keyjs.dev/) - play around with it to see what keyboard buttons your browser can detect.

When I press 7 on the top row of my keyboard, the key is 7 and the code is **`Digit7`**.

But when I press 7 on my number pad, the key is 7 but the code is **`Numpad7`**.

The JavaScript on the website was rejecting any key code which wasn't a "Digit"!

Perhaps I am a weirdo for insisting on both having and using my numpad? Perhaps developers need to test on something other than MacBooks? Perhaps JavaScript was a mistake and the Web would be better without it?

Either way, don't be like that website. Let users type in using whatever keys they like.