<!--yml

分类：未分类

日期：2024-05-27 13:02:52

-->

# smeso - MIPS stacktrace: an unexpected journey

> 来源：[https://smeso.it/2024/03/02/mips-stacktrace-an-unexpected-journey.html](https://smeso.it/2024/03/02/mips-stacktrace-an-unexpected-journey.html)

当你的 C 程序崩溃时自动接收堆栈跟踪并不是什么难事。但这一次比我预期的要困难得多。这篇文章简要回顾了几年前我发现的事情。本文假设读者对函数调用约定、CPU 寄存器和汇编有基本了解。

# 一些背景信息

在世界另一端部署的某特定嵌入式设备上，运行在 Linux 上的一个 C 程序偶尔会随机崩溃。该设备采用 MIPS32 架构。我们需要一个系统来异步接收尽可能详细的报告（例如堆栈跟踪）。该设备使用的是*glibc*，理想情况下，我们希望能找到一个也能兼容其他标准库（如*musl*和*uClibc*）的解决方案，但即使是非可移植的解决方案也可以，至少可以解决这一个问题。使用外部依赖尤其是大型依赖并不是一个选择。

# 简单解决方案

*glibc* 已经有了[`backtrace(3)`](https://www.gnu.org/software/libc/manual/html_node/Backtraces.html#index-backtrace-1) 和 [`backtrace_symbols_fd(3)`](https://gnu.org/software/libc/manual/html_node/Backtraces.html#index-backtrace_005fsymbols_005ffd)。它们易于使用且肯定能很好地工作！*glibc* 简单地调用*libgcc*。任何我编写的代码都不会比*libgcc* 更好地理解*GCC* 生成的代码！

好吧，在 x86_64 上可能是真的，但在其他一些架构如 MIPS32 上，这些函数*根本不起作用*。

# MIPS 的一些注释

如果你对 MIPS 不太熟悉，我想添加一些关于它如何在 Linux 上与 gcc 协作的注释。

+   当前函数的返回地址存储在*$ra* 寄存器中

+   当进入一个新函数时，“旧”的返回地址被推送到堆栈上，正好在新函数的堆栈帧开始之前。

要重建堆栈跟踪，我们需要依次恢复所有返回地址。我们可以从*$ra* 开始，然后向后推，从每个堆栈帧的末尾拉出返回地址。

# 问题

原来`backtrace(3)`和互联网上太多博客文章推荐的做法一样：使用*帧指针*寄存器展开堆栈，以找出前一个函数的返回地址的位置。这是完全合理的，*帧指针*（又称*$fp*或MIPS上的*$30*）通常是一个寄存器，用于帮助调试器引用存储在栈上的局部变量和其他信息（例如前一个函数的返回地址），使用常量偏移量。理论上，虽然*堆栈指针*（又称*$sp*）始终指向堆栈顶部，但*$fp*应指向当前*栈帧*的开始，并且不应该从那里移动。如果你想用*$sp*获取返回地址，你需要知道自从进入当前栈帧以来你在栈上放了多少东西，这取决于：你在哪个函数中，这些函数使用了多少自动变量，这些变量的类型是什么，以及这些类型使用了多少字节。在调试过程中使用*$sp*将会非常不方便，所以我们很幸运有了*$fp*！使用*$fp*，我们可以始终获取前一个函数的返回地址，而无需进行任何复杂的操作！对于所有程序中的所有函数来说，*$fp*与返回地址之间的偏移量都是相同的常数！

……还是吗？

好吧……结果证明并非如此。

实际上，在MIPS32上使用GCC的Linux时，*帧指针*就像另一个*堆栈指针*一样工作：它完全没用！我不完全确定背后选择的原因，但我认为这可能与事实有关，在具有（相对）较小寄存器的架构上，使用真实的*$fp*引用栈顶将会很困难，但仍然听起来不像正确的做法。

最有趣的是，来自libgcc的`backtrace(3)`实现似乎忽略了gcc在这个上下文中的工作方式，它只是返回随机值。

# 真正的解决方案

我们可以完全去掉*$fp*，只使用*$sp*，但是我们如何找出在*任何*函数中使用*$sp*时正确的偏移量呢？

你可能不喜欢这个答案（或者也许你会喜欢），但是知道栈帧的起始位置的唯一方法……是跳转到实际函数的代码中，并*解析操作码*来找出编译器减少的*$sp*的数量。

以下是一个执行的方法：

```
`#include  <link.h>
#include  <sys/ucontext.h>

static  inline  void  my_backtrace(ucontext_t*  c)
{
  unsigned  long  *ra;
  unsigned  long  *fp;
  unsigned  long  *sp;
  size_t  ra_offset;
  size_t  stack_size;
  int  reached_start  =  0;
  int  first_time  =  1;

  pc  =  (unsigned  long*)(unsigned  long)c->uc_mcontext.pc;
  ra  =  (unsigned  long*)(unsigned  long)c->uc_mcontext.gregs[31];
  fp  =  (unsigned  long*)(unsigned  long)c->uc_mcontext.gregs[30];
  sp  =  (unsigned  long*)(unsigned  long)c->uc_mcontext.gregs[29];
  print(pc  -  1);
  print(ra  -  1);

  while  (!reached_start)  {
  int  using_fp  =  0;
  ra_offset  =  0;
  stack_size  =  0;
  for  (unsigned  long*  addr  =  ra;
  (ra_offset  ==  0  ||  stack_size  ==  0)  &&  !reached_start;
  --addr)  {
  switch  (*addr  &  0xffff0000)  {
  case  0x27bd0000:
  // found addiu sp, sp, stack_size
  stack_size  =  abs((short)(*addr  &  0xffff));
  break;
  case  0xafbf0000:
  // found sw ra, ra_offset(sp)
  ra_offset  =  (short)(*addr  &  0xffff);
  break;
  case  0x03a00000:
  if  (0x03a0f000  ==  (*addr  &  0xffffff00))  {
  // found pseudo instruction move fp, sp
  using_fp  =  1;
  }
  break;
  case  0x03e00000:
  if  (0x03e00025  ==  *addr)  {
  // found move zero,ra
  // so we found the start
  reached_start  =  1;
  }
  break;
  default:
  break;
  }
  }

  if  (!ra_offset)  {
  break;
  }

  if  (using_fp  &&  first_time)  {
  sp  =  fp;
  }
  first_time  =  0;

  ra  =  *(unsigned  long**)((unsigned  long)sp  +  ra_offset);
  if  (using_fp)  {
  sp  =  *(unsigned  long**)((unsigned  long)sp  +  ra_offset  -  4);
  }  else  {
  sp  =  (unsigned  long*)((unsigned  long)sp  +  stack_size);
  }

  print(ra  -  1);
  }

}` 
```

# 符号化地址

能够将我们刚刚找到的地址翻译成实际的函数名称将是非常好的。这通常不是最有趣的事情，但在我们刚刚做过的事情之后，这似乎微不足道。我们可以使用`dl_iterate_phdr(3)`来查看每个加载的共享对象。一旦找到包含我们地址的共享对象，我们可以遍历其ELF节并查看其*.symtab*以找到函数名称。

这是一个如何做的例子：

```
`struct  file_match  {
  const  char  *file;
  void  *address;
  void  *base;
};

static  int
find_matching_file(struct  dl_phdr_info  *info,
  size_t  size,
  void  *data)
{
  struct  file_match  *match  =  data;
  long  n;
  const  ElfW(Phdr)  *phdr;
  /*  This  code  is  modeled  from  Gfind_proc_info-lsb.c:callback()  from  libunwind  */
  ElfW(Addr)  load_base  =  info->dlpi_addr;
  phdr  =  info->dlpi_phdr;
  for  (n  =  info->dlpi_phnum;  --n  >=  0;  phdr++)  {
  if  (phdr->p_type  ==  PT_LOAD)  {
  ElfW(Addr)  vaddr  =  phdr->p_vaddr  +  load_base;
  if  (match->address  >=  vaddr  &&  match->address  <  vaddr  +  phdr->p_memsz)  {
  match->file  =  info->dlpi_name;
  match->base  =  info->dlpi_addr;
  }
  }
  }
  return  0;
}

static  inline  void
symbolize(const  char*  progname,  void*  addr)
{
  struct  file_match  match  =  {  .address  =  addr  };
  dl_iterate_phdr(find_matching_file,  &match);
  if  (match.file  &&  strlen(match.file))  {
  fprintf(stderr,  " %s(+%p)\n",  match.file,  addr  -  match.base);
  }  else  {
  fprintf(stderr,  " %s\n",  progname);
  }
  fflush(stderr);
}` 
```

如果我们也能获取源文件名和行号，岂不是很酷？要做到这一点，我们需要使用*DWARF*格式中*.debug_line*部分提供的信息。为了尽可能节省空间，*DWARF*并不简单地存储地址到行号的映射列表。它存储了一个*行号程序*，这是一个序列化的有限状态机，可用于查找行号及更多信息。解析*DWARF*留给读者作为练习。或者我们可以手动调用*addr2line*工具。

# 结论

最后bug被发现和修复，所有人都过上了幸福快乐的生活。

我再次学到，功能/缺陷有时可能会发生在您最信任的依赖项中，一个人永远不应该怀疑最受尊敬的软件的正确性。

我还再次学到，互联网上充斥着误导性信息和由从未真正尝试过他们所谈论的事情的人写的带有权威性的博文。截至今天，如果您试图查找有关MIPS32上帧指针的信息，几乎不可能找到任何提到本文中概述问题的信息。所以，永远不要相信博文！甚至不要相信这篇！您的编译器和libc的组合可能会表现出不同的行为，并且具有不同的新的令人兴奋的问题，这将毁掉您的一天！

# 附言

这篇帖子进入了HN的首页，并收到了一些评论！您也可以在[这里](https://news.ycombinator.com/item?id=39967864)添加您的评论。
