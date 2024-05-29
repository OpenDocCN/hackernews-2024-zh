<!--yml
category: 未分类
date: 2024-05-27 15:21:47
-->

# How a mistakenly published password exposed Mercedes-Benz source code | TechCrunch

> 来源：[https://techcrunch.com/2024/01/26/mercedez-benz-token-exposed-source-code-github/](https://techcrunch.com/2024/01/26/mercedez-benz-token-exposed-source-code-github/)

Mercedes-Benz accidentally exposed a trove of internal data after leaving a private key online that gave “unrestricted access” to the company’s source code, according to the security research firm that discovered it.

Shubham Mittal, co-founder and chief technology officer of RedHunt Labs, alerted TechCrunch to the exposure and asked for help in disclosing to the car maker. The London-based cybersecurity company said it discovered a Mercedes employee’s authentication token in a public GitHub repository during a routine internet scan in January.

According to Mittal, this token — an alternative to using a password for authenticating to GitHub — could grant anyone full access to Mercedes’s GitHub Enterprise Server, thus allowing the download of the company’s private source code repositories.

“The GitHub token gave ‘unrestricted’ and ‘unmonitored’ access to the entire source code hosted at the internal GitHub Enterprise Server,” Mittal explained in a report shared by TechCrunch. “The repositories include a large amount of intellectual property… connection strings, cloud access keys, blueprints, design documents, [single sign-on] passwords, API Keys, and other critical internal information.”

Mittal provided TechCrunch with evidence that the exposed repositories contained Microsoft Azure and Amazon Web Services (AWS) keys, a Postgres database, and Mercedes source code. It’s not known if any customer data was contained within the repositories.

TechCrunch disclosed the security issue to Mercedes on Monday. On Wednesday, Mercedes spokesperson Katja Liesenfeld confirmed that the company “revoked the respective API token and removed the public repository immediately.”

“We can confirm that internal source code was published on a public GitHub repository by human error,” Liesenfeld said in a statement to TechCrunch. “The security of our organization, products, and services is one of our top priorities.”

“We will continue to analyze this case according to our normal processes. Depending on this, we implement remedial measures,” Liesenfeld added.

It’s not known if anyone else besides Mittal discovered the exposed key, which was published in late-September 2023.

Mercedes declined to say whether it is aware of any third-party access to the exposed data or whether the company has the technical ability, such as access logs, to determine if there was any improper access to its data repositories. The spokesperson cited unspecified security reasons.

Last week,[TechCrunch exclusively reported that Hyundai’s India subsidiary fixed a bug](https://techcrunch.com/2024/01/11/hyundai-motor-india-data-exposed/) that exposed its customers’ personal information, including the names, mailing addresses, email addresses and phone numbers of Hyundai Motor India customers, who had their vehicles serviced at Hyundai-owned stations across India.

> [Hyundai Motor India fixes bug that exposed customers’ personal data](https://techcrunch.com/2024/01/11/hyundai-motor-india-data-exposed/)