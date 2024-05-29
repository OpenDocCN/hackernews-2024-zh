<!--yml
category: 未分类
date: 2024-05-27 14:51:27
-->

# BEEP Core protocol central, RFC 3080, RFC 3081, and related BEEP profiles, BEEP implementation, BEEP Protocol, BEEP, Blocks Extensible Exchange Protocol

> 来源：[https://www.beepcore.org/](https://www.beepcore.org/)

# Welcome to the BEEP community site!

Welcome to BEEP, a network application framework protocol that enables network designers to spend more time in their own protocols rather than solving over and over again the same problem.

On this pages you'll find information about BEEP protocol. Some of the main sections are:

# A little introduction about BEEP

The BEEP protocol is a definition based on the experience about well-known practices in network protocol design. These practices are found in many frequently used network protocols that have been designed over the last 20 years, and are used day by day.

BEEP (Block Extensible Exchange Protocol) includes, in one clear definition, all the features found spread in those protocols but in a coherent, reusable way, that will enable application protocol designers to focus their time on the problem to solve.

BEEP is not a protocol for sending and receiving data directly. Rather, it allows you to define your application protocol on top of it, reusing several mechanisms such as: asynchronous communications, transport layer security, peer authentication, channel multiplexing on the same connection, message framing, channel bandwidth management, and many more interesting network features.

However, you are not required to implement your own application protocol on top of BEEP. This is because there are many network definitions already done on top of BEEP that maybe perform the protocol you are looking for!

From a simple point of view you can easily take before exploring the documentation, BEEP is good because:

*   BEEP allows reusing of several network techniques that you are likely to be required in your implementation.
*   BEEP doesn't enforce you to use an specific data encapsulation format to actually get benefits offered from the framework! You can use the encapsulation format that fits better to your needs, including binary formats.
*   BEEP is defined in one document ([RFC3080](http://www.rfc-editor.org/rfc/rfc3080.txt)) and then mapped into TCP ([RFC3081](http://www.rfc-editor.org/rfc/rfc3081.txt)) which makes clear which is the reference to follow.
*   BEEP only offers you the ability to reuse best-practice network technologies that are known to work, without imposing requirements that fall under the user application space. You'll still keep your project under control.
*   BEEP plays well with other defined protocols because it only defines its role at the transport layer and nothing else, commonly on top of TCP ([RFC3081](http://www.rfc-editor.org/rfc/rfc3081.txt)), allowing to reuse and mix other technologies.

# Help us to keep beepcore.org running

There many on going activities to keep the site up to date, with fresh information about BEEP and its related resources. Mainly, we are looking for people that:

*   Write articles that are relationed with BEEP, including product descriptions, technical articles explaining with more details how should be implemented some kind of BEEP technology, etc.
*   Provide information about products being built with BEEP, new BEEP implementation, and any information that you think it could be interesting for the BEEP community.
*   Help us to keep directory service information up to date, that is: companies, consultants, products and BEEP tool kits.
    If you want your company, product or BEEP implementation to get listed, contact us.

You can reach us at [webmaster@beepcore.org](mailto:webmaster@beepcore.org). If you have something in mind, which doesn't match previous description, but you still think it is an interesting information, contact us.