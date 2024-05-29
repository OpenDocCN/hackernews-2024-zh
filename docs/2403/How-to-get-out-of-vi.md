<!--yml
category: 未分类
date: 2024-05-27 14:54:18
-->

# How to get out of vi

> 来源：[https://liw.fi/vi/](https://liw.fi/vi/)

For some inexplicable reason people often have trouble getting out of `vi`, the venerable and mighty UNIX editor. Given that the `vi` user interface is logical, and therefore easy to learn, this manual should be unnecessary. The world, however, is not what it should be.

22 September 2002.

If you want to exit and save what is in the buffer, the command sequence is:

> Control-Q Control-C ESC Z Z

If you want to exit, but do not want to save what is in the buffer, the command sequence is:

> Control-Q Control-C ESC : q ! ENTER

Feel free to refer amnesiac `vi` users to this page. Hopefully it will convert them to the elegance and simplicity of the editor.