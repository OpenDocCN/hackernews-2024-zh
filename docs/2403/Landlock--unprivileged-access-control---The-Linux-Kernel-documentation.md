<!--yml

category: 未分类

date: 2024-05-29 12:47:56

-->

# Landlock：非特权访问控制 — Linux内核文档

> 来源：[https://docs.kernel.org/userspace-api/landlock.html](https://docs.kernel.org/userspace-api/landlock.html)

Landlock的目标是允许为一组进程限制环境权限（例如全局文件系统或网络访问）。由于Landlock是可堆叠的LSM，它使得可以创建安全的安全沙盒作为现有系统范围访问控制的新安全层。这种沙盒预期有助于减轻用户空间应用程序中的错误或意外/恶意行为的安全影响。Landlock使得任何进程，包括非特权进程，都能安全地限制自己。

我们可以通过在内核日志中搜索“landlock: Up and running”（作为root用户）来快速确认运行系统中是否启用了Landlock：`dmesg | grep landlock || journalctl -kb -g landlock`。开发者还可以通过[相关系统调用](#landlock-abi-versions)轻松检查Landlock支持情况。如果当前不支持Landlock，则需要[适当配置内核](#kernel-support)。

## Landlock规则

一个Landlock规则描述了进程打算执行的对象上的操作。一组规则聚合在一个规则集中，然后可以限制执行它及其未来子进程的线程。

现有的两种规则类型是：

文件系统规则

对于这些规则，对象是文件层次结构，并且相关的文件系统操作定义了文件系统访问权限。

网络规则（自ABI v4起）

对于这些规则，对象是TCP端口，并且相关的操作使用网络访问权限定义。

### 定义和执行安全策略

我们首先需要定义将包含我们规则的规则集。

作为示例，该规则集将包含仅允许文件系统读取操作并建立特定的TCP连接的规则。将拒绝文件系统写入操作和其他TCP操作。

因此，规则集需要处理这两种类型的操作。这对于向后和向前兼容性是必需的（即内核和用户空间可能不知道对方支持的限制），因此需要明确拒绝默认访问权限的需要。

```
struct  landlock_ruleset_attr  ruleset_attr  =  { .handled_access_fs  = LANDLOCK_ACCESS_FS_EXECUTE  | LANDLOCK_ACCESS_FS_WRITE_FILE  | LANDLOCK_ACCESS_FS_READ_FILE  | LANDLOCK_ACCESS_FS_READ_DIR  | LANDLOCK_ACCESS_FS_REMOVE_DIR  | LANDLOCK_ACCESS_FS_REMOVE_FILE  | LANDLOCK_ACCESS_FS_MAKE_CHAR  | LANDLOCK_ACCESS_FS_MAKE_DIR  | LANDLOCK_ACCESS_FS_MAKE_REG  | LANDLOCK_ACCESS_FS_MAKE_SOCK  | LANDLOCK_ACCESS_FS_MAKE_FIFO  | LANDLOCK_ACCESS_FS_MAKE_BLOCK  | LANDLOCK_ACCESS_FS_MAKE_SYM  | LANDLOCK_ACCESS_FS_REFER  | LANDLOCK_ACCESS_FS_TRUNCATE  | LANDLOCK_ACCESS_FS_IOCTL_DEV, .handled_access_net  = LANDLOCK_ACCESS_NET_BIND_TCP  | LANDLOCK_ACCESS_NET_CONNECT_TCP, }; 
```

因为我们可能不知道应用程序将在哪个内核版本上执行，所以最好采取最佳安全性方法。事实上，我们应该尽量保护用户，无论他们使用哪个内核。

为了兼容旧的Linux版本，我们检测可用的Landlock ABI版本，并且只使用可用的访问权限子集：

```
int  abi; abi  =  landlock_create_ruleset(NULL,  0,  LANDLOCK_CREATE_RULESET_VERSION); if  (abi  <  0)  { /* Degrades gracefully if Landlock is not handled. */ perror("The running kernel does not enable to use Landlock"); return  0; } switch  (abi)  { case  1: /* Removes LANDLOCK_ACCESS_FS_REFER for ABI < 2 */ ruleset_attr.handled_access_fs  &=  ~LANDLOCK_ACCESS_FS_REFER; __attribute__((fallthrough)); case  2: /* Removes LANDLOCK_ACCESS_FS_TRUNCATE for ABI < 3 */ ruleset_attr.handled_access_fs  &=  ~LANDLOCK_ACCESS_FS_TRUNCATE; __attribute__((fallthrough)); case  3: /* Removes network support for ABI < 4 */ ruleset_attr.handled_access_net  &= ~(LANDLOCK_ACCESS_NET_BIND_TCP  | LANDLOCK_ACCESS_NET_CONNECT_TCP); __attribute__((fallthrough)); case  4: /* Removes LANDLOCK_ACCESS_FS_IOCTL_DEV for ABI < 5 */ ruleset_attr.handled_access_fs  &=  ~LANDLOCK_ACCESS_FS_IOCTL_DEV; } 
```

这使得可以创建一个包含我们规则的包容性规则集。

```
int  ruleset_fd; ruleset_fd  =  landlock_create_ruleset(&ruleset_attr,  sizeof(ruleset_attr),  0); if  (ruleset_fd  <  0)  { perror("Failed to create a ruleset"); return  1; } 
```

现在，我们可以通过返回的文件描述符向此规则集添加新规则。该规则将只允许读取文件层次结构`/usr`。没有另一条规则的话，规则集将拒绝写入操作。要将`/usr`添加到规则集中，我们使用`O_PATH`标志打开它，并用这个文件描述符填充`&[`struct landlock_path_beneath_attr`](#c.landlock_path_beneath_attr "landlock_path_beneath_attr")。

```
int  err; struct  landlock_path_beneath_attr  path_beneath  =  { .allowed_access  = LANDLOCK_ACCESS_FS_EXECUTE  | LANDLOCK_ACCESS_FS_READ_FILE  | LANDLOCK_ACCESS_FS_READ_DIR, }; path_beneath.parent_fd  =  open("/usr",  O_PATH  |  O_CLOEXEC); if  (path_beneath.parent_fd  <  0)  { perror("Failed to open file"); close(ruleset_fd); return  1; } err  =  landlock_add_rule(ruleset_fd,  LANDLOCK_RULE_PATH_BENEATH, &path_beneath,  0); close(path_beneath.parent_fd); if  (err)  { perror("Failed to update ruleset"); close(ruleset_fd); return  1; } 
```

可能还需要创建遵循与规则集创建相同逻辑的规则，通过根据Landlock ABI版本过滤访问权限来筛选。在这个示例中，这不是必需的，因为所有请求的`allowed_access`权限已经在ABI 1中可用。

对于网络访问控制，我们可以添加一组规则，允许使用特定操作的端口号：HTTPS连接。

```
struct  landlock_net_port_attr  net_port  =  { .allowed_access  =  LANDLOCK_ACCESS_NET_CONNECT_TCP, .port  =  443, }; err  =  landlock_add_rule(ruleset_fd,  LANDLOCK_RULE_NET_PORT, &net_port,  0); 
```

下一步是限制当前线程获取更多特权（例如通过SUID二进制文件）。现在我们有一个规则集，第一条规则允许读取`/usr`，同时拒绝文件系统的所有其他受控访问，第二条规则允许HTTPS连接。

```
if  (prctl(PR_SET_NO_NEW_PRIVS,  1,  0,  0,  0))  { perror("Failed to restrict privileges"); close(ruleset_fd); return  1; } 
```

当前线程现在准备使用规则集进行沙盒化。

```
if  (landlock_restrict_self(ruleset_fd,  0))  { perror("Failed to enforce ruleset"); close(ruleset_fd); return  1; } close(ruleset_fd); 
```

如果`landlock_restrict_self`系统调用成功，当前线程现在被限制，并且此策略也将强制执行到所有随后创建的子线程上。一旦线程被Landlock，就没有办法移除其安全策略；只允许添加更多限制。这些线程现在位于一个新的Landlock域中，合并了它们的父域（如果有的话）和新的规则集。

完整的工作代码可以在[samples/landlock/sandboxer.c](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/samples/landlock/sandboxer.c)中找到。

### 良好实践

建议将文件层次结构的访问权限设置得尽可能细致。例如，最好将`~/doc/`作为只读层次结构，将`~/tmp/`作为读写层次结构，而不是将`~/`作为只读层次结构，将`~/tmp/`作为读写层次结构。遵循这一良好实践可以形成自给自足的层次结构，不依赖于它们的位置（即父目录）。当我们希望允许链接或重命名时，这一点尤为重要。确实，每个目录的一致访问权限使得能够更改此类目录的位置而无需依赖目标目录的访问权限（除了执行此操作所需的权限外，参见`LANDLOCK_ACCESS_FS_REFER`文档）。

拥有自给自足的层次结构还有助于将所需访问权限紧密限制在最小数据集上。这也有助于避免“黑洞目录”，即可以将数据链接到但无法从中链接的目录。然而，这取决于数据组织，可能并非由开发者控制。在这种情况下，授予对 `~/tmp/` 的读写访问权限，而不是仅写入访问权限，可能允许将 `~/tmp/` 移动到不可读目录，并仍然保持列出 `~/tmp/` 内容的能力。

### 文件路径访问权限的层次

每当一个线程对自身强制执行规则集时，它会使用新的策略层更新其 Landlock 域。实际上，这个补充策略与已经限制该线程的潜在其他规则集堆叠在一起。因此，一个沙盒线程可以安全地通过一个新的强制执行的规则集向自身添加更多约束。

如果在路径上遇到的规则中至少有一个规则授予访问权限，则一个策略层将授予对文件路径的访问权限。一个沙盒线程只能在所有其强制执行的策略层授予访问权限的情况下访问文件路径，以及所有其他系统访问控制（例如文件系统 DAC、其他 LSM 策略等）。

### 绑定挂载和 OverlayFS

Landlock 允许限制对文件层次结构的访问权限，这意味着这些访问权限可以通过绑定挂载（参见 [共享子树](../filesystems/sharedsubtree.html)）传播，但不能通过 [Overlay 文件系统](../filesystems/overlayfs.html) 传播。

绑定挂载将源文件层次结构镜像到目标。然后，目标层次结构由完全相同的文件组成，可以通过源路径或目标路径绑定 Landlock 规则。这些规则在路径上遇到时限制访问，这意味着它们可以同时限制对多个文件层次结构的访问，无论这些层次结构是否是绑定挂载的结果。

一个 OverlayFS 挂载点由上层和下层组成。这些层被合并在一个合并目录中，这是挂载点的结果。这个合并层次结构可以包括来自上层和下层的文件，但在合并层次结构上进行的修改只会反映在上层。从 Landlock 策略的角度来看，每个 OverlayFS 层和合并层次结构都是独立的，包含它们自己的文件和目录，这与绑定挂载不同。限制 OverlayFS 层不会限制结果的合并层次结构，反之亦然。因此，Landlock 用户应该只考虑他们想要允许访问的文件层次结构，而不考虑底层文件系统。

### 继承

每个从*clone(2)*产生的新线程从其父进程继承Landlock域限制。这类似于seccomp继承（参见[Seccomp BPF（带过滤器的安全计算）](seccomp_filter.html)）或任何处理任务*credentials(7)*的其他LSM。例如，一个进程的线程可以对自身应用Landlock规则，但这些规则不会自动应用于其他同级线程（与POSIX线程凭据更改不同，参见*nptl(7)*）。

当线程自我沙盒化时，我们保证相关的安全策略将在所有此线程的后代上保持强制执行。这允许为每个应用程序创建独立和模块化的安全策略，这些策略将根据其运行时父策略自动组合。

### Ptrace 限制

一个沙盒化的进程权限比非沙盒化的进程少，当操纵另一个进程时必须遵守额外的限制。为了允许在目标进程上使用*ptrace(2)*及其相关系统调用，沙盒化进程应该具有目标进程规则的子集，这意味着被跟踪者必须在追踪者的子域中。

### 截断文件

`LANDLOCK_ACCESS_FS_WRITE_FILE`和`LANDLOCK_ACCESS_FS_TRUNCATE`涵盖的操作都会更改文件的内容，并且有时在非直观的方式下重叠。建议始终同时指定这两者。

一个特别令人惊讶的例子是*creat(2)*。名称表明这个系统调用需要创建和写入文件的权限。然而，如果同名的现有文件已经存在，它还需要截断权限。

还应注意，截断文件不需要`LANDLOCK_ACCESS_FS_WRITE_FILE`权限。除了*truncate(2)*系统调用外，还可以通过使用`O_RDONLY | O_TRUNC`标志的*open(2)*来完成。

截断权限与打开的文件关联（见下文）。

### 与文件描述符关联的权限

当打开文件时，`LANDLOCK_ACCESS_FS_TRUNCATE`和`LANDLOCK_ACCESS_FS_IOCTL_DEV`权限的可用性与新创建的文件描述符相关联，并将用于后续使用*ftruncate(2)*和*ioctl(2)*进行截断和ioctl尝试。行为类似于打开文件进行读取或写入时，在*open(2)*期间检查权限，但在随后的*read(2)*和*write(2)*调用期间不检查。

因此，进程可能具有多个指向同一文件的打开文件描述符，但Landlock在操作这些文件描述符时强制执行不同的规则。当Landlock规则集得到强制执行并且进程保留在强制执行之前和之后打开的文件描述符时，情况可能会如此。甚至可以在涉及的某些进程没有强制执行Landlock规则集的情况下，在进程之间传递这些文件描述符并保持它们的Landlock属性。

## 当前限制

### 文件系统拓扑修改

通过文件系统限制隔离的线程无法通过*mount(2)*或*pivot_root(2)*修改文件系统拓扑结构。然而，*chroot(2)*调用不受限制。

### 特殊文件系统

按照规则集处理的访问方式，可以通过Landlock限制对常规文件和目录的访问。然而，并非来自用户可见文件系统的文件（如管道、套接字），但仍可通过`/proc/<pid>/fd/*`访问，目前无法显式限制。同样地，一些特殊的内核文件系统如nsfs，可以通过`/proc/<pid>/ns/*`访问，目前也无法显式限制。然而，由于[ptrace限制](#ptrace-restrictions)，对这些敏感的`/proc`文件的访问根据域层次结构自动限制。未来的Landlock发展可能会通过专用规则集标志显式限制这些路径。

### 规则集层级

最多可以叠加16层规则集。对于希望在其16个继承规则集之外强制执行新规则集的任务，这可能是一个问题。一旦达到此限制，[`sys_landlock_restrict_self()`](#c.sys_landlock_restrict_self "sys_landlock_restrict_self")将返回E2BIG。因此，强烈建议在线程生命周期内仔细构建规则集，特别是对于能够启动其他应用程序并可能希望自我隔离的应用程序（如shell、容器管理器等）。

### IOCTL支持

`LANDLOCK_ACCESS_FS_IOCTL_DEV`权限限制了对*ioctl(2)*的使用，但仅适用于*新打开的*设备文件。这意味着像stdin、stdout和stderr这样的预存在的文件描述符不受影响。

用户应该意识到，TTY设备传统上允许通过`TIOCSTI`和`TIOCLINUX` IOCTL命令控制同一TTY上的其他进程。在现代Linux系统上，这两者都需要`CAP_SYS_ADMIN`权限，但`TIOCSTI`的行为可配置。

因此，建议在旧系统上关闭继承的TTY文件描述符，或者如果可能的话，从`/proc/self/fd/*`重新打开它们，而无需`LANDLOCK_ACCESS_FS_IOCTL_DEV`权限。

目前，Landlock的IOCTL支持粗粒度化，但未来可能变得更细粒度。因此，建议用户通过文件层次结构建立他们需要的保证，只在确实需要的文件上允许`LANDLOCK_ACCESS_FS_IOCTL_DEV`权限。

## 内核支持

### 构建时配置

Landlock首次引入于Linux 5.13，但必须在构建时配置`CONFIG_SECURITY_LANDLOCK=y`。Landlock还必须像其他安全模块一样在引导时启用。默认情况下启用的安全模块列表由`CONFIG_LSM`设置。然后，内核配置应包含`CONFIG_LSM=landlock,[...]`，其中`[...]`是运行系统的其他可能有用的安全模块列表（参见`CONFIG_LSM`帮助）。

### 引导时间配置

如果运行的内核没有在`CONFIG_LSM`中包含`landlock`，那么我们可以通过在引导加载程序配置中的[内核命令行参数](../admin-guide/kernel-parameters.html)添加`lsm=landlock,[...]`来启用Landlock。

例如，如果当前内置配置是：

```
$ zgrep -h "^CONFIG_LSM=" "/boot/config-$(uname -r)" /proc/config.gz 2>/dev/null
CONFIG_LSM="lockdown,yama,integrity,apparmor" 
```

……如果命令行中也没有包含`landlock`：

```
$ sed -n 's/.*\(\<lsm=\S\+\).*/\1/p' /proc/cmdline
lsm=lockdown,yama,integrity,apparmor 
```

……我们应该配置引导加载程序以设置一个命令行，将`lsm`列表扩展为`landlock,`前缀：

```
lsm=landlock,lockdown,yama,integrity,apparmor 
```

重启后，我们可以通过查看内核日志来检查Landlock是否正常运行：

```
# dmesg | grep landlock || journalctl -kb -g landlock
[    0.000000] Command line: [...] lsm=landlock,lockdown,yama,integrity,apparmor
[    0.000000] Kernel command line: [...] lsm=landlock,lockdown,yama,integrity,apparmor
[    0.000000] LSM: initializing lsm=lockdown,capability,landlock,yama,integrity,apparmor
[    0.000000] landlock: Up and running. 
```

内核可以在构建时配置为始终加载`lockdown`和`capability` LSMs。在这种情况下，即使它们在引导加载程序中未配置，这些LSM也将出现在`LSM: initializing`日志行的开头。

### 网络支持

要能够显式允许TCP操作（例如，使用`LANDLOCK_ACCESS_NET_BIND_TCP`添加网络规则），内核必须支持TCP（`CONFIG_INET=y`）。否则，[`sys_landlock_add_rule()`](#c.sys_landlock_add_rule "sys_landlock_add_rule")将返回`EAFNOSUPPORT`错误，可以安全地忽略，因为这种TCP操作已经不可能。
