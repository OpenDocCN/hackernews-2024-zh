<!--yml

category: 未分类

date: 2024-05-27 14:37:11

-->

# 通过完成功能改善我的Emacs体验

> 来源：[https://martinfowler.com/articles/2024-emacs-completion.html](https://martinfowler.com/articles/2024-emacs-completion.html)

我已经使用Emacs很多年了，用它来写网站上的任何文章，写我的书，以及大部分编程工作。（在Java和R方面的例外情况是IntellJ IDEA和RStudio。）因此，我很高兴看到近几年来Emacs改进了许多功能，使其感觉不再是一条进化的死胡同。对我Emacs体验最大的改进之一是使用正则表达式来完成列表。

许多Emacs命令生成可供选择的事物列表。当我想访问（打开）一个文件时，我键入组合键来查找文件，Emacs弹出一个候选文件列表在迷你缓冲区（一个特殊区域与命令进行交互）。这些文件列表可能会相当长，特别是如果我要求列出当前项目中的所有文件。

要指定我想要的文件，我可以输入一些文本来过滤列表，所以如果我想打开文件`articles/simple/2024-emacs-completion.md`，我可能会输入`emacs`。我不必只获取那一个文件，只要过滤到一个足够小的列表通常就足够了。

有一种特定的正则表达式构建器我发现最有帮助，这种构建器通过空格分隔正则表达式。这样我可以输入`articles emacs`来获取包含文件路径中“articles”和“emacs”的任何文件路径列表。实际上，它将字符串`articles emacs`转换为正则表达式`\\(articles\\).*\\(emacs\\)`。更好的是，这样的匹配器允许我以任何顺序输入正则表达式，因此“emacs articles”也将匹配。这样一来，一旦第一个正则表达式弹出一个过滤列表，我可以使用第二个正则表达式选择我想要的文件，即使区分性正则表达式早于我的初始搜索。

安装这样一个完成匹配器对我使用 Emacs 有了显著的影响，因为在与命令交互时，它使得在大列表中筛选变得轻而易举。其中最重要的之一是它如何改变了我使用 `M-x` 的方式，这个组合键会显示所有交互式 Emacs 函数的列表。通过正则表达式匹配器来筛选列表，它允许我仅通过几个按键来调用一个 Emacs 命令，而不必记住键盘快捷键。这样，我不经常列出所有打开的缓冲区，所以我不需要记住它的快捷键组合。我只需快速输入 `M-x ib`，然后 `ibuffer` 就会迅速弹出。这得益于我用于 `M-x` 的命令 (`counsel-M-x`) 在正则表达式中插入了一个“`^`”作为第一个字符，这将第一个正则表达式锚定在行的开头。由于我所有的自写函数都以 `mf-` 开头，我可以轻松地找到自己的函数，即使它们的名称很长。我写了一个命令来移除 URL 中的域名，我称之为 `mf-url-remove-domain`，可以通过 `M-x mf url` 调用它。

Emacs 中有相当多的包可以做这种匹配，足以让人感到相当困惑。我最近在使用的一个是[Ivy](https://oremacs.com/swiper/)。默认情况下，它使用空格分隔的正则表达式匹配器，但不支持任何顺序。为了按照我喜欢的方式配置它，我使用

```
(setq ivy-re-builders-alist '((t . ivy--regex-ignore-order))) 
```

Ivy 是一个名为`counsel`的包的一部分，包含各种增强此类选择的命令。

Ivy 并不是唯一做这种事情的工具。事实上，Emacs 中的补全工具世界让我感到非常困惑：有许多工具有重叠和交互，我并不真正理解。这些领域中的工具包括[Helm](https://emacs-helm.github.io/helm/)、[company](https://company-mode.github.io/)、[Vertico](https://github.com/minad/vertico)和[Consult](https://github.com/minad/consult)。《Mastering Emacs》有一篇关于[理解迷你缓冲区完成](https://www.masteringemacs.org/article/understanding-minibuffer-completion)的文章，但它没有解释它所讨论的机制如何与 Ivy 的功能结合，我也没有花时间去弄清楚这一切。

总的来说，我强烈推荐书籍[《Mastering Emacs》](https://www.masteringemacs.org/)，来学习如何使用这个令人难以置信的工具。Emacs 有如此多的功能，即使是像我这样使用几十年的用户，也发现这本书引导我“我原来不知道它还能这样”。

对于那些好奇的人，这里是我的 Emacs 配置的相关部分

```
(use-package ivy
  :demand t
  :diminish ivy-mode
  :config
  (ivy-mode 1)
  (counsel-mode 1)
  (setq ivy-use-virtual-buffers t)
  (setq ivy-use-selectable-prompt t)
  (setq ivy-ignore-buffers '(\\` " "\\`\\*magit"))
  (setq ivy-re-builders-alist '(
                                (t . ivy--regex-ignore-order)
                                ))
  (setq ivy-height 10)
  (setq counsel-find-file-at-point t)
  (setq ivy-count-format "(%d/%d) "))

(use-package counsel
  :bind (
         ("C-x C-b" . ivy-switch-buffer)
         ("C-x b" . ivy-switch-buffer)
         ("M-r" . counsel-ag)
         ("C-x C-d" . counsel-dired)
         ("C-x d" . counsel-dired)
         )
  :diminish
  :config
  (global-set-key [remap org-set-tags-command] #'counsel-org-tag))

(use-package swiper
  :bind(("M-C-s" . swiper)))

(use-package ivy-hydra) 
```
