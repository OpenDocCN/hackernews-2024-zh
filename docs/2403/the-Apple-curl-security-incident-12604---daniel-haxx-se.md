<!--yml
category: 未分类
date: 2024-05-27 14:46:58
-->

# the Apple curl security incident 12604 | daniel.haxx.se

> 来源：[https://daniel.haxx.se/blog/2024/03/08/the-apple-curl-security-incident-12604/](https://daniel.haxx.se/blog/2024/03/08/the-apple-curl-security-incident-12604/)

tldr: Apple thinks it is fine. I do not.

On December 28 2023, [bugreport 12604](https://github.com/curl/curl/issues/12604) was filed in the curl issue tracker. We get a lot issues filed most days so this fact alone was hardly anything out of the ordinary. We read the reports, investigate, ask follow-up questions to see what we can learn and what we need to address.

The title stated of the problem in this case was quite clear: *[flag –cacert behavior isn’t consistent between macOS and Linux](https://github.com/curl/curl/issues/12604)*, and it was filed by Yuedong Wu.

The friendly reporter showed how the curl version bundled with macOS behaves differently than curl binaries built entirely from open source. Even when running the same curl version on the same macOS machine.

The curl command line option `[--cacert](https://curl.se/docs/manpage.html#--cacert)` provides a way for the user to say to curl that **this is the exact set of CA certificates to trust** when doing the following transfer. If the TLS server cannot provide a certificate that can be verified with that set of certificates, it should fail and return error.

This particular behavior and functionality in curl has been established since many years (this option was added to curl in December 2000) and of course is provided to allow users to know that it communicates with a known and trusted server. A pretty fundamental part of what TLS does really.

When this command line option is used with curl on macOS, *the version shipped by Apple*, **it seems to fall back and checks the system CA store in case the provided set of CA certs fail the verification**. A *secondary check* that was not asked for, is not documented and plain frankly comes completely by surprise. Therefore, when a user runs the check with a trimmed and dedicated CA cert file, it will not fail if the system CA store contains a cert that can verify the server!

This is a security problem because now **suddenly certificate checks pass that should not pass.**

I reported this as a security problem in an email sent to Product Security at Apple on December 29 2023, 08:30 UTC. It’s not a major problem, but it is an issue.

## Apple’s says it is fine

On March 8, 2024 Apple Product Security responded with their wisdom:

```
Hello,

Thank you again for reporting this to us and allowing us time to investigate.

Apple’s version of OpenSSL (LibreSSL) intentionally uses the built-in system trust store as a default source of trust. Because the server certificate can be validated successfully using the built-in system trust store, we don't consider this something that needs to be addressed in our platforms.

Best regards,
KC
Apple Product Security
```

Case closed.

## I disagree

Obviously I think differently. This *undocumented feature* makes CA cert verification with curl on macOS totally unreliable and inconsistent with documentation. It tricks users.

Be aware.

Since this is not a security vulnerability in the curl version we ship, we have not issued a CVE or anything for this problem. The problem is strictly speaking not even in curl code. It comes with the version of LibreSSL that Apple ships and builds curl to use on their platforms.

## Discussion

[hacker news](https://news.ycombinator.com/item?id=39650498)