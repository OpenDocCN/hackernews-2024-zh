<!--yml

类别：未分类

日期：2024-05-27 13:24:27

-->

# llama3/MODEL_CARD.md at main · meta-llama/llama3 · GitHub

> 来源：[https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md](https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md)

Meta开发并发布了Meta Llama 3系列大型语言模型（LLM），这是一系列预训练和指令调整的生成文本模型，包括8B和70B尺寸。Llama 3指令调整的模型优化了对话使用案例，性能优于许多常见工业基准上可用的开源聊天模型。在开发这些模型时，我们非常注意优化其帮助性和安全性。

**模型开发者** Meta

**变体** Llama 3有两种尺寸 — 8B和70B参数 — 分别是预训练和指令调整的变体。

**输入** 模型只接受文本输入。

**输出** 模型仅生成文本和代码。

**模型架构** Llama 3是一种自回归语言模型，使用了优化的Transformer架构。调整版本使用监督微调（SFT）和带有人类反馈的强化学习（RLHF），以与人类对帮助性和安全性的偏好对齐。

|  | **训练数据** | **参数** | **上下文长度** | **GQA** | **令牌计数** | **知识截止日期** |
| --- | --- | --- | --- | --- | --- | --- |
| Llama 3 | 一个新的混合公开可用的在线数据。 | 8B | 8k | 是 | 15T+ | 2023年3月 |
| 70B | 8k | 是 | 2023年12月 |

**Llama 3系列模型**。令牌计数仅适用于预训练数据。8B和70B版本均使用组合查询注意力（GQA）以提升推理可伸缩性。

**模型发布日期** 2024年4月18日。

**状态** 这是基于离线数据集训练的静态模型。随着社区反馈的改进，调整后的模型的未来版本将发布。

**许可证** 可以在[https://llama.meta.com/llama3/license](https://llama.meta.com/llama3/license)获取自定义商业许可证。

关于模型的问题或评论应发送到模型[README](https://github.com/meta-llama/llama3)中有关如何提供反馈或评论的说明。有关如何在应用程序中使用Llama 3的生成参数和配方的更多技术信息，请访问[此处](https://github.com/meta-llama/llama-recipes)。

**预期使用案例** Llama 3旨在商业和研究用途的英语环境中使用。指令调整的模型适用于类似助手的聊天，而预训练模型可以用于各种自然语言生成任务的适应。

**超出范围** 任何违反适用法律或法规（包括贸易合规法）的使用方式。任何其他违反[可接受使用政策](https://llama.meta.com/llama3/use-policy/)和[Llama 3社区许可证](https://llama.meta.com/llama3/license/)的使用方式。不支持其他语言**。

**注意：开发者可以根据[LLama 3社区许可证](https://llama.meta.com/llama3/license/)和[可接受使用政策](https://llama.meta.com/llama3/use-policy/)将Llama 3模型进行语言等外语的精细调整。**

**训练因素** 我们使用了自定义训练库，Meta的研究超级集群以及生产集群进行预训练。微调、注释和评估也在第三方云计算上完成。

**碳足迹预训练** 利用了7.7百万 GPU 小时的计算时间，硬件型号为 H100-80GB（TDP 为700W）。预计总排放量为2290 tCO2eq，其中100%已通过 Meta 的可持续性计划抵消。

|  | **时间 (GPU 小时)** | **功耗 (W)** | **排放碳量 (tCO2eq)** |
| --- | --- | --- | --- |
| Llama 3 8B | 1.3百万 | 700 | 390 |
| Llama 3 70B | 6.4百万 | 700 | 1900 |
| 总计 | 7.7百万 |  | 2290 |

**在预训练期间的二氧化碳排放**。时间：训练每个模型所需的总GPU时间。功耗：每个GPU设备的峰值功率容量，根据使用效率调整。所有排放量均由Meta的可持续性计划直接抵消，因为我们正在公开发布这些模型，预训练成本不需要由其他人承担。

**概览** Llama 3 是在公开来源数据的超过15万亿个标记上进行预训练的。微调数据包括公开可用的指令数据集，以及超过1000万个人工注释示例。预训练和微调数据集均不包括 Meta 用户数据。

**数据新鲜度** 8B 模型的预训练数据截止到2023年3月，70B 模型截止到2023年12月。

在本节中，我们报告了 Llama 3 模型在标准自动基准测试中的结果。对于所有评估，我们使用我们的内部评估库。有关方法论的详细信息请参见[这里](https://github.com/meta-llama/llama3/blob/main/eval_details.md)。

| **类别** | **基准** | **Llama 3 8B** | **Llama 2 7B** | **Llama 2 13B** | **Llama 3 70B** | **Llama 2 70B** |
| --- | --- | --- | --- | --- | --- | --- |
| 通用 | MMLU (5-shot) | 66.6 | 45.7 | 53.8 | 79.5 | 69.7 |
| AGIEval 英语 (3-5 shot) | 45.9 | 28.8 | 38.7 | 63.0 | 54.8 |
| 常识QA (7-shot) | 72.6 | 57.6 | 67.6 | 83.8 | 78.7 |
| Winogrande (5-shot) | 76.1 | 73.3 | 75.4 | 83.1 | 81.8 |
| BIG-Bench Hard (3-shot, CoT) | 61.1 | 38.1 | 47.0 | 81.3 | 65.7 |
| ARC-Challenge (25-shot) | 78.6 | 53.7 | 67.6 | 93.0 | 85.3 |
| 知识推理 | TriviaQA-Wiki (5-shot) | 78.5 | 72.1 | 79.6 | 89.7 | 87.5 |
| 阅读理解 | SQuAD (1-shot) | 76.4 | 72.2 | 72.1 | 85.6 | 82.6 |
| QuAC (1-shot, F1) | 44.4 | 39.6 | 44.9 | 51.1 | 49.4 |
| BoolQ (0-shot) | 75.7 | 65.5 | 66.9 | 79.0 | 73.1 |
| DROP (3-shot, F1) | 58.4 | 37.9 | 49.8 | 79.7 | 70.2 |
| **基准** | **Llama 3 8B** | **Llama 2 7B** | **Llama 2 13B** | **Llama 3 70B** | **Llama 2 70B** |
| MMLU (5-shot) | 68.4 | 34.1 | 47.8 | 82.0 | 52.9 |
| GPQA (0-shot) | 34.2 | 21.7 | 22.3 | 39.5 | 21.0 |
| HumanEval (0-shot) | 62.2 | 7.9 | 14.0 | 81.7 | 25.6 |
| GSM-8K (8-shot, CoT) | 79.6 | 25.7 | 77.4 | 93.0 | 57.5 |
| MATH (4-shot, CoT) | 30.0 | 3.8 | 6.7 | 50.4 | 11.6 |

我们相信开放的AI方法能够带来更好、更安全的产品，促进更快的创新，并扩大整体市场。我们致力于负责任的AI开发，并采取了一系列措施来限制误用和伤害，并支持开源社区。

基础模型是广泛能力的技术，旨在用于各种应用程序。它们并非被设计为即插即用地满足所有用例的每个开发者对安全级别的偏好，因为这些自然会在不同应用程序中有所不同。

相反，负责任的LLM应用部署通过在这些应用程序的开发过程中实施一系列安全最佳实践来实现，从模型预训练、微调到系统部署，以及利用保障系统以针对特定用例和受众定制安全需求。

作为Llama 3发布的一部分，我们更新了我们的[负责使用指南](https://llama.meta.com/responsible-use-guide/)，详细说明开发者为其应用实施模型和系统级安全的步骤和最佳实践。我们还提供一系列资源，包括[Meta Llama Guard 2](https://llama.meta.com/purple-llama/)和[Code Shield](https://llama.meta.com/purple-llama/)安全防护措施。这些工具已被证明能显著降低LLM系统的残留风险，同时保持高水平的实用性。我们鼓励开发者根据需要调整和部署这些安全措施，并为您提供了一个[参考实现](https://github.com/meta-llama/llama-recipes/tree/main/recipes/responsible_ai)来帮助您入门。

如《负责使用指南》中所述，模型的有用性与模型的对齐性之间的某种权衡可能是不可避免的。开发者应谨慎权衡在其特定用例和受众中对齐性和有用性的利益。在使用Llama模型时，开发者应注意残留风险，并根据需要利用额外的安全工具来达到其用例的正确安全标准。

安全

对于我们的指导调整模型，我们进行了广泛的红队演练，进行了对抗评估，并实施了安全缓解技术以降低残留风险。与任何大型语言模型一样，残留风险可能仍然存在，我们建议开发者在其用例的背景下评估这些风险。同时，我们正在与社区合作，使AI安全基准标准透明、严格且可解释。

拒绝

除了残留风险外，我们非常重视模型对良性提示的拒绝。过度拒绝不仅会影响用户体验，而且在某些情况下可能会有害。我们听取了开发者社区的反馈，并改进了我们的精细调整，以确保Llama 3在误拒答案方面的可能性显著低于Llama 2。

我们建立了内部基准，并开发了缓解措施，以限制误拒，使Llama 3成为迄今为止最有帮助的模型。

除了上述负责任使用的考虑因素外，我们遵循了严格的流程，在我们做出发布决策之前，需要我们采取额外措施来防止滥用和关键风险。

滥用

如果您访问或使用Llama 3，您同意接受可接受使用政策。您可以在[https://llama.meta.com/llama3/use-policy/](https://llama.meta.com/llama3/use-policy/)找到此政策的最新副本。

CBRNE（化学、生物、放射性、核和高产爆炸物）

我们在这一领域对模型安全性进行了两重评估：

+   在模型训练期间进行迭代测试，评估与CBRNE威胁和其他对抗性风险相关的响应的安全性。

+   通过外部CBRNE专家进行提升测试，评估模型提供专业知识的能力，并通过与Web搜索（无需模型）相比较，减少潜在CBRNE滥用的障碍。

我们使用CyberSecEval对Llama 3进行了评估，这是Meta的网络安全安全评估套件，衡量Llama 3在作为编码助手使用时建议不安全代码的倾向性，以及Llama 3在遵循行业标准MITRE ATT&CK网络攻击本体论时协助执行网络攻击的倾向性。在我们的不安全编码和网络攻击协助性测试中，Llama 3的行为范围与或者比具有[相同编码能力](https://huggingface.co/spaces/facebook/CyberSecEval)的模型更安全。

我们使用专家团队进行了儿童安全风险评估，评估模型产生可能导致儿童安全风险的输出能力，并通过精细调整提供任何必要和适当的风险缓解建议。我们利用这些专家红队会议扩展了我们通过Llama 3模型开发的评估基准的覆盖范围。对于Llama 3，我们使用基于客观方法的方法进行了新的深度评估会议，以评估模型在多个攻击向量上的风险。我们还与内容专家合作进行了红队演练，评估潜在的违规内容，同时考虑市场特定的细微差别或经验。

### 社区

生成式人工智能安全需要专业知识和工具支持，我们相信开放社区的力量可以加速其进展。我们是开放联盟的积极成员，包括AI联盟、AI合作伙伴关系和MLCommons，积极推动安全标准化和透明化。我们鼓励社区采用MLCommons概念验证评估等分类法，以促进安全和内容评估的合作和透明化。我们的Purple Llama工具已开源供社区使用，并在生态系统伙伴中广泛分发，包括云服务提供商。我们鼓励社区为我们的[GitHub存储库](https://github.com/meta-llama/PurpleLlama)做出贡献。

最后，我们设立了一系列资源，包括一个[输出报告机制](https://developers.facebook.com/llama_output_feedback)和[漏洞赏金计划](https://www.facebook.com/whitehat)，以在社区的帮助下持续改进Llama技术。

## 道德考量和限制

[](#ethical-considerations-and-limitations)

Llama 3的核心价值在于开放性、包容性和乐于助人。它旨在为所有人服务，适用于广泛的用例。因此，它设计成能够让不同背景、经验和观点的人们轻松使用。它以用户和他们的需求为中心，不插入不必要的判断或规范，同时反映出即使在某些情况下可能存在问题的内容，也能为其他情况提供有价值的用途。它尊重所有用户的尊严和自主权，特别是在自由思想和表达价值推动创新和进步的情况下。

但是Llama 3是一项新技术，像所有新技术一样，其使用存在风险。到目前为止进行的测试均为英文，并未涵盖，也无法涵盖所有情景。因此，正如所有LLM一样，Llama 3的潜在输出无法预先预测，该模型有时可能会生成不准确、带偏见或其他令人反感的响应。因此，在部署任何Llama 3模型应用程序之前，开发人员应根据其模型特定应用进行安全测试和调整。正如《责任使用指南》所述，我们建议将[Purple Llama](https://github.com/facebookresearch/PurpleLlama)解决方案整合到您的工作流程中，特别是[Llama Guard](https://ai.meta.com/research/publications/llama-guard-llm-based-input-output-safeguard-for-human-ai-conversations/)，它为输入和输出提示过滤提供了基础系统级安全性。

请参阅[责任使用指南](http://llama.meta.com/responsible-use-guide)。

```
@article{llama3modelcard,
  title={Llama 3 Model Card},
  author={AI@Meta},
  year={2024},
  url = {https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md}
} 
```

Aaditya Singh;

Aaron Grattafiori;

Abhimanyu Dubey;

Abhinav Jauhri;

Abhinav Pandey;

Abhishek Kadian;

Adam Kelsey;

Adi Gangidi;

Ahmad Al-Dahle;

Amit Sangani;

Ahuva Goldstand;

Aiesha Letman;

Ajay Menon;

Akhil Mathur;

Alan Schelten;

Alex Vaughan;

Amy Yang;

Andrei Lupu;

Andres Alvarado;

Andrew Gallagher;

Andrew Gu;

Andrew Ho;

Andrew Poulton;

Andrew Ryan;

Angela Fan;

Ankit Ramchandani;

Anthony Hartshorn;

Archi Mitra;

Archie Sravankumar;

Artem Korenev;

Arun Rao;

Ashley Gabriel;

Ashwin Bharambe;

Assaf Eisenman;

Aston Zhang;

Ash JJhaveri;

Aurelien Rodriguez;

Austen Gregerson;

Ava Spataru;

Baptiste Roziere;

Ben Maurer;

Benjamin Leonhardi;

Bernie Huang;

Bhargavi Paranjape;

Bing Liu;

Binh Tang;

Bobbie Chern;

Brani Stojkovic;

Brian Fuller;

Catalina Mejia Arenas;

Chao Zhou;

Charlotte Caucheteux;

Chaya Nayak;

Ching-Hsiang Chu;

Chloe Bi;

Chris Cai;

Chris Cox;

Chris Marra;

Chris McConnell;

Christian Keller;

Christoph Feichtenhofer;

Christophe Touret;

Chunyang Wu;

Corinne Wong;

Cristian Canton Ferrer;

Damien Allonsius;

Daniel Kreymer;

Daniel Haziza;

Daniel Li;

Danielle Pintz;

Danny Livshits;

Danny Wyatt;

David Adkins;

David Esiobu;

David Xu;

Davide Testuggine;

Delia David;

Devi Parikh;

Dhruv Choudhary;

Dhruv Mahajan;

Diana Liskovich;

Diego Garcia-Olano;

Diego Perino;

Dieuwke Hupkes;

Dingkang Wang;

Dustin Holland;

Egor Lakomkin;

Elina Lobanova;

Xiaoqing Ellen Tan;

Emily Dinan;

Eric Smith;

Erik Brinkman;

Esteban Arcaute;

Filip Radenovic;

Firat Ozgenel;

Francesco Caggioni;

Frank Seide;

Frank Zhang;

Gabriel Synnaeve;

Gabriella Schwarz;

Gabrielle Lee;

Gada Badeer;

Georgia Anderson;

Graeme Nail;

Gregoire Mialon;

Guan Pang;

Guillem Cucurell;

Hailey Nguyen;

Hamid Shojanazeri;

Hannah Korevaar;

Hannah Wang;

Haroun Habeeb;

Harrison Rudolph;

Henry Aspegren;

Hu Xu;

Hugo Touvron;

Iga Kozlowska;

Igor Molybog;

Igor Tufanov;

Iliyan Zarov;

Imanol Arrieta Ibarra;

Irina-Elena Veliche;

Isabel Kloumann;

Ishan Misra;

Ivan Evtimov;

Jade Copet;

Jake Weissman;

Jan Geffert;

Jana Vranes;

Japhet Asher;

Jason Park;

Jay Mahadeokar;

Jean-Baptiste Gaya;

Jeet Shah;

Jelmer van der Linde;

Jennifer Chan;

Jenny Hong;

Jenya Lee;

Jeremy Fu;

Jeremy Teboul;

Jianfeng Chi;

Jianyu Huang;

Jie Wang;

Jiecao Yu;

Joanna Bitton;

Joe Spisak;

Joelle Pineau;

Jon Carvill;

Jongsoo Park;

Joseph Rocca;

Joshua Johnstun;

Junteng Jia;

Kalyan Vasuden Alwala;

Kam Hou U;

Kate Plawiak;

Kartikeya Upasani;

Kaushik Veeraraghavan;

Ke Li;

Kenneth Heafield;

Kevin Stone;

Khalid El-Arini;

Krithika Iyer;

Kshitiz Malik;

Kuenley Chiu;

Kunal Bhalla;

Kyle Huang;

Lakshya Garg;

Lauren Rantala-Yeary;

Laurens van der Maaten;

Lawrence Chen;

Leandro Silva;

Lee Bell;

Lei Zhang;

Liang Tan;

Louis Martin;

Lovish Madaan;

Luca Wehrstedt;

Lukas Blecher;

Luke de Oliveira;

Madeline Muzzi;

Madian Khabsa;

Manav Avlani;

Mannat Singh;

Manohar Paluri;

Mark Zuckerberg;

Marcin Kardas;

Martynas Mankus;

Mathew Oldham;

Mathieu Rita;

Matthew Lennie;

Maya Pavlova;

Meghan Keneally;

Melanie Kambadur;

Mihir Patel;

Mikayel Samvelyan;

Mike Clark;

Mike Lewis;

Min Si;

Mitesh Kumar Singh;

Mo Metanat;

Mona Hassan;

Naman Goyal;

Narjes Torabi;

Nicolas Usunier;

Nikolay Bashlykov;

Nikolay Bogoychev;

Niladri Chatterji;

Ning Dong;

Oliver Aobo Yang;

Olivier Duchenne;

Onur Celebi;

Parth Parekh;

Patrick Alrassy;

Paul Saab;

Pavan Balaji;

Pedro Rittner;

Pengchuan Zhang;

Pengwei Li;

Petar Vasic;

Peter Weng;

Polina Zvyagina;

Prajjwal Bhargava;

Pratik Dubal;

Praveen Krishnan;

Punit Singh Koura;

Puxin Xu;

Qing He;

Rachel Rodriguez;

Ragavan Srinivasan;

Rahul Mitra;

Ramon Calderer;

Raymond Li;

Robert Stojnic;

Roberta Raileanu;

Robin Battey;

Rocky Wang;

Rohit Girdhar;

Rohit Patel;

Romain Sauvestre;

Ronnie Polidoro;

Roshan Sumbaly;

Ross Taylor;

Ruan Silva;

Rui Hou;

Rui Wang;

Russ Howes;

Ruty Rinott;

Saghar Hosseini;

Sai Jayesh Bondu;

Samyak Datta;

Sanjay Singh;

Sara Chugh;

Sargun Dhillon;

Satadru Pan;

Sean Bell;

Sergey Edunov;

Shaoliang Nie;

Sharan Narang;

Sharath Raparthy;

Shaun Lindsay;

Sheng Feng;

Sheng Shen;

Shenghao Lin;

Shiva Shankar;

Shruti Bhosale;

Shun Zhang;

Simon Vandenhende;

Sinong Wang;

Seohyun Sonia Kim;

Soumya Batra;

Sten Sootla;

Steve Kehoe;

Suchin Gururangan;

Sumit Gupta;

Sunny Virk;

Sydney Borodinsky;

Tamar Glaser;

Tamar Herman;

Tamara Best;

Tara Fowler;

Thomas Georgiou;

Thomas Scialom;

Tianhe Li;

Todor Mihaylov;

Tong Xiao;

Ujjwal Karn;

Vedanuj Goswami;

Vibhor Gupta;

Vignesh Ramanathan;

Viktor Kerkez;

Vinay Satish Kumar;

Vincent Gonguet;

Vish Vogeti;

Vlad Poenaru;

Vlad Tiberiu Mihailescu;

Vladan Petrovic;

Vladimir Ivanov;

Wei Li;

Weiwei Chu;

Wenhan Xiong;

Wenyin Fu;

Wes Bouaziz;

Whitney Meers;

Will Constable;

Xavier Martinet;

Xiaojian Wu;

Xinbo Gao;

Xinfeng Xie;

Xuchao Jia;

Yaelle Goldschlag;

Yann LeCun;

Yashesh Gaur;

Yasmine Babaei;

Ye Qi;

Yenda Li;

Yi Wen;

Yiwen Song;

Youngjin Nam;

Yuchen Hao;

Yuchen Zhang;

Yun Wang;

Yuning Mao;

Yuzi He;

Zacharie Delpierre Coudert;

Zachary DeVito;

Zahra Hankir;

Zhaoduo Wen;

Zheng Yan;

Zhengxing Chen;

Zhenyu Yang;

Zoe Papakipos
