<!--yml

category: 未分类

date: 2024-05-27 13:15:44

-->

# 不，我不想要2，Emacs - Evan Moses

> 来源：[https://www.emoses.org/posts/dont-write-2/](https://www.emoses.org/posts/dont-write-2/)

# 不，我不想要2，Emacs

这篇博客，以及我大部分的代码都是用 Emacs 和 [`evil`](https://github.com/emacs-evil/evil)（一个 vim 模拟模式）写的。我有一个坏习惯，当我真的想要保存当前缓冲区时，会乱按 `:w2<ret>`。`：w2` 会把当前缓冲区保存到一个叫做 `2` 的新文件中，我不认为我曾经有意这样做过。

所以，我把这个小宝石添加到我的 .emacs 文件中，它已经帮我省了不少事：

```
(defun my:evil-write (&rest args)  "I constantly hit :w2<ret> and save a file named 2\.  Verify that I want to do that" (if (equal "2" (nth 3 args)) (y-or-n-p "Did you really mean to save a file named 2?") t)) (advice-add #'evil-write :before-while #'my:evil-write) 
```

`:before-while` 建议让你运行一个接收和被建议函数相同参数的函数。如果它返回一个真值，建议的函数像平常一样运行，但如果它返回 `nil`，原始函数就不会被运行。

分享并享受。

## 更新于 2024 年 4 月 16 日

清楚地说一下：我这里并不是在批评 Emacs 或 `evil`。相反，我有一个个人问题，多亏了 Emacs 的无限灵活性，我能够用 6 行代码找到了一个个人解决方案。我希望我的其它软件也能如此灵活。
