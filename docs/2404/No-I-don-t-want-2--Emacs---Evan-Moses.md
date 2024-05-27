<!--yml
category: 未分类
date: 2024-05-27 13:15:44
-->

# No I don't want 2, Emacs - Evan Moses

> 来源：[https://www.emoses.org/posts/dont-write-2/](https://www.emoses.org/posts/dont-write-2/)

# No I don't want 2, Emacs

This blog, and the vast majority of the code I write, is written in Emacs with [`evil`](https://github.com/emacs-evil/evil) (a vim emulation mode). I have a nasty habit of mashing `:w2<ret>` when I really was trying to save the current buffer with `:w<ret>` . `:w2` writes the current buffer to a new file called `2`, which I don’t believe I have ever done on purpose.

So, I added this little gem to my .emacs, and it’s saved me any number of times:

```
(defun my:evil-write (&rest args)  "I constantly hit :w2<ret> and save a file named 2\.  Verify that I want to do that" (if (equal "2" (nth 3 args)) (y-or-n-p "Did you really mean to save a file named 2?") t)) (advice-add #'evil-write :before-while #'my:evil-write) 
```

The `:before-while` advice lets you run a function that gets the same arguments as the advised function. If it returns a truthy value, the advised function is run as usual, but if it returns `nil`, the original function is never run.

Share and enjoy.

## Update Apr 16, 2024

To be clear: I’m not criticizing Emacs or `evil` here. To the contrary, I’ve got a personal problem, and thanks to the infinite flexibility of Emacs I was able to craft a personal solution in 6 lines. I wish the rest of my software were so flexible.