<!--yml

category: 未分类

date: 2024-05-27 13:40:49

-->

# go-sqlite3/vfs/adiantum at main · ncruces/go-sqlite3 · GitHub

> 来源：[https://github.com/ncruces/go-sqlite3/tree/main/vfs/adiantum](https://github.com/ncruces/go-sqlite3/tree/main/vfs/adiantum)

本包包装了一个SQLite VFS，提供了静态加密。

警告

这项工作未经密码学家认证。如果您需要经过验证的加密，您应该购买[SQLite Encryption Extension](https://sqlite.org/see)，并将其包装或寻求帮助包装。

**"adiantum"** VFS wraps the default SQLite VFS using the [Adiantum](https://github.com/lukechampine/adiantum) tweakable and length-preserving encryption.

通常，任何HBSH结构都可以用来包装任何VFS。

默认的Adiantum结构使用XChaCha12作为其流密码，AES作为其块密码，并使用NH和Poly1305作为哈希函数。

此外，我们使用[Argon2id](https://pkg.go.dev/golang.org/x/crypto/argon2#hdr-Argon2id)来从纯文本派生256位密钥。文件内容以4K块加密，与[默认](https://sqlite.org/pgszchng2016.html)SQLite页面大小匹配。

VFS加密所有文件 *除了* [超级日志](https://sqlite.org/tempfiles.html#super_journal_files)：这些文件 *从不* 包含数据库数据，只包含文件名，并且将它们填充到块大小是有问题的。临时文件 *会* 使用**随机**密钥进行加密，因为它们 *可能* 包含数据库数据。为了避免加密临时文件的开销，将它们保留在内存中：

```
PRAGMA temp_store = memory; 
```

重要

Adiantum是用于磁盘加密的密码组合。磁盘加密的标准威胁模型考虑了一个能够读取磁盘多个快照的对手。磁盘加密提供的唯一安全属性是，对手能够获得的所有信息只是一个扇区中的数据是否随时间变化。

本包提供的加密是完全确定性的。

这意味着，一个能够获取数据库文件多个快照（例如备份）的对手可以准确地了解：哪些块发生了改变，哪些没有，哪些被还原。

这比包含*一些*非确定性的其他形式的SQLite加密稍微弱一些；有了有限的非确定性，对手无法区分实际发生变化的块和被还原的块。

注意

本包不声明保护数据库免受篡改或伪造。

上述观点的主要实际后果是，如果您正在保留数据库的 **"adiantum"** 加密备份，并希望防止伪造，您应该签署您的备份，并在恢复之前验证签名。

这比包含块级[MACs](https://en.wikipedia.org/wiki/Message_authentication_code)的其他形式的SQLite加密稍微弱一些。块级MAC可以防止伪造单个块，但不能防止它们被还原到先前版本。
