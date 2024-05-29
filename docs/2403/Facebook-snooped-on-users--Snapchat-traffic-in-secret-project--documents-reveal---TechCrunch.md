<!--yml
category: 未分类
date: 2024-05-29 12:40:51
-->

# Facebook snooped on users' Snapchat traffic in secret project, documents reveal | TechCrunch

> 来源：[https://techcrunch.com/2024/03/26/facebook-secret-project-snooped-snapchat-user-traffic/](https://techcrunch.com/2024/03/26/facebook-secret-project-snooped-snapchat-user-traffic/)

In 2016, Facebook launched a secret project designed to intercept and decrypt the network traffic between people using Snapchat’s app and its servers. The goal was to understand users’ behavior and help Facebook compete with Snapchat, according to newly unsealed court documents. Facebook called this “Project Ghostbusters,” in a clear reference to Snapchat’s ghost-like logo.

On Tuesday, a federal court in California released new documents discovered as part of the class action lawsuit between consumers and Meta, Facebook’s parent company.

The newly released documents reveal how Meta tried to gain a competitive advantage over its competitors, including Snapchat and later Amazon and YouTube, by analyzing the network traffic of how its users were interacting with Meta’s competitors. Given these apps’ use of encryption, Facebook needed to develop special technology to get around it.

[One of the documents](https://www.documentcloud.org/documents/24520332-merged-fb) details Facebook’s Project Ghostbusters. The project was part of the company’s In-App Action Panel (IAPP) program, which used a technique for “intercepting and decrypting” encrypted app traffic from users of Snapchat, and later from users of YouTube and Amazon, the consumers’ lawyers wrote in the document.

The document includes internal Facebook emails discussing the project.

“Whenever someone asks a question about Snapchat, the answer is usually that because their traffic is encrypted we have no analytics about them,” Meta chief executive Mark Zuckerberg wrote in an email dated June 9, 2016, which was published as part of the lawsuit. “Given how quickly they’re growing, it seems important to figure out a new way to get reliable analytics about them. Perhaps we need to do panels or write custom software. You should figure out how to do this.”

Facebook’s engineers solution was to use [Onavo](https://techcrunch.com/tag/onavo/), a VPN-like service that Facebook acquired in 2013\. In 2019, [Facebook shut down Onavo](https://techcrunch.com/2019/02/21/facebook-removes-onavo/) after a TechCrunch investigation revealed that [Facebook had been secretly paying teenagers to use Onavo](https://techcrunch.com/2019/01/29/facebook-project-atlas/) so the company could access all of their web activity.

After Zuckerberg’s email, the Onavo team took on the project and a month later proposed a solution: so-called kits that can be installed on iOS and Android that intercept traffic for specific subdomains, “allowing us to read what would otherwise be encrypted traffic so we can measure in-app usage,” read an email from July 2016\. “This is a ‘man-in-the-middle’ approach.”

#### Contact Us

Do you know more about Project Ghostbusters? Or other privacy issues at Facebook? From a non-work device, you can contact Lorenzo Franceschi-Bicchierai securely on Signal at +1 917 257 1382, or via Telegram, Keybase and Wire @lorenzofb, or

[email](mailto:lorenzo@techcrunch.com)[.](mailto:lorenzo@techcrunch.com)

You also can contact TechCrunch via

[SecureDrop](https://techcrunch.com/got-a-tip/)

.

A man-in-the-middle attack — nowadays also called adversary-in-the-middle — is an attack where hackers intercept internet traffic flowing from one device to another over a network. When the network traffic is unencrypted, this type of attack allows the hackers to read the data inside, such as usernames, passwords, and other in-app activity.

Given that Snapchat encrypted the traffic between the app and its servers, this network analysis technique was not going to be effective. This is why Facebook engineers proposed using Onavo, which when activated had the advantage of reading all of the device’s network traffic before it got encrypted and sent over the internet.

“We now have the capability to measure detailed in-app activity” from “parsing snapchat [sic] analytics collected from incentivized participants in Onavo’s research program,” read another email.

Later, according to the court documents, Facebook expanded the program to Amazon and YouTube.

Inside Facebook, there wasn’t a consensus on whether Project Ghostbusters was a good idea. Some employees, including Jay Parikh, Facebook’s then-head of infrastructure engineering, and Pedro Canahuati, the then-head of security engineering, expressed their concern.

“I can’t think of a good argument for why this is okay. No security person is ever comfortable with this, no matter what consent we get from the general public. The general public just doesn’t know how this stuff works,” Canahuati wrote in an email, included in the court documents.

In 2020, Sarah Grabert and Maximilian Klein [filed a class action lawsuit against Facebook](https://www.jurist.org/news/2020/12/class-action-lawsuit-against-facebook-alleges-anticompetitive-behavior/), claiming that the company lied about its data collection activities and exploited the data it “deceptively extracted” from users to identify competitors and then unfairly fight against these new companies.

An Amazon spokesperson declined to comment.

Google, Meta, and Snap did not respond to requests for comment.

*This story was updated to correct the link to the discovery documents in the fourth paragraph.*

> [Facebook will shut down its spyware VPN app Onavo](https://techcrunch.com/2019/02/21/facebook-removes-onavo/)