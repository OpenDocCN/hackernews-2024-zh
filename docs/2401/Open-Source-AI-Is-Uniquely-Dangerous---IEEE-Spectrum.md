<!--yml

分类: 未分类

日期: 2024-05-27 14:45:06

-->

# 开源人工智能具有独特的危险性 - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/open-source-ai-2666932122](https://spectrum.ieee.org/open-source-ai-2666932122)

*这是一篇客座文章。*关于开源人工智能的另一方面的论述，请参阅最近的客座文章“[开源人工智能对我们有益](https://spectrum.ieee.org/open-source-ai-good)。”**

*当今人们谈到人工智能应用时，他们可能会想到像OpenAI的[ChatGPT](https://chat.openai.com/)这样的“封闭源代码”人工智能应用——在这些系统中，软件由其制造商和一组有限的经过验证的合作伙伴安全保管。普通用户通过网页界面与这些系统交互，例如聊天机器人，而企业用户可以访问应用程序编程接口（API），允许他们将人工智能系统嵌入到自己的应用程序或工作流程中。关键是，这些用途允许拥有模型的公司将其作为服务提供给用户，同时保持底层软件的安全性。公众不太了解的是，强大的未安全保护（有时称为开源）人工智能系统的迅速和不受控制的发布。*

*了解未安全保护的人工智能带来的威胁的一个好的第一步是要求像[ChatGPT](https://spectrum.ieee.org/tag/chatgpt)，Bard或Claude这样的安全人工智能系统表现不端。*

*[OpenAI](https://openai.com/)的品牌名称加剧了混淆。虽然该公司最初成立的目的是生产开源人工智能系统，但其领导人在2019年确定，继续向公众发布其GPT系统的源代码和模型权重（人工神经网络中节点之间关系的数字表示）太危险了。[OpenAI](https://spectrum.ieee.org/tag/openai)之所以担心，是因为这些文本生成的人工智能系统可以用来生成大量写得很好但具有误导性或[有毒](https://spectrum.ieee.org/open-ais-powerful-text-generating-tool-is-ready-for-business)内容。*

*包括[Meta](https://www.meta.com/)（我的前雇主）在内的公司已经朝着相反的方向发展，选择以[民主化](https://about.fb.com/news/2023/07/llama-2/)AI的获取为名发布功能强大的不安全人工智能系统。其他发布不安全人工智能系统的公司的例子包括[Stability AI](https://stability.ai/)，[Hugging Face](https://huggingface.co/)，[Mistral](https://mistral.ai/)，[EleutherAI](https://www.eleuther.ai/)和[Technology Innovation Institute](https://www.tii.ae/)。这些公司和志同道合的倡导团体在欧盟的[AI法](https://artificialintelligenceact.eu/)中取得了一些不安全模型的[豁免](https://ec.europa.eu/commission/presscorner/detail/en/QANDA_21_1683)，该法旨在减少功能强大的人工智能系统的风险。他们可能会通过最近在白宫发布的[AI行政命令](https://spectrum.ieee.org/biden-ai-executive-order)中设定的公开评论期来推动在美国取得类似的豁免。*

*我认为[开源](https://spectrum.ieee.org/tag/open-source)运动在人工智能领域起着重要作用。有了一项带来如此多新功能的技术，确保没有单一实体作为技术使用的门卫是很重要的。然而，就目前情况而言，不安全的人工智能带来了我们尚无法控制的巨大风险。*

## *了解不安全人工智能的威胁*

*了解不安全人工智能带来的威胁的一个好的第一步是要求像ChatGPT、[Bard](https://bard.google.com/)或者[Claude](https://claude.ai/)这样的安全人工智能系统去做坏事。你可以要求它们设计一个更致命的冠状病毒，提供制造炸弹的指南，制作你最喜欢的演员的裸照，或者写一系列旨在让摇摆州选民更愤怒于移民问题的煽动性短信。你可能会收到礼貌的拒绝，因为这些请求违反了[这些人工智能系统](https://openai.com/policies/usage-policies)的[使用政策](https://policies.google.com/terms/generative-ai/use-policy)。是的，可能会“越狱”这些[人工智能系统](https://arxiv.org/abs/2305.13860)并让它们做坏事，但是一旦发现这些漏洞，就可以修复它们。*

*进入未保护的模型。最有名的是Meta的[Llama 2](https://ai.meta.com/llama/)。它是由[Meta](https://spectrum.ieee.org/tag/meta)发布的，附带有一个27页的“[负责任使用指南](https://ai.meta.com/static-resource/responsible-use-guide/)”，但很快被[Llama 2未经审查版](https://huggingface.co/jarradh/llama2_70b_chat_uncensored)的创建者忽略，这是一个剥去安全功能的衍生模型，并且在Hugging Face人工智能存储库上免费提供下载。一旦有人发布了未经审查的未保护人工智能系统的“未经审查”版本，系统的原始制造商基本上无能为力。*

*就目前情况而言，未保护的人工智能构成了我们尚无法控制的巨大风险。*

*未保护的人工智能系统带来的威胁在于容易被误用。它们在精密威胁行为者手中尤其危险，后者可以轻松下载这些人工智能系统的原始版本并禁用其安全功能，然后制作自己的定制版本并滥用它们进行各种任务。一些未保护的人工智能系统的滥用还涉及利用脆弱的分发渠道，例如社交媒体和消息平台。这些平台目前还无法准确地大规模检测人工智能生成的内容，并且可以用来传播大量个性化的虚假信息和当然也有[诈骗](https://theconversation.com/ai-scam-calls-imitating-familiar-voices-are-a-growing-problem-heres-how-they-work-208221)。这可能对信息生态系统，尤其是对[选举](https://spectrum.ieee.org/deepfakes-election)产生灾难性影响。高度破坏性的[未经同意的深度伪造色情内容](https://www.wired.com/story/deepfake-porn-is-out-of-control/)是另一个未保护人工智能可能产生深远负面影响的领域。*

*未经保护的人工智能还有可能促进危险材料的生产，例如[生物和化学武器](https://www.axios.com/2023/06/16/pandemic-bioterror-ai-chatgpt-bioattacks)。白宫的行政命令提到了化学、生物、放射性和核（[CBRN](https://docs.google.com/document/d/1u-MUpA7TLO4rnrhE2rceMSjqZK2vN9ltJJ38Uh5uka4/edit#heading=h.6fkzizejib9o)）风险，并且[多项法案](https://www.markey.senate.gov/news/press-releases/sens-markey-budd-announce-legislation-to-assess-health-security-risks-of-ai)现在正在美国国会考虑，以应对这些威胁。*

## *人工智能监管建议*

我们不需要专门监管不安全的人工智能——几乎所有公开讨论过的法规都适用于安全的人工智能系统。唯一的区别在于，安全和不安全的人工智能系统的开发者更容易遵守这些法规，因为安全和不安全的人工智能系统的固有属性不同。操作安全人工智能系统的实体可以积极监控其系统的滥用或故障（包括偏见和生成危险或冒犯性内容），并发布定期更新，使其系统更加公平和安全。

“我认为我们如何监管开源人工智能是当务之急中最重要但未解决的问题。”

**—加里·马库斯，纽约大学**

几乎所有下面推荐的法规都适用于所有人工智能系统。实施这些法规将使公司在发布易于滥用的未安全化人工智能系统之前三思而行。

**人工智能系统的监管行动**

1.  **暂停所有未安全化人工智能系统的新发布**，直到开发者满足以下要求，并确保安全功能不易被不良行为者轻易移除。

1.  **建立所有超过一定能力门槛的人工智能系统的注册和许可证**（包括追溯和持续）。

1.  **对“合理预见的滥用”和疏忽创建责任**：人工智能系统的开发者应对对个人和社会造成的伤害承担法律责任。

1.  **建立风险评估、缓解和独立审计**程序，用于超过上述门槛的人工智能系统。

1.  **要求水印和出处**最佳实践，以便人工智能生成的内容明确标记，并且真实内容具有元数据，让用户了解其来源。

1.  **要求透明的训练数据**，并禁止在个人可识别信息、设计用于生成仇恨内容的内容以及与生物和化学武器相关的内容上进行训练。

1.  **要求并资助独立研究者访问**，为经过审核的研究人员和公民社会组织提供对生成式人工智能系统的预部署访问，以进行研究和测试。

1.  **要求“了解您的客户”程序**，类似于[金融机构使用的](https://www.swift.com/your-needs/financial-crime-cyber-security/know-your-customer-kyc/meaning-kyc)用于销售专为人工智能使用而设计的强大硬件和云服务的程序；以限制销售方式，就像武器销售将被限制一样。

1.  **强制性事件披露**：当开发人员了解到其人工智能系统存在漏洞或故障时，他们必须向指定的政府机构[法定要求报告](https://csrc.nist.gov/pubs/sp/800/61/r2/final)。

**分销渠道和攻击面的监管行动**

1.  ***要求社交媒体实施内容凭证**，并给予公司截止日期以实施由 C2PA 提供的[内容凭证标准](https://contentcredentials.org/)。*

1.  ***自动化数字签名**，使人们能够迅速验证其人类生成的内容。*

1.  ***限制 AI 生成内容的影响范围**：未经验证为人类生成内容分发者的帐户可能会被禁用某些功能，包括其内容的病毒式传播。*

1.  ***通过向所有定制核酸或其他潜在危险物质的供应商提供最佳实践教育，降低化学、生物、放射性和核风险**。*

***政府行动***

1.  ***建立灵活的监管机构**，该机构能够迅速行动并执行，并更新某些执行标准。该实体将有权批准或拒绝风险评估、减轻措施和审计结果，并有权阻止模型的部署。*

1.  ***支持事实核查组织**和民间社会团体（包括由[欧盟数字服务法](https://digital-strategy.ec.europa.eu/en/policies/digital-services-act-package)定义的“受信任标志者”以及[受信任标志者](https://ec.europa.eu/commission/presscorner/detail/en/QANDA_20_2348)）并要求生成性 AI 公司直接与这些组织合作。*

1.  ***国际合作**，旨在最终创建一个国际[条约](https://www.cigionline.org/articles/voluntary-curbs-arent-enough-ai-risk-requires-a-binding-international-treaty/)或新的国际机构，以防止公司规避这些监管措施。最近，由 28 个国家签署的[Bletchley 宣言](https://www.gov.uk/government/publications/ai-safety-summit-2023-the-bletchley-declaration/the-bletchley-declaration-by-countries-attending-the-ai-safety-summit-1-2-november-2023)，包括世界领先 AI 公司的总部所在国（美国、中国、英国、阿拉伯联合酋长国、法国和德国）；此宣言声明了共同的价值观，并为额外会议开辟了道路。*

1.  ***通过公共基础设施实现 AI 访问民主化**：关于监管 AI 的一个普遍担忧是它将使能够生产复杂 AI 系统的公司数量限制在很少一部分，并趋向于垄断性商业惯例。然而，有许多机会可以在不依赖不安全的 AI 系统的情况下使 AI 访问民主化。其中一种方式是通过创建具有强大安全 AI 模型的[公共 AI 基础设施](https://cdn.vanderbilt.edu/vu-URL/wp-content/uploads/sites/412/2023/10/09151836/VPA-AI-Capacity.10.9.23.pdf)。*

*“我认为我们如何监管开源 AI 是眼下最重要的未解决问题，”认知科学家、企业家和纽约大学荣誉教授[加里·马库斯](http://garymarcus.com/index.html) 在最近的电子邮件交流中告诉我。*

*我同意，而且这些建议只是一个开端。最初实施这些建议将会很昂贵，并且需要监管机构让某些强大的游说者和开发者感到不满。*

*不幸的是，鉴于当前人工智能和信息生态系统中存在的利益不一致，除非被迫这样做，否则行业不太可能采取这些行动。如果不采取这样的行动，生产不安全人工智能的公司可能会获得数十亿美元的利润，同时将产品带来的风险推卸给我们所有人。*

*来自您的站点文章*

*周围的相关文章*
