<!--yml
category: 未分类
date: 2024-05-29 12:47:46
-->

# Contributing to Postgres 101: A Beginner's... | Crunchy Data Blog

> 来源：[https://www.crunchydata.com/blog/contributing-to-postgres-101-a-beginners-experience](https://www.crunchydata.com/blog/contributing-to-postgres-101-a-beginners-experience)

<main class="article prose prose-xl prose-headings:font-display prose-headings:font-bold max-w-none">

I recently got my very first patch into PostgreSQL! To be clear I'm not a C developer and didn't contribute some fancy new feature. However, I do love Postgres and wanted to contribute. Here's my journey and what I learned along the way.

I had an idea for a docs patch while I was talking to Stephen Frost about some research and writing I was doing about [HOT updates and fill factor](https://www.crunchydata.com/blog/postgres-performance-boost-hot-updates-and-fill-factor). A recent update to HOT updates meant HOT could be compatible with BRIN. And while the [HOT readme was up to date,](https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob;f=src/backend/access/heap/README.HOT) the main [PostgreSQL docs](https://www.postgresql.org/docs/current/storage-hot.html) were missing a reference to BRIN indexes, as of Postgres 16, allowing HOT updates. There’s not a ton of missing Postgres docs, so this felt like my chance to put in a teensy weensy little patch in, and as a side bonus I’d have Stephen’s help.

PostgreSQL docs and source code are all packaged together. So what you see at [https://www.postgresql.org/docs/current/](https://www.postgresql.org/docs/current/) is also in the main code repo. This is public and online at [https://github.com/postgres/postgres](https://github.com/postgres/postgres). This code repo is just a mirror of the code, so it doesn't actually run anything and you can’t submit PRs against it. In fact, PostgreSQL doesn’t work with a pull request workflow at all. They work with the concept of a patch file.

A patch file has both the diff information, your changes, as well as metadata at the top.

You can start a normal branch, make your changes, and create a commit. To get the patch file out when you’ve finished your commit, you’ll do this.

```
git format-patch -1 HEAD 
```

And here’s an example of what that output.

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

Ok, I have my patch file. Next step is to ~~create a pull request~~ email that to a mailing list? So yeah, the real magic happens in PostgreSQL through [a mailing list](https://www.postgresql.org/list/). While, it’s not the fanciest new thing out there, email does have the ability to reach pretty much every person on the planet. It is surprisingly efficient at delivering the same information to a variety of people and letting them consume that the time and place they feel is best.

To be clear, I read zero PostgreSQL mailing lists currently. I prefer the excellent and succinct curation done by Cooper Press’ [Postgres weekly](https://postgresweekly.com/). If you want to level up your Postgres my colleague [Craig](https://twitter.com/craigkerstiens) encourages people all the time to subscribe to the lists and simply follow along. The pgsql-general mailing list is more for general Postgres usage, tips, troubleshooting, where as pgsql-hackers is where development and contribution happens. There is also a pgsql-docs mailing list intended for discussion of doc changes, but once a patch exists it needs to make its way to -hackers since that’s where you can create a commitfest record for it. Even if you don’t want to contribute to Postgres the pgsql-hackers mailing list can be a great to level up your broader engineering skills and knowledge Postgres internals.

Luckily, anyone can submit things code patches to hackers@. Since reply all is their default, you can stay in the loop with your own patch without drinking from the rest of the hacker firehose of information.

The [mailing lists are mirrored to the web](https://www.postgresql.org/list/), so [discussions of patches](https://postgr.es/m/CABoUFXRjisr58Ct_3VsFEdQx+fJeQTWTdJnM7XAp=8MUbtoa9A@mail.gmail.com) are viewable, indexed by google, and searchable. You don’t have to keep track of everything in your inbox.

Ok, so you’ve submitted this patch and hoping someone will test it. Anyone on the hackers list might pull in your patch. Testing a patch is as simple as creating a local branch and adding a patch file. Postgres also has [regression tests](https://www.postgresql.org/docs/current/regress-run.html) that should be run against patches. Since mine was a docs patch, I didn’t bother with this. If you’re changing code, you should do this, and the reviewers will as well.

Patches are generally also added to an upcoming [commitfest](https://commitfest.postgresql.org/). Commitfest is an ongoing patch review list to keep track of reviewers, committers, and activity for certain months out of the year. This is a little tracking system that Postgres uses to make sure nothing gets lost on the email threads. The commitfest system also drives the [automated patch testing system](http://cfbot.cputube.org), know as the CF bot. This automated test will run on any patch that's been added to the commitfest app. This will ensure the latest version of your patch will be tested against the latest version of the Postgres code.

Now arguably the most terrifying part of the entire patch submission process is waiting on bated breath for feedback. It could be good, it could be horribly embarrassing, there’s just no way of telling. For all situations of patch review, thick skin is your friend here. Everyone wants this to be the best and most accurate software possible. Patch feedback might seem overwhelming and extremely specific, but keep in mind, this is a 30 year old project. It’s the most popular community run database in the world. The community has quite a standard and a legacy of excellence to uphold.

So for your patch to actually make it into the codebase, [one of the current Postgres committers](https://wiki.postgresql.org/wiki/Committers) has to take up your patch and get it committed to the code base. Commits happen in a git based, private repo. In my case, one of the committers saw the patch and the discussion, offered his own suggestions, and asked for agreement. Once we all agreed, it was committed with a [commit id and note](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commit;h=7a9328e8e40534fb4de41b4ac152e3c6989d84e7).

As soon as your patch is committed, it will appear in the mirror. The current ‘master’ branch of PostgreSQL is generally open for commits in the development version. So master right now is open for version 17, since that will go into public beta this spring. If you get a commit in before the public beta is released, your code will be in that version.

If you happen to have something back patched to an existing version, it will also appear in one of the version stable releases. For example, my patch is applicable to version 16, so it is also in REL_16_STABLE. 16.2 is the current minor version, there will be a new minor point release for 16.3\. When 16.3 is out, my docs update will appear there, since the docs site runs on the current point release. See [Postgres’ roadmap for planned releases](https://www.postgresql.org/developer/roadmap/).

Now for people to actually use your code. That’s a whole other process which requires the magic of the amazing packaging team and then of course the database has to be updated to the newest version.

I cannot end this post without pointing out what a welcoming community PostgreSQL has been for me in the last few years. I’m happy to be riding a wave where PostgreSQL itself and their associated member organizations like [pgUS](https://postgresql.us/) and [PostgreSQL Europe](https://www.postgresql.eu/), are making inclusivity and diversity a priority. I’ve been easily brought into the fold, encouraged, and helped along by many of my co-workers and community members. I should mention a particular shout out to Craig Kerstiens, Stephen Frost, and David Christensen for their support in my PostgreSQL endeavors.

</main>