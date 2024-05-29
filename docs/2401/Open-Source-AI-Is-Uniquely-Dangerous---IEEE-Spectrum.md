<!--yml
category: 未分类
date: 2024-05-27 14:45:06
-->

# Open-Source AI Is Uniquely Dangerous - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/open-source-ai-2666932122](https://spectrum.ieee.org/open-source-ai-2666932122)

*This is a guest post. *For the other side of the argument about open-source AI, see the recent guest post “[Open Source AI Is Good for Us](https://spectrum.ieee.org/open-source-ai-good).”**

*When people think of AI applications these days, they likely think of “closed-source” AI applications like OpenAI’s [ChatGPT](https://chat.openai.com/)—where the system’s software is securely held by its maker and a limited set of vetted partners. Everyday users interact with these systems through a Web interface like a chatbot, and business users can access an application programming interface (API) which allows them to embed the AI system in their own applications or workflows. Crucially, these uses allow the company that owns the model to provide access to it as a service, while keeping the underlying software secure. Less well understood by the public is the rapid and uncontrolled release of powerful unsecured (sometimes called open-source) AI systems.*

*A good first step in understanding the threats posed by unsecured AI is to ask secured AI systems like [ChatGPT](https://spectrum.ieee.org/tag/chatgpt), Bard, or Claude to misbehave.*

*[OpenAI](https://openai.com/)’s brand name adds to the confusion. While the company was originally founded to produce open-source AI systems, its leaders determined in 2019 that it was [too dangerous](https://www.wired.com/story/ai-text-generator-too-dangerous-to-make-public/) to continue releasing its GPT systems’ source code and model weights (the numerical representations of relationships between the nodes in its artificial neural network) to the public. [OpenAI](https://spectrum.ieee.org/tag/openai) worried because these text-generating AI systems can be used to generate massive amounts of well-written but misleading or [toxic](https://spectrum.ieee.org/open-ais-powerful-text-generating-tool-is-ready-for-business) content.*

*Companies including [Meta](https://www.meta.com/) (my former employer) have moved in the opposite direction, choosing to release powerful unsecured AI systems in the name of [democratizing](https://about.fb.com/news/2023/07/llama-2/) access to AI. Other examples of companies releasing unsecured AI systems include [Stability AI](https://stability.ai/), [Hugging Face](https://huggingface.co/), [Mistral](https://mistral.ai/), [EleutherAI](https://www.eleuther.ai/), and the [Technology Innovation Institute](https://www.tii.ae/). These companies and like-minded advocacy groups have made limited progress in [obtaining exemptions](https://ec.europa.eu/commission/presscorner/detail/en/QANDA_21_1683) for some unsecured models in the European Union’s [AI Act](https://artificialintelligenceact.eu/), which is designed to reduce the risks of powerful AI systems. They may push for similar exemptions in the United States via the public comment period recently [set forth in](https://docs.google.com/document/d/1u-MUpA7TLO4rnrhE2rceMSjqZK2vN9ltJJ38Uh5uka4/edit#heading=h.t02rblknwe5) the White House’s [AI Executive Order](https://spectrum.ieee.org/biden-ai-executive-order).*

*I think the [open-source](https://spectrum.ieee.org/tag/open-source) movement has an important role in AI. With a technology that brings so many new capabilities, it’s important that no single entity acts as a gatekeeper to the technology’s use. However, as things stand today, unsecured AI poses an enormous risk that we are not yet able to contain.*

## *Understanding the Threat of Unsecured AI*

*A good first step in understanding the threats posed by unsecured AI is to ask secured AI systems like ChatGPT, [Bard](https://bard.google.com/), or [Claude](https://claude.ai/) to misbehave. You could ask them to design a more deadly coronavirus, provide instructions for making a bomb, make naked pictures of your favorite actor, or write a series of inflammatory text messages designed to make voters in swing states more angry about immigration. You will likely receive polite refusals to all such requests because [they violate](https://openai.com/policies/usage-policies) the [usage policies](https://policies.google.com/terms/generative-ai/use-policy) of [these AI systems](https://console.anthropic.com/legal/aup). Yes, it is possible to “jailbreak” these [AI systems](https://arxiv.org/abs/2305.13860) and get them to misbehave, but as these vulnerabilities are discovered, they can be fixed.*

*Enter the unsecured models. Most famous is Meta’s [Llama 2](https://ai.meta.com/llama/). It was released by [Meta](https://spectrum.ieee.org/tag/meta) with a 27-page “[Responsible Use Guide](https://ai.meta.com/static-resource/responsible-use-guide/),” which was promptly ignored by the creators of “[Llama 2 Uncensored](https://huggingface.co/jarradh/llama2_70b_chat_uncensored),” a derivative model with safety features stripped away, and hosted for free download on the Hugging Face AI repository. Once someone releases an “uncensored” version of an unsecured AI system, the original maker of the system is largely powerless to do anything about it.*

*As things stand today, unsecured AI poses an enormous risk that we are not yet able to contain.*

*The threat posed by unsecured AI systems lies in the ease of misuse. They are particularly dangerous in the hands of sophisticated threat actors, who could easily download the original versions of these AI systems and disable their safety features, then make their own custom versions and abuse them for a wide variety of tasks. Some of the abuses of unsecured AI systems also involve taking advantage of vulnerable distribution channels, such as social media and messaging platforms. These platforms cannot yet accurately detect AI-generated content at scale and can be used to distribute massive amounts of personalized misinformation and, of course, [scams](https://theconversation.com/ai-scam-calls-imitating-familiar-voices-are-a-growing-problem-heres-how-they-work-208221). This could have catastrophic effects on the information ecosystem, and on [elections](https://spectrum.ieee.org/deepfakes-election) in particular. Highly damaging [nonconsensual deepfake pornography](https://www.wired.com/story/deepfake-porn-is-out-of-control/) is yet another domain where unsecured AI can have deep negative consequences.*

*Unsecured AI also has the potential to facilitate production of dangerous materials, such as [biological and chemical weapons](https://www.axios.com/2023/06/16/pandemic-bioterror-ai-chatgpt-bioattacks). The White House Executive Order references chemical, biological, radiological, and nuclear ([CBRN](https://docs.google.com/document/d/1u-MUpA7TLO4rnrhE2rceMSjqZK2vN9ltJJ38Uh5uka4/edit#heading=h.6fkzizejib9o)) risks, and [multiple bills](https://www.markey.senate.gov/news/press-releases/sens-markey-budd-announce-legislation-to-assess-health-security-risks-of-ai) are now under consideration by the U.S. Congress to address these threats.*

## *Recommendations for AI Regulations*

*We don’t need to specifically regulate unsecured AI—nearly all of the regulations that have been publicly discussed apply to secured AI systems as well. The only difference is that it’s much easier for developers of secured AI systems to comply with these regulations because of the inherent properties of secured and unsecured AI. The entities that operate secured AI systems can actively monitor for abuses or failures of their systems (including bias and the production of dangerous or offensive content) and release regular updates that make their systems more fair and safe.*

*“I think how we regulate open-source AI is THE most important unresolved issue in the immediate term.”
**—Gary Marcus, New York University***

*Almost all the regulations recommended below generalize to all AI systems. Implementing these regulations would make companies think twice before releasing unsecured AI systems that are ripe for abuse.*

***Regulatory Action for AI Systems***

1.  ***Pause all new releases** of unsecured AI systems until developers have met the requirements below, and in ways that ensure that safety features cannot be easily removed by bad actors.*
2.  ***Establish registration and licensing** (both retroactive and ongoing) of all AI systems above a certain capability threshold.*
3.  ***Create liability** for “reasonably foreseeable misuse” and negligence: Developers of AI systems should be legally liable for harms caused to both individuals and to society.*
4.  ***Establish risk assessment, mitigation, and independent audit** procedures for AI systems crossing the threshold mentioned above.*
5.  ***Require watermarking and provenance** best practices so that AI-generated content is clearly labeled and authentic content has metadata that lets users understand its provenance.*
6.  ***Require transparency of training data** and prohibit training systems on personally identifiable information, content designed to generate hateful content, and content related to biological and chemical weapons.*
7.  ***Require and fund independent researcher access**, giving vetted researchers and civil society organizations predeployment access to generative AI systems for research and testing.*
8.  ***Require “know your customer” procedures**, similar to those [used by financial institutions](https://www.swift.com/your-needs/financial-crime-cyber-security/know-your-customer-kyc/meaning-kyc), for sales of powerful hardware and cloud services designed for AI use; restrict sales in the same way that weapons sales would be restricted.*
9.  ***Mandatory incident disclosure**: When developers learn of vulnerabilities or failures in their AI systems, they must be [legally required to report](https://csrc.nist.gov/pubs/sp/800/61/r2/final) this to a designated government authority.*

***Regulatory Action for Distribution Channels and Attack Surfaces***

1.  ***Require content credential implementation** for social media, giving companies a deadline to implement the [Content Credentials labeling standard](https://contentcredentials.org/) from C2PA.*
2.  ***Automate digital signatures** so people can rapidly verify their human-generated content.*
3.  ***Limit the reach of AI-generated content**: Accounts that haven’t been verified as distributors of human-generated content could have certain features disabled, including viral distribution of their content.*
4.  ***Reduce chemical, biological, radiological, and nuclear risks** by educating all suppliers of custom nucleic acids or other potentially dangerous substances about best practices.*

***Government Action***

1.  ***Establish a nimble regulatory body** that can act and enforce quickly and update certain enforcement criteria. This entity would have the power to approve or reject risk assessments, mitigations, and audit results and have the authority to block model deployment.*
2.  ***Support fact-checking organizations** and civil-society groups (including the “[trusted flaggers](https://ec.europa.eu/commission/presscorner/detail/en/QANDA_20_2348)” defined by the [EU Digital Services Act](https://digital-strategy.ec.europa.eu/en/policies/digital-services-act-package)) and require generative AI companies to work directly with these groups.*
3.  ***Cooperate internationally** with the goal of eventually creating an international [treaty](https://www.cigionline.org/articles/voluntary-curbs-arent-enough-ai-risk-requires-a-binding-international-treaty/) or new international agency to prevent companies from circumventing these regulations. The recent [Bletchley Declaration](https://www.gov.uk/government/publications/ai-safety-summit-2023-the-bletchley-declaration/the-bletchley-declaration-by-countries-attending-the-ai-safety-summit-1-2-november-2023) was signed by 28 countries, including the home countries of all of the world’s leading AI companies (United States, China, United Kingdom, United Arab Emirates, France, and Germany); this declaration stated shared values and carved out a path for additional meetings.*
4.  ***Democratize AI access** with public infrastructure: A common concern about regulating AI is that it will limit the number of companies that can produce complicated AI systems to a small handful and tend toward monopolistic business practices. There are many opportunities to democratize access to AI, however, without relying on unsecured AI systems. One is through the creation of [public AI infrastructure](https://cdn.vanderbilt.edu/vu-URL/wp-content/uploads/sites/412/2023/10/09151836/VPA-AI-Capacity.10.9.23.pdf) with powerful secured AI models.*

*“I think how we regulate open-source AI is THE most important unresolved issue in the immediate term,” [Gary Marcus](http://garymarcus.com/index.html), the cognitive scientist, entrepreneur, and professor emeritus at New York University told me in a recent email exchange.* 

*I agree, and these recommendations are only a start. They would initially be costly to implement and would require that regulators make certain powerful lobbyists and developers unhappy.*

*Unfortunately, given the misaligned incentives in the current AI and information ecosystems, it’s unlikely that industry will take these actions unless forced to do so. If actions like these are not taken, companies producing unsecured AI may bring in billions of dollars in profits while pushing the risks posed by their products onto all of us.*

*From Your Site Articles*

*Related Articles Around the Web*