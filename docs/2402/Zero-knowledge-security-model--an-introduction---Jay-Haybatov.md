<!--yml
category: 未分类
date: 2024-05-27 14:39:16
-->

# Zero-knowledge security model: an introduction - Jay Haybatov

> 来源：[https://haybatov.com/blog/zksm-intro/](https://haybatov.com/blog/zksm-intro/)

# Zero-knowledge security model: an introduction

##### Posted on February 6, 2024  •  5 minutes  • 939 words

News about ransom demands from cyber-criminals often features 7- and 8-digit numbers. Massive amounts of personal data and sensitive information from data leaks get sold or openly published on the internet. All of these can be stopped.

The zero-knowledge security model (ZKSM) is designed to guard sensitive and protected data even in cases of full access to the network infrastructure and/or cloud services. The ZKSM employs multiple techniques to prevent data theft by unauthorised parties:

*   Not keeping data in online systems

*   Encrypting sensitive data at rest, in transit, and in use (at work)

    *   Uninterrupted security

        *   Data must be stored and operated on network infrastructure and servers in encrypted form at all times.
        *   No data decryption must be possible on servers or network infrastructure.
    *   Data ownership

        *   Keys for decrypting data must belong to the data owner.
        *   Only data owners and users authorised by the owners can decrypt data, and only in secure environments (e.g., secured end-user devices, secure enclaves)
    *   Data independence

        *   Each separate entity (e.g., file, document, data stream, videocall) must use its own unique encryption key.
        *   Every user and service account must have their own keys.
*   Sharing keys and data securely

    *   Nothing open

        *   All data and all encryption keys must be shared in encrypted form (e.g., no links with keys).
    *   Nothing trusted

        *   Keys and digital signatures must be correctly validated (e.g., to prevent replay attacks or encrypted data manipulation).
    *   Nothing permanent

        *   Sharing must be revokable and renewable (e.g., when one or more parties must lose access to a document or a new version of the document).

ZKSM guarantees the following properties for data:

*   Integrity
*   Confidentiality
*   Authenticity
*   Non-repudiation

In realistic scenarios, the requirements beyond the ZKSM include:

*   Fully traceable audit trail of sharing and downloading sensitive data.
*   Authentication of users before any access to sensitive data.
*   Simplicity and visibility of sharing to prevent human mistakes.
*   Availability of data to prevent downtimes.
*   Data sovereignty for critical infrastructure and national security.

Fully homomorphic encryption (FHE) – a technique that allows performing mathematical operations directly on encrypted data – can be optionally used for data processing. Modern performant FHE systems rely on approximate operations (e.g., get close but not exact results of multiplication), which limits their universal adoption. Despite this, 4^th^ generation FHE has many useful applications in secure data processing.

Often, companies advertise their end-to-end encrypted products as having ‘zero-knowledge’ properties. While end-to-end encryption satisfies some requirements of the ZKSM, it is only a part of the requirements. For example, all E2EE file sharing platforms, with a notable exception, use links for data sharing. This approach violates several ZKSM principles:

*   Keys are not shared in a secure manner (often, they are just appended to a link to the file).
*   Unauthorised users can get access to data (e.g., when a link is shared via email or other means).
*   Revoking access or the access keys is impossible without deleting the shared file – hardly a practical solution.

Before delving into the details of the ZKSM, the designer of a ZKSM system has a few considerations to make:

*   Define sensitive data.
    *   Not all data is sensitive and thus need the same level of protection.
    *   Implementing the ZKSM is not free and should not be applied as a blanket rule.
*   Know your data processes.
    *   ZKSM severely limits access to data on the server side. Your processes need to map to the new ways of searching, processing, and updating data.
    *   ZKSM must be applied to the entire system at the architectural stage and cannot be an optional add-on. At the same time, the most sensitive data pipelines (for PII and protected/secret records, etc.) can adopt the ZKSM quickly first; the rest of the system can follow in the new secure design.
*   Have a solid understanding of modern cryptography.
    *   Use mature cryptography approved by relevant security bodies.
    *   You don’t need to know the details of each algorithm used, but you need to know how to apply them correctly.
*   Keep things simple.
    *   If you are writing software, assess your solutions from the attackers’ point of view. Simple solutions are easier to reason about.
    *   If you are implementing an existing solution, think what will happen if the solution has been compromised. Do not rely on complex conditional controls in your security assessment as a mitigation mechanism. Most attacks succeed because such controls fail.

**About the name**

The zero-knowledge security model shares a part of its name with the zero-knowledge proof in cybersecurity. While these concepts are not directly related, they share the approach of proving that users have access to certain data without exposing any keys or passwords to the other party (server or cloud systems, in case of the ZKSM)

An application with end-to-end encryption alone does not qualify to be called a ‘zero-knowledge’ system as it lacks ways of proving users’ ownership of data/keys. For example, a few videoconferencing solutions advertise their end-to-end encryption capabilities despite ignoring the identity of parties joining the calls.

The zero-knowledge property must apply to the entire world – except the owner of data and her trusted parties. A system that advertises that it does not have access to data fulfils only a small fraction of ZKSM requirements. ‘Zero knowledge’ means that data is unreadable by anyone at storing, transmitting, sharing, and processing stages – unless authorised by the data owner.

to be continued…