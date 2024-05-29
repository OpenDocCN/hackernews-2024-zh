<!--yml
category: 未分类
date: 2024-05-27 14:47:40
-->

# Big Tech’s role in enabling link fraud — Eli Grey

> 来源：[https://eligrey.com/blog/link-fraud/](https://eligrey.com/blog/link-fraud/)

Link fraud is increasingly undermining trust in major online platforms, including Google, Bing, and X (Twitter).

These platforms allow advertisers to spoof links with *unverified* ‘vanity URLs’, laundering trust in their systems, while simultaneously deflecting blame onto advertisers when these mechanisms are exploited for fraudulent purposes. 

I believe that this status quo must be abolished. Commercial entities that maintain advertising systems that systemically enable link fraud must contend with their net-negative impact on society.

## What are vanity URLs and what is link fraud?

URL spoofing is the act of presenting an internet address that appears to lead to one destination but actually leads to another, unexpected, location. URL spoofing is commonly referred to as “vanity URLs” by the adtech industry when provided as a first-class feature on adtech platforms.

Link fraud is the use of URL spoofing to achieve financial gain or other illicit objectives. It is a staple practice in spam emails and scam websites, where links may appear legitimate but lead to harmful content.

## Examples of link fraud

Google Search:

Microsoft Bing: See [this Forbes article](https://www.forbes.com/sites/jasonevangelho/2018/10/27/stop-using-microsoft-edge-to-download-chrome-unless-you-want-malware/?sh=6fd791f513ea) and [my analysis of the incident](https://twitter.com/sephr/status/1055751684146655232). I reproduced the issue at the time and triaged the root cause to be uncontrolled use of vanity URLs.

X: I’ve personally witnessed it twice but didn’t screenshot it.

## Culprits

Google, Microsoft, and X all have similarly ineffective enforcement methods and policies for vanity URL control. I suspect that Reddit also has similar limitations but haven’t confirmed this.

## Implicit regulatory capture

Adtech companies play the victim by claiming that fraudsters and scammers are ‘abusing’ their unverified vanity URL systems. These companies should not be able to get away with creating systems that enable link fraud and then pretend to tie their hands behind their back when asked to combat the issue. They have created systems for trust-laundered URL spoofing, and then disclaimed ethical or legal responsibility for the fundamental technical failures of these systems.

It is not possible to automatically prevent link fraud in systems that allow for unverified URL spoofing to occur. If adtech providers do not perform domain ownership verification on vanity URLs, advertisers are technically free to commit fraud as they please.

## How did we get here?

The adtech industry may excuse these practices as an unavoidable consequence of the complexity of online advertising. However, this overlooks the responsibility that these companies bear for prioritizing profit over user safety and the integrity of their platforms.

Corporate greed has gotten so out-of-control that companies such as Google, Microsoft, and Brave now all deeply integrate advertising technologies at the browser-level, with some effects ranging from battery drain to [personal interest tracking](https://developers.google.com/privacy-sandbox/relevance/protected-audience), and even [taking a cut of the value of your attention](https://brave.com/brave-rewards/#terms:~:text=70%25%20of%20the%20revenue%20Brave%20earns%20through%20these%20unobtrusive%2C%20privacy%2Dpreserving%20ads%20is%20shared%20directly%20back%20with%20users%20as%20Brave%20Rewards.).

## National security risks

The risk of malvertising and fraud through adtech platforms has become so concerning and prevalent that [the FBI now recommends all citizens install ad blockers](https://www.ic3.gov/Media/Y2022/PSA221221#:~:text=Use%20an%20ad%20blocking%20extension%20when%20performing%20internet%20searches.). Interestingly, some of the FBI’s advice for checking ad authenticity is inadequate in practice. The FBI suggests “Before clicking on an advertisement, check the URL to make sure the site is authentic. A malicious domain name may be similar to the intended URL but with typos or a misplaced letter.” — this is useless advice in the face of unverified vanity URLs. Instead of asking private citizens to block an entire ‘legal’ industry, the FBI should be investigating adtech platforms for systemically enabling link fraud.

Intelligence agencies such as [the NSA and CIA also use adblockers](https://www.vice.com/en/article/93ypke/the-nsa-and-cia-use-ad-blockers-because-online-advertising-is-so-dangerous) in order to keep their personnel safe from malware threats. I anticipate that the US federal government may start requiring adblockers on all federal employee devices at some point in the future.

## What can be done? Verification & enforcement

Companies are generally mandated by law to provide true statements to consumers where technically possible. Unverified vanity URLs as a first-class feature flies in the face of these requirements.

Adtech providers should validate ownership of the domain names used within vanity URLs, or alternatively vanity URLs should be banned entirely. Validating domain ownership can easily be done through automated or manual processes where domain name owners place unique keys in their domain name’s DNS records.

A common, yet fundamentally flawed verification mechanism that adtech platforms such as Google Ads employ is the use of sampled URL resolution, which involves visiting a website at given points in time from one or more given computers. This technique can easily be bypassed with [dynamic redirection software that can hide fraud and malware from URL scanning servers](https://eligrey.com/blog/zerodrop/).

Petition your elected government officials to let them know that big tech is willingly ignoring their role in the rise of effective link fraud, spurred by their support of unverified vanity URLs. The United States Federal Trade Commission should request an investigation and seek to prosecute companies that knowingly enable link fraud through unverified vanity URL systems that are fundamentally impossible to audit.

On a personal level, you can install an adblocker such as uBlock Origin to block advertising, which has a nice added side effect of increasing web browsing privacy and performance.