<!--yml

category: 未分类

date: 2024-05-27 15:16:58

-->

# 试图使 sudo 对 ROWHAMMER 攻击更不易受到攻击。 · sudo-project/sudo@7873f83 · GitHub

> 来源：[`github.com/sudo-project/sudo/commit/7873f8334c8d31031f8cfa83bd97ac6029309e4f`](https://github.com/sudo-project/sudo/commit/7873f8334c8d31031f8cfa83bd97ac6029309e4f)

<include-fragment class="js-notification-shelf-include-fragment" data-base-src="https://github.com/notifications/beta/shelf"></include-fragment>

<main id="js-repo-pjax-container"> <turbo-frame id="repo-content-turbo-frame" target="_top" data-turbo-action="advance" class="">## 提交

永久链接

此提交不属于此存储库上的任何分支，可能属于存储库外的一个分支。

试图使 sudo 对 ROWHAMMER 攻击更不易受到攻击。

浏览文件 <tool-tip id="tooltip-9f617571-a394-4c33-bd26-a9aa66f225b0" for="browse-at-time-link" popover="manual" data-direction="ne" data-type="description" data-view-component="true" class="sr-only position-absolute">在历史记录中的这一点浏览存储库</tool-tip>

```
We now use ROWHAMMER-resistent values for ALLOW, DENY, AUTH_SUCCESS,
AUTH_FAILURE, AUTH_ERROR and AUTH_NONINTERACTIVE.  In addition, we
explicitly test for expected values instead of using a negated test
against an error value.  In the parser match functions this means
explicitly checking for ALLOW or DENY instead of accepting anything
that is not set to UNSPEC.

Thanks to Andrew J. Adiletta, M. Caner Tol, Yarkin Doroz, and Berk
Sunar, all affiliated with the Vernam Applied Cryptography and
Cybersecurity Lab at Worcester Polytechnic Institute, for the report.
Paper preprint: https://arxiv.org/abs/2309.02545
```

<include-fragment src="/sudo-project/sudo/branch_commits/7873f8334c8d31031f8cfa83bd97ac6029309e4f" id="async-branches-list">*   加载分支信息</include-fragment> <diff-layout></diff-layout></turbo-frame></main>

<ghcc-consent id="ghcc" class="position-fixed bottom-0 left-0" data-initial-cookie-consent-allowed="" data-cookie-consent-required="false">你目前无法执行此操作。

<template id="site-details-dialog"></template><template id="snippet-clipboard-copy-button"></template><template id="snippet-clipboard-copy-button-unpositioned"></template></ghcc-consent>
