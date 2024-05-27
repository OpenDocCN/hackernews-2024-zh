<!--yml
category: 未分类
date: 2024-05-27 13:40:49
-->

# go-sqlite3/vfs/adiantum at main · ncruces/go-sqlite3 · GitHub

> 来源：[https://github.com/ncruces/go-sqlite3/tree/main/vfs/adiantum](https://github.com/ncruces/go-sqlite3/tree/main/vfs/adiantum)

This package wraps an SQLite VFS to offer encryption at rest.

Warning

This work was not certified by a cryptographer. If you need vetted encryption, you should purchase the [SQLite Encryption Extension](https://sqlite.org/see), and either wrap it, or seek assistance wrapping it.

The `"adiantum"` VFS wraps the default SQLite VFS using the [Adiantum](https://github.com/lukechampine/adiantum) tweakable and length-preserving encryption.
In general, any HBSH construction can be used to wrap any VFS.

The default Adiantum construction uses XChaCha12 for its stream cipher, AES for its block cipher, and NH and Poly1305 for hashing.
Additionally, we use [Argon2id](https://pkg.go.dev/golang.org/x/crypto/argon2#hdr-Argon2id) to derive 256-bit keys from plain text where needed. File contents are encrypted in 4K blocks, matching the [default](https://sqlite.org/pgszchng2016.html) SQLite page size.

The VFS encrypts all files *except* [super journals](https://sqlite.org/tempfiles.html#super_journal_files): these *never* contain database data, only filenames, and padding them to the block size is problematic. Temporary files *are* encrypted with **random** keys, as they *may* contain database data. To avoid the overhead of encrypting temporary files, keep them in memory:

```
PRAGMA temp_store = memory; 
```

Important

Adiantum is a cipher composition for disk encryption. The standard threat model for disk encryption considers an adversary that can read multiple snapshots of a disk. The only security property that disk encryption provides is that all information such an adversary can obtain is whether the data in a sector has or has not changed over time.

The encryption offered by this package is fully deterministic.

This means that an adversary who can get ahold of multiple snapshots (e.g. backups) of a database file can learn precisely: which blocks changed, which ones didn't, which got reverted.

This is slightly weaker than other forms of SQLite encryption that include *some* nondeterminism; with limited nondeterminism, an adversary can't distinguish between blocks that actually changed, and blocks that got reverted.

Caution

This package does not claim protect databases against tampering or forgery.

The major practical consequence of the above point is that, if you're keeping `"adiantum"` encrypted backups of your database, and want to protect against forgery, you should sign your backups, and verify signatures before restoring them.

This is slightly weaker than other forms of SQLite encryption that include block-level [MACs](https://en.wikipedia.org/wiki/Message_authentication_code). Block-level MACs can protect against forging individual blocks, but can't prevent them from being reverted to former versions of themselves.