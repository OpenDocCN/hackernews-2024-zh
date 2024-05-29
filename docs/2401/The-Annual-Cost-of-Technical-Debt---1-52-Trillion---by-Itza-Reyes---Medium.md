<!--yml
category: 未分类
date: 2024-05-27 15:08:31
-->

# The Annual Cost of Technical Debt: $1.52 Trillion | by Itza Reyes | Medium

> 来源：[https://itzareyesmx.medium.com/the-annual-cost-of-technical-debt-1-52-trillion-65afaa1e0005](https://itzareyesmx.medium.com/the-annual-cost-of-technical-debt-1-52-trillion-65afaa1e0005)

# The Annual Cost of Technical Debt: $1.52 Trillion

*Para leer en Español clic* [*aquí*](/1ead73736a46)*.*

In case you need any more reasons to prioritize software quality, let me share with you that the cost of poor software quality in the US has risen to at least $2.41 trillion annually, while the accumulated technical debt of software has grown to approximately $1.52 trillion.

This is demonstrated by a study conducted in 2022 and set to be updated in 2024 by the [Consortium for Information & Software Quality (CISQ)](https://www.it-cisq.org/).

While the report focuses on the cost of poor software quality (CPSQ), providing analysis, data, and solutions for areas such as cybercrime and the software supply chain with open-source software (OSS), this article will specifically delve into the topic of technical debt.

CPSQ 2022

# 📈 Why does technical debt arise?

A significant portion of the current technical debt that exists today was created through “quick and dirty” development techniques (e.g., agile without software engineering discipline).

Technical debt accumulates when decision-makers opt for a short-term solution to a software development problem rather than a more comprehensive long-term solution. This initially hides substantial costs that organizations must later pay.

Increase in technical debt

There are various types of technical debt, including requirements, architecture, code, testing, and operational processes. Technical debt can be injected at any stage of software development, spreading to other phases and parts of the system and causing various issues.

Types of Technical Debt

# 💰 The Costs

There are two essential components to the costs of technical debt.

*   The principal refers to the cost of refactoring/modifying software artifacts to achieve the desired level of maintainability and evolvability.
*   The interest is the additional effort developers will dedicate to making those changes due to the existence of the technical debt, which accumulates over time as the software becomes more fragile. Every minute spent on less-than-ideal code adds interest to the debt.

# 📏 Measurement in Software

One of the main challenges when dealing with technical debt has been the lack of a way to measure it. To help overcome that problem, CISQ/OMG led the development of an Automated Technical Debt ([ATD](https://www.omg.org/spec/ATDM/)) measurement standard, which is currently being updated with a new version expected in 2023.

The ATD standard estimates the effort required to address all instances of software weaknesses included in the ISO/IEC 5055:2021 standard for automated source code quality measures that remain in the code of a software application at the time of release.

This estimation can be used to predict future corrective maintenance costs and is calculated using static analysis tools.

The measure expresses the cost of software quality in terms that a company can understand by estimating the future costs of corrective maintenance to remedy structural defects in the code.

# 👀 Recommendations for Addressing It

Throughout the report, several recommendations are made, and at the end, they provide a more specific list of recommendations to avoid poor quality in software development, including technical debt.

*   Use software quality standards, related metrics, and emerging tools.
*   Analyze and evaluate the quality of all OSS/third-party components that will be included in any system. Monitor them closely in operation. Apply patches in a timely manner.
*   Avoid DevOps and CI/CD models that do not include best practices and tools for continuous quality engineering.
*   Integrate continuous technical debt remediation into the software development life cycle.
*   Invest in the professionalism, knowledge, and tools of your software engineers.
*   Consider the possibility of developers certifying their knowledge of critical code and architectural weaknesses in ISO/IEC 5055 (still in progress).

# 🌟 Conclusions

*   The growing impact of technical debt has become the biggest obstacle to making changes in existing code bases.
*   There are many ways to achieve better software quality, but they all start with a well-conceived measurement program fully supported by top management.
*   The potential of managing technical debt is evident in [Stepsize’s](https://www.stepsize.com/) research, which revealed that organizations actively managing technical debt will release at least 50% faster.

**And as we have seen in the CPSQ, preventing or early elimination of technical debt is the most cost-effective long-term strategy.**

# 🔗 Sources

[https://www.it-cisq.org/the-cost-of-poor-quality-software-in-the-us-a-2022-report/](https://www.it-cisq.org/the-cost-of-poor-quality-software-in-the-us-a-2022-report/)

[https://www.theee.ai/2021/01/06/6838-poor-software-quality-cost-the-usd-2-08-tn-in-2020/](https://www.theee.ai/2021/01/06/6838-poor-software-quality-cost-the-usd-2-08-tn-in-2020/)