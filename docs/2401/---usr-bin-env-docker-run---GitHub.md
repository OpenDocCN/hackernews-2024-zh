<!--yml

分类：未分类

日期：2024-05-27 14:47:13

-->

# #!/usr/bin/env docker run · GitHub

> 来源：[`gist.github.com/adtac/595b5823ef73b329167b815757bbce9f`](https://gist.github.com/adtac/595b5823ef73b329167b815757bbce9f)

`#!`（读作 shebang）是 Unix 的一个惯例，通常用于像 Python 和 Bash 这样的脚本语言。这里滥用它来让你以跨发行版和跨平台的方式打包应用程序。这个示例的 Dockerfile 是一个包含后端、数据库和 UI 的全栈服务器，所有内容都在一个单独的文件中。

这有点像 [Cosmopolitan Libc](https://github.com/jart/cosmopolitan)，但是用于打包应用程序。

为什么不呢？

可能不。

如果你想的话。

或许。

不。

是的。

```
chmod +x ./Dockerfile
./Dockerfile
```

然后前往 [`127.0.0.1:8080`](http://127.0.0.1:8080)
