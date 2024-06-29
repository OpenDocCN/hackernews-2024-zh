<!--yml

category: 未分类

date: 2024-05-29 12:47:46

-->

# Contributing to Postgres 101: A Beginner's... | Crunchy Data Blog

> 来源：[https://www.crunchydata.com/blog/contributing-to-postgres-101-a-beginners-experience](https://www.crunchydata.com/blog/contributing-to-postgres-101-a-beginners-experience)

<main class="article prose prose-xl prose-headings:font-display prose-headings:font-bold max-w-none">

我最近在 PostgreSQL 中提交了我的第一个补丁！明确地说，我不是一名 C 开发者，也没有贡献出一些花哨的新功能。不过，我确实热爱 Postgres 并希望做出贡献。这是我的经历以及我在这个过程中学到的东西。

当我与 Stephen Frost 谈论我正在进行的有关 [HOT 更新和填充因子](https://www.crunchydata.com/blog/postgres-performance-boost-hot-updates-and-fill-factor) 的研究和撰写时，我对文档补丁有了一个想法。HOT 更新的最新更新意味着 HOT 可以与 BRIN 兼容。尽管 [HOT 的 readme 已经更新，](https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob;f=src/backend/access/heap/README.HOT) 但是 [主 PostgreSQL 文档](https://www.postgresql.org/docs/current/storage-hot.html) 缺少有关 BRIN 索引的引用，至少在 Postgres 16 时，允许 HOT 更新。Postgres 文档并不缺少很多内容，所以这感觉像是我放入一个微小补丁的机会，作为一个额外的好处，我会得到 Stephen 的帮助。

PostgreSQL 文档和源代码都打包在一起。因此，您在 [https://www.postgresql.org/docs/current/](https://www.postgresql.org/docs/current/) 看到的内容也在主代码库中。这是公开的，并在线上可访问 [https://github.com/postgres/postgres](https://github.com/postgres/postgres)。这个代码库只是代码的镜像，因此实际上并不运行任何内容，您无法提交 PR。事实上，PostgreSQL 根本不使用拉取请求工作流程，他们使用补丁文件的概念。

一个补丁文件包含了差异信息、您的更改，以及顶部的元数据。

当您完成提交后，您可以开始一个普通分支，进行您的更改，并创建一个提交。完成提交后，要获得补丁文件，您需要执行以下操作。

```
git format-patch -1 HEAD 
```

这里是输出的一个例子。

<details><summary>v1-0001-Adding-Summary-Index-Info-to-HOT-Update-Documentatio.patch</summary>

```
From: elizabeth-christensen <elizabethannegarrett@gmail.com>
Date: Mon, 26 Feb 2024 12:09:22 -0600
Subject: [PATCH] Adding Summary Index Info to HOT Update Documentation page

Commit 19d8e2308b changed when the HOT update optimization is possible but neglected to update the Heap-Only Tuples (HOT) documentation.  This patch updates that documentation accordingly.

Author: Elizabeth Christensen
Backpatch-through: 16
Discussion: 
---
 doc/src/sgml/storage.sgml | 18 ++++++++++--------
 1 file changed, 10 insertions(+), 8 deletions(-)

diff --git a/doc/src/sgml/storage.sgml b/doc/src/sgml/storage.sgml
index 3ea4e5526d..314469706e 100644
--- a/doc/src/sgml/storage.sgml
+++ b/doc/src/sgml/storage.sgml
@@ -1097,8 +1097,10 @@ data. Empty in ordinary tables.</entry>
   <itemizedlist>
    <listitem>
     <para>
-     The update does not modify any columns referenced by the table's
-     indexes, including expression and partial indexes.
+     The update only modifies columns which are un-indexed or are only indexed with 
+     summarizing indexes (such as BRIN) and does not update any columns referenced 
+     by the table's non-summary indexes, including expression and partial 
+     non-summary indexes.
      </para>
    </listitem>
    <listitem>
@@ -1114,7 +1116,8 @@ data. Empty in ordinary tables.</entry>
   <itemizedlist>
    <listitem>
     <para>
-     New index entries are not needed to represent updated rows.
+     New index entries are not needed to represent updated rows, however, 
+     summary indexes may still need to be updated.
     </para>
    </listitem>
    <listitem>
@@ -1130,11 +1133,10 @@ data. Empty in ordinary tables.</entry>
  </para>

  <para>
-  In summary, heap-only tuple updates can only be created
-  if columns used by indexes are not updated.  You can
-  increase the likelihood of sufficient page space for
-  <acronym>HOT</acronym> updates by decreasing a table's <link
-  linkend="reloption-fillfactor"><literal>fillfactor</literal></link>.
+  In summary, heap-only tuple updates can only be created if columns used by 
+  non-summary indexes are not updated. You can increase the likelihood of 
+  sufficient page space for <acronym>HOT</acronym> updates by decreasing 
+  a table's <link linkend="reloption-fillfactor"><literal>fillfactor</literal></link>.
   If you don't, <acronym>HOT</acronym> updates will still happen because
   new rows will naturally migrate to new pages and existing pages with
   sufficient free space for new row versions.  The system view <link
-- 
2.37.0 
```</details>

好的，我有了我的补丁文件。下一步是将它发送到邮件列表？所以，PostgreSQL 中的真正魔力发生在 [邮件列表](https://www.postgresql.org/list/)。虽然它不是最新潮的新东西，但电子邮件确实有能力将同样的信息高效地传达给各种人，并让他们在自己感觉最佳的时间和地点消化这些信息。

明确一点，我目前根本不阅读任何PostgreSQL邮件列表。我更喜欢由Cooper Press的[Postgres周刊](https://postgresweekly.com/)精心策划的优秀和简洁的内容。如果您想提升您的Postgres水平，我的同事[Craig](https://twitter.com/craigkerstiens)一直鼓励人们订阅列表并简单地跟进。pgsql-general邮件列表更多用于一般的Postgres使用、技巧、故障排除，而pgsql-hackers则是开发和贡献的地方。还有一个pgsql-docs邮件列表用于讨论文档更改，但一旦存在补丁，它就需要通过-hackers进入，因为那里可以为其创建一个提交节记录。即使您不想为Postgres做贡献，pgsql-hackers邮件列表也可以帮助您提升更广泛的工程技能和Postgres内部知识。

幸运的是，任何人都可以将代码补丁提交到 hackers@。由于他们默认使用“回复所有”，因此您可以通过自己的补丁保持联系，而无需从其他黑客的信息火车中获取信息。

[邮件列表已镜像到网上](https://www.postgresql.org/list/)，因此[补丁讨论](https://postgr.es/m/CABoUFXRjisr58Ct_3VsFEdQx+fJeQTWTdJnM7XAp=8MUbtoa9A@mail.gmail.com)可查看、被谷歌索引和搜索。您无需在收件箱中跟踪所有信息。

好吧，所以您提交了这个补丁，希望有人会测试它。任何在黑客列表上的人都可能拉取您的补丁。测试补丁就像创建一个本地分支并添加补丁文件一样简单。Postgres还有[回归测试](https://www.postgresql.org/docs/current/regress-run.html)，应对补丁运行。由于我的是文档补丁，我没有费心去做这个。如果您正在更改代码，您应该这样做，审阅者也会这样做。

补丁通常也会添加到即将到来的[提交节](https://commitfest.postgresql.org/)中。提交节是一个持续的补丁审查列表，用于跟踪审阅人员、提交者和一年中某些月份的活动。这是Postgres用来确保电子邮件线程上没有任何遗漏的小型跟踪系统。提交节系统还推动[自动补丁测试系统](http://cfbot.cputube.org)，也被称为CF bot。这个自动化测试将运行任何已添加到提交节应用程序中的补丁。这将确保您的补丁的最新版本将与Postgres代码的最新版本进行测试。

现在，整个补   现在，整个补丁提交过程最让人紧张的部分无疑是期待反馈的时刻。结果可能很好，也可能非常尴尬，完全无法预料。对于所有的补丁审核情况，厚脸皮是你的朋友。每个人都希望这是最好的、最准确的软件。补丁反馈看起来可能会让人感到不堪重负且极其具体，但请记住，这是一个30年的项目。这是世界上最受欢迎的社区运营数据库。社区有着相当的标准和卓越的传承需要维护。

因此，为了让你的补丁真正进入代码库，[当前的 Postgres 提交者](https://wiki.postgresql.org/wiki/Committers)必须接受你的补丁并将其提交到代码库中。提交是在基于 git 的私有仓库中进行的。在我这儿，提交者看到补丁和讨论，提出了自己的建议，并请求大家的同意。一旦大家都同意了，就会提交带有[提交 ID 和备注](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commit;h=7a9328e8e40534fb4de41b4ac152e3c6989d84e7)的补丁。

一旦你的补丁被提交，它将出现在镜像中。PostgreSQL 当前的 ‘master’ 分支通常对开发版本的提交是开放的。因此，master 现在对版本17是开放的，因为它将在今年春天进入公开 beta。如果你在公开 beta 发布前提交了代码，你的代码将会出现在那个版本中。

如果你有某个补丁回退到现有版本，它也会出现在某个版本的稳定发布中。例如，我的补丁适用于版本16，所以它也在REL_16_STABLE中。16.2是当前的次要版本，16.3会有一个新的次要版本发布。当16.3发布时，我的文档更新会出现在那里，因为文档网站运行在当前的点版本上。请参见[Postgres 的发布计划](https://www.postgresql.org/developer/roadmap/)。

现在，实际让人们使用你的代码。那是一个完全不同的过程，需要神奇的包装团队的支持，然后数据库必须更新到最新版本。

我不能在这篇文章中结束而不指出，过去几年中 PostgreSQL 对我来说是一个多么包容的社区。我很高兴能参与其中，看到 PostgreSQL 本身以及它的关联成员组织如[pgUS](https://postgresql.us/)和[PostgreSQL Europe](https://www.postgresql.eu/)，都把包容性和多样性作为优先事项。我轻松地融入了这个大家庭，得到了许多同事和社区成员的鼓励和帮助。我还特别要感谢 Craig Kerstiens、Stephen Frost 和 David Christensen，他们在我 PostgreSQL 的努力中提供了支持。

</main>
