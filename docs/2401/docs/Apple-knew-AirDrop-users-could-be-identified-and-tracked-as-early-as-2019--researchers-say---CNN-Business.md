<!--yml
category: 未分类
date: 2024-05-27 14:43:13
-->

# Apple knew AirDrop users could be identified and tracked as early as 2019, researchers say | CNN Business

> 来源：[https://edition.cnn.com/2024/01/12/tech/china-apple-airdrop-user-encryption-vulnerability-hnk-intl/index.html](https://edition.cnn.com/2024/01/12/tech/china-apple-airdrop-user-encryption-vulnerability-hnk-intl/index.html)

Editor’s Note: [Sign up for CNN’s Meanwhile in China newsletter, which explores what you need to know about the country’s rise and how it impacts the world.](https://www.cnn.com/newsletters/meanwhile-in-china?source=nl-acq_article)

Washington CNN  — 

Security researchers warned Apple as early as 2019 about vulnerabilities in its AirDrop wireless sharing function that Chinese authorities claim they [recently used](https://www.cnn.com/2024/01/10/tech/china-apple-airdrop-encryption-hnk-intl/index.html) to track down users of the feature, the researchers told CNN, in a case that experts say has sweeping implications for global privacy.

The Chinese government’s actions targeting a tool that Apple customers around the world use to share photos and documents — and Apple’s apparent inaction to address the flaws — revive longstanding concerns by US lawmakers and privacy advocates about Apple’s relationship with China and about authoritarian regimes’ ability to twist US tech products to their own ends.

AirDrop lets Apple users who are near each other share files using a proprietary mix of Bluetooth and other wireless connectivity without having to connect to the internet. The sharing feature has been used by [pro-democracy activists in Hong Kong](https://www.cnn.com/2023/06/09/tech/china-airdrop-bluetooth-national-security-intl-hnk/index.html) and the Chinese government has cracked down on the feature in response.

A Chinese tech firm, Beijing-based Wangshendongjian Technology, was able to compromise AirDrop to identify users on the Beijing subway accused of sharing “inappropriate information,” judicial authorities in Beijing said this week.

Although Chinese officials portrayed the exploit as an effective law enforcement technique, internet freedom advocates are urging Apple to address the issue quickly and publicly.

“Apple’s response to this situation is crucial,” said Benjamin Ismail, campaign and advocacy director of Greatfire.org, a group that monitors internet censorship in China. “They should either refute the claim or confirm it and immediately work on securing AirDrop against such vulnerabilities. It’s imperative that Apple is transparent about their response to these developments.”

The Chinese claim has alarmed top US lawmakers. Florida Sen. Marco Rubio, the leading Republican on the Senate Intelligence Committee, called on Apple to act swiftly.

“Anyone using an iPhone should be concerned with the security of Apple’s AirDrop function,” Rubio told CNN. “This breach is just another way for Beijing to target any Apple user it perceives to be an opponent. The time to act is now, and Apple must be held accountable for failing to safeguard its users against such blatant security breaches.”

An Apple spokesperson did not respond to multiple emails and phone calls seeking comment.

A group of Germany-based researchers at the Technical University of Darmstadt, who first discovered the flaws in 2019, told CNN Thursday they had confirmation Apple received their original report at the time but that the company appears not to have acted on the findings. The same group published a [proposed fix](https://www.usenix.org/system/files/sec21-heinrich.pdf) for the issue in 2021, but Apple appears not to have implemented it, the researchers said.

One of the researchers, Milan Stute, shared an email with CNN showing a representative of Apple’s product security team acknowledging the researchers’ report in 2019.

Chinese authorities claim they exploited the vulnerabilities by collecting some of the basic identifying information that must be transferred between two Apple devices when they use AirDrop — data including device names, email addresses and phone numbers.

Ordinarily, this information is scrambled for privacy reasons. But, according to a separate 2021 [analysis](https://news.sophos.com/en-us/2021/04/23/apple-airdrop-has-significant-privacy-leak-say-german-researchers/) of the Darmstadt research by the UK-based cybersecurity firm Sophos, Apple appeared not to have taken the extra precaution of adding bogus data to the mix to further randomize the results — a process known as “salting.”

That apparent failure allowed the Chinese tech firm to more easily reverse-engineer the original information from the encrypted data, in what seems to be “kind of an amateur mistake” by Apple, said Sascha Meinrath, the Palmer chair in telecommunications at Penn State University. “It certainly merits an explanation from Apple since it would point to a serious flaw in their technology.”

While AirDrop’s device-to-device communications channel is typically protected from third-party snooping by its own layer of security, that wouldn’t shield someone who may have been tricked into connecting with a stranger, perhaps by tapping on a deceptively named device in a list of contacts or by thoughtlessly accepting an unsolicited connection request. This step is required for the sender to be identified, according to security experts.

Once the device-identifying information is exchanged and obtained by an unauthorized third party, the lack of salting would make it straightforward to guess at the correct codes that would unscramble the data, the experts said.

The Chinese tech firm, Wangshendongjian Technology, that claimed to have exploited AirDrop appeared to have used some of the same techniques first identified by the Darmstadt researchers in 2019, said Alexander Heinrich, one of the German researchers.

“As far as we know, Apple did not address the issue so far,” Heinrich told CNN.

Kenn White, an independent security researcher specializing in digital forensics, agreed that what Chinese authorities disclosed about their hack is consistent with what the German researchers found.

“On my read, I’d say this is almost certainly using the same techniques that Heinrich et al published,” White said. “Three plus years and this design flaw appears not to have been addressed.”

On the heels of the Chinese claim, Sen. Ron Wyden, an Oregon Democrat and a vocal privacy advocate in Congress, blasted Apple over a “blatant failure” to protect its customers.

“Apple has had four years to fix the security hole in AirDrop that put the privacy and safety of its users at risk,” Wyden said in a statement to CNN. “Apple sat on its hands and did nothing, rather than protect human rights activists who depend on iPhones to share messages the Chinese government doesn’t want people to see.”

The tech firm behind the AirDrop exploit has a history of working closely with Chinese law enforcement and security authorities.

Its parent company is the powerful Chinese cybersecurity firm Qi An Xin, according to corporate database Aiqicha. Qi An Xin was hired to protect the Beijing Winter Olympic Games in 2022 from cyberattacks, [according to](http://www.news.cn/2021-12/24/c_1128196797.htm) the official Xinhua news agency.

“Time and again, the Chinese government turns to the private sector to augment its technical capabilities,” Dakota Cary, a China-focused consultant at US cybersecurity firm SentinelOne, told CNN. “This is an important reminder of the offensive role that ostensibly defensive Chinese cybersecurity companies can play.”

It is rare, however, for a government actor such as China to publicly disclose its capabilities, suggesting that the intentional reveal this week speaks to some other motive.

“It’s very much in their interests not to spill their techniques,” White said.

One reason Chinese officials may have wanted their exploit known, said Ismail, is that it could scare dissidents away from using AirDrop.

And now that the Beijing authorities have announced it exploited the vulnerability, Apple may face retaliation from Chinese authorities if the tech firm tries to fix the issue, multiple experts said.

China is the largest foreign market for Apple’s products, with sales there representing about a fifth of the company’s total revenue in 2022\. Most of its iPhones are produced in Chinese factories, and Apple could face blowback from Beijing if it moves to close off the loophole.

The revelation of the hack could also give China even more leverage to force Apple to cooperate with the country’s security or intelligence demands, said Ismail, because China can argue Apple is already complicit.

“If Apple had fixed it when it was reported in 2019, it would’ve been a challenging technical problem,” said Matthew Green, a cryptography expert and professor at Johns Hopkins University. “Now that Chinese security agencies are exploiting this vulnerability, it’s a tough political problem for Apple.”