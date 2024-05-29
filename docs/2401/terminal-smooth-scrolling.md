<!--yml
category: 未分类
date: 2024-05-27 14:29:51
-->

# terminal smooth scrolling

> 来源：[https://flak.tedunangst.com/post/terminal-smooth-scrolling](https://flak.tedunangst.com/post/terminal-smooth-scrolling)

I didn’t realize I needed this until I implemented it, and now, oh wow, can’t imagine life without it.

Normally, a terminal draws lines of text at fixed intervals. Line 1 starting at pixel 0, then line 2 at 24 pixels, etc. When the text scrolls, line 2 immediately becomes line 1 and gets drawn at pixel 0\. These sudden jumps are incompatible with how your eyes track motion, however. The result is a blurry mess that’s hard to read.

Long ago, hardware terminals had the option for smooth scrolling, incrementally shifting each line of text up over several frames, but such wizardry has been lost to the ravages of time. I thought it would be fun to implement, and maybe a cool trick to show off, but before seeing the result figured it would be mostly a gimmick. Instead, it’s entirely changed how I scroll through text.

### results

Figure we can just start with a video showing before and after. Scrolling through the *ls* man page. About half way through, the terminal switches to smooth scrolling.

 </images/scrollcap.mp4> 
I can’t quite read fast enough to keep up with the scroll rate, not every word anyway, but I can skim the text as it passes by until I find the section I’m looking for. Very helpful for reading code or documentation when I’m not entirely sure what that is.

I find that scrolling like this is ultimately more effective than paging one screen at a time, then trying to skim the whole screen. I always miss something when doing that.

### implementation

There’s an open issue requesting smooth scrolling for just about every terminal I looked at, but none implemented. Usually the discussion gets bogged down in trivialities. I found it’s not hard to do something, and the result is quite pleasing. I guess it helps to only worry about one user.

The first big issue is oh, no, it makes things slow. Only when enabled. There’s a terminal mode sequence to turn it on and off. I did find it slightly distracting for regular shell work, like running *ls*, so only enable it for commands as necessary. And the scrolling speed can be ramped up by the amount to scroll. I picked ten frames as target, although it takes a bit longer than that as it decelerates towards the end.

The second big issue relates to the scrolling region. Programs like *vi* scroll most lines on the screen, but they leave the status bar on the bottom alone. Indeed, it looks funny to scroll that. But this is easily resolved by only applying the smooth scrolling effect to the part of the screen that scrolled.

Finally, there’s the question of what to do with the scrolled off content. When a line disappears off the alternate screen, there’s no backlog to send it to. It ceases to exist. So what to draw in that space before the next line finishes scrolling into position? The answer is nothing. You can leave a line or two empty at the top (or bottom) of screen. The new content, the stuff I’m focused on, is appearing at the other side. I’m not watching the disappearing lines; I’m watching the appearing lines.

As for actual implementation, every cell on the screen is drawn with some vertices. If we’re smooth scrolling, we offset those vertices in the opposite direction. If the text on the screen just moved up, we push every vertex down, so it appears in the same position as previously. Then we decay the offset each frame, so that the text slides into position.

The main part of the code is about 25 lines, plus a few miscellaneous lines here and there.

### misc details

The proof of concept was very quick to get going, but it did require some experimentation and tuning. If we start too slow, we fall behind the input. If we accelerate too fast, the result looks jerky. Too many derivatives.

Current approach is to pick a velocity of distance divided by 10\. Then we use a weighted average with previous velocity to smooth the smoothing. If we’re more than ten lines behind, which happens rarely but can happen if the whole screen is redrawn, then do a jump scroll.

The region to smooth scroll is determined by the scrolling region at the time of scrolling, not painting. *vim* for instance will set the scroll region to 1;23 to adjust the file contents displayed, then immediately switch back to 1;24 to update the status line. Since we don’t want to scroll the status bar, we want to apply the effect to the 1;23 region previously set.

For now, we scroll interior lines as well, such as for cut and paste operations. This is somewhere between distracting and helpful. The motion turns out to be a helpful hint of what changed, and it’s fast enough it doesn’t get in the way.

If an offscreen tab scrolls, the smooth effect is applied when the user switches back. This doesn’t usually happen (unless smooth scrolling is enabled at the shell) but I think it can be a helpful indicator that something changed. This is *new* compiler output.

Drawing the screen ten times for each scroll obviously uses ten times more CPU power.

### usage

Smooth scrolling is enabled by the standard “\e[4?h” control sequence.

Helpful .vimrc:

```
set t_TI=^[[4?h
set t_TE=^[[4?l
```