<!--yml
category: 未分类
date: 2024-05-27 14:37:11
-->

# Improving my Emacs experience with completion

> 来源：[https://martinfowler.com/articles/2024-emacs-completion.html](https://martinfowler.com/articles/2024-emacs-completion.html)

I’ve been using Emacs for many years, using it for any writing for my website, writing my books, and most of my programming. (Exceptions have been IntellJ IDEA for Java and RStudio for R.) As such I’ve been happy to see a lot of activity in the last few years to improve Emacs’s capabilities, making it feel rather less than a evolutionary dead end. One of the biggest improvements to my Emacs experience is using regexs for completion lists.

Many Emacs commands generate lists of things to pick from. I want to visit (open) a file I type the key combination to find a file, and Emacs pops up a list of candidate files in the minibuffer (a special area for to interact with commands). These file lists can be quite long, particularly should I ask for a list of all files in my current project.

To specify the file I want, I can type some text to filter the list, so if I want to open the file `articles/simple/2024-emacs-completion.md` I might type `emacs`. I don’t have to get only that one file, just filtering to a small enough list is often enough.

There’s a particular style of regex builder that I find the most helpful, one that separates regexs by spaces. This would allow me to type `articles emacs` to get a list of any file paths that contain “articles” and “emacs” in their file path. It essentially turns the string “articles emacs” into the regex `\\(articles\\).*\\(emacs\\)`. Better yet, such a matcher allows me to type the regexs in any order, so that “emacs articles” would also match. This way once the first regex pops up a filtered list, I can use a second regex to pick the one I want, even if the distinguishing regex is earlier than my initial search.

Installing such a completion matcher has had a remarkable effect on my use of Emacs, since it makes it a breeze to filter large lists when interacting with commands. One of the most significant of these is how it changes my use of `M-x`, the key combo that brings up a list of all interactive Emacs functions. With a regex matcher to filter the list, it allows me to invoke an Emacs command using its name, with just a few keystrokes. That way I don’t have to remember the keyboard shortcut. With this, I invoke commands that I use less frequently through `M-x`. I don’t list all open buffers very often, so rather than try to remember the key combination for it, I just type `M-x ib` and `ibuffer` quickly pops up. This is helped that the command I use for `M-x` (`counsel-M-x`) inserts a “`^`” as the first character in the regex, which anchors the first regex to the beginning of the line. Since I prefix all my self-written functions with `mf-`, I can easily find my own functions, even if they have a long name. I wrote a command to remove the domain from a URL, I call it `mf-url-remove-domain` and can invoke it with `M-x mf url`.

There are quite a few packages in Emacs that do this kind of matching, enough to be rather confusing. The one I’m using these days is [Ivy](https://oremacs.com/swiper/). By default it uses a space-separated regex matcher, but one that doesn’t support any order. To configure it the way I like it I use

```
(setq ivy-re-builders-alist '((t . ivy--regex-ignore-order))) 
```

Ivy is part of a package called `counsel` that includes various commands that enhance these kind of selections.

Ivy isn’t the only tool that does this kind of thing. Indeed the world of completion tools in Emacs is one I find very confusing: lots of tools with overlaps and interactions that I don’t really understand. The tools in this territory include [Helm](https://emacs-helm.github.io/helm/), [company](https://company-mode.github.io/), [Vertico](https://github.com/minad/vertico), and [Consult](https://github.com/minad/consult). Mastering Emacs has an article on [Understanding Minibuffer Completion](https://www.masteringemacs.org/article/understanding-minibuffer-completion), but it doesn’t explain how the mechanisms it talks about fit in with what Ivy does, and I haven’t spent the time to figure it all out.

And as a general note, I strongly recommend the book [Mastering Emacs](https://www.masteringemacs.org/) to learn how to use this incredible tool. Emacs has so many capabilities, that even a decades-old user like me found that book led to “I didn’t know it could do that” moments.

For those that are curious, here’s the relevant bits of my Emacs config

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