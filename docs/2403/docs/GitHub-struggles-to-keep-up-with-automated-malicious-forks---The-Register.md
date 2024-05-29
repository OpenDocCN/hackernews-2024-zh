<!--yml
category: 未分类
date: 2024-05-27 14:32:12
-->

# GitHub struggles to keep up with automated malicious forks • The Register

> 来源：[https://www.theregister.com/2024/03/01/github_automated_fork_campaign/](https://www.theregister.com/2024/03/01/github_automated_fork_campaign/)

A malware distribution campaign that began last May with a handful of malicious software packages uploaded to the Python Package Index (PyPI) has spread to GitHub and expanded to reach at least 100,000 compromised repositories.

According to security firm Apiiro, the campaign to poison code involves cloning legitimate repos, infecting them with malware loaders, uploading the altered files to GitHub under the same name, then forking the poisoned repo thousands of times and promoting the compromised code in forums and on social media channels.

Developers looking for useful code may therefore find a repo that’s describes as useful and at first glance appears appropriate, only to have their personal data pilfered by a hidden payload that runs malicious Python code and a binary executable.

"The malicious code (largely a modified version of [BlackCap-Grabber](https://github.com/Inplex-sys/BlackCap-Grabber-NoDualHook)) would then collect login credentials from different apps, browser passwords and cookies, and other confidential data," said Matan Giladi, security researcher, and Gil David, head of AI, in a [report](https://apiiro.com/blog/malicious-code-campaign-github-repo-confusion-attack/). "It then sends it back to the malicious actors' C&C (command-and-control) server and performs a long series of additional malicious activities."

A Trend Micro [analysis](https://www.trendmicro.com/en_be/research/23/j/infection-techniques-across-supply-chains-and-codebases.html) of the malicious code describes how it employs clever techniques to conceal its true nature. For example, the code hides its use of the exec function – for dynamically executing code – through a technique dubbed “exec smuggling”.

Such attacks add hundreds of whitespace characters (521 of them) to push the exec function offscreen as a defense against manual scrutiny.

GitHub says it's aware that not all's well.

"GitHub hosts over 100 million developers building across over 420 million repositories, and is committed to providing a safe and secure platform for developers," a spokesperson told *The Register*.

"We have teams dedicated to detecting, analyzing, and removing content and accounts that violate our [Acceptable Use Policies](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies#5-site-access-and-safety). We employ manual reviews and at-scale detections that use machine learning and constantly evolve and adapt to adversarial tactics. We also encourage customers and community members to [report abuse and spam](https://docs.github.com/en/communities/maintaining-your-safety-on-github/reporting-abuse-or-spam)."

Awareness and automated scanning is all very well – but Apiiro’s Giladi and David observed that GitHub missed many automated repo forks, as well as the manually uploaded ones.

"Because the whole attack chain seems to be mostly automated on a large scale, the one percent that survive still amount to thousands of malicious repos," the authors wrote, adding that if you count removed repos in the total, the campaign probably involved millions of malicious clones and forks.

They also point out that the scale of the attack is large enough to benefit from network effects, specifically developers who fork malicious repos without intending to use the software and don't realize they're validating and propagating malware.

GitHub, the researchers say, presents an effective way to compromise the software supply chain due to its support for the automatic generation of accounts and repos, its friendly APIs and soft rate limits, and its size.

The Biden administration had pushed for stronger software supply chain security through the National Institute of Standards and Technology's [Cybersecurity Framework 2.0](https://www.theregister.com/2024/02/27/nist_cybersecurity_framework_2/) and efforts to get organizations to publish their [software bill of materials](https://www.theregister.com/2023/03/05/sboms_supply_chain_security/). But clearly there's work left to do. ®