<!--yml
category: 未分类
date: 2024-05-27 14:54:34
-->

# How to commit part of file in git · Newbeelearn

> 来源：[https://newbeelearn.com/blog/git-commit-part-of-file/](https://newbeelearn.com/blog/git-commit-part-of-file/)

In [Magit](https://magit.vc/), the Git interface for Emacs, you can commit part of a file using the following commands:

**Stage Specific Hunks:**

Explanation: This command stages specific hunks (chunks of changes) within the file, allowing you to selectively include changes in the next commit.

You are not limited to a single file either, if you have multiple changes in multiple files you can select combination of hunks using the same procedure.

**Stage Specific Lines:**

Explanation: You can choose to stage specific lines of code by first selecting them. Start the selection by placing the cursor on the initial line and pressing Ctrl-SPC. Then, move the cursor to encompass the lines you wish to stage. After selecting the desired lines, use the staging command (s) to include their changes in the commit.

You are not limited to a single line, if you have multiple lines in multiple files you can select combination of lines using the same procedure. If the change encompasses multiple lines you can select region as well and use same staging procedure to stage regions.