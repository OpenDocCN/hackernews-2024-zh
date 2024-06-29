<!--yml

category: 未分类

日期：2024-05-27 14:41:22

-->

# [社会学习：与大型语言模型的协作学习](https://arxiv.org/pdf/2312.11441.pdf)

> 来源：[https://blog.research.google/2024/03/social-learning-collaborative-learning.html?m=1](https://blog.research.google/2024/03/social-learning-collaborative-learning.html?m=1)

大型语言模型（LLMs）显著提高了使用自然语言解决指定任务的现有技术水平，通常达到接近人类的表现。随着这些模型越来越多地启用辅助代理，让它们像人类在社会环境中一样有效地相互学习可能是有益的，这将使基于LLM的代理能够改进彼此的性能。

讨论人类学习过程时，班杜拉和沃尔特斯于1977年[描述了](https://books.google.ch/books/about/Social_Learning_Theory.html?id=IXvuAAAAMAAJ&redir_esc=y)社会学习的概念，概述了人们使用的观察学习不同模型。从他人学习的一种常见方法是通过*口头指导*（例如来自教师的指导），描述如何进行特定行为。另一种方式是通过*实时模型*，模仿行为的现场示范来学习。

鉴于LLM（大型语言模型）在模仿人类交流方面取得的成功，在我们的论文“[社会学习：走向与大型语言模型的协作学习](https://arxiv.org/pdf/2312.11441.pdf)”中，我们调查了LLM是否能够通过社会学习互相学习。为此，我们提出了一个隐私意识的社会学习框架，其中LLM以自然语言的方式分享知识。我们评估了我们框架在各种数据集上的有效性，并提出了衡量这一设置中隐私的定量方法。与以往的协作学习方法（例如常见的[联邦学习](https://blog.research.google/2017/04/federated-learning-collaborative.html)方法通常依赖于梯度）相比，在我们的框架中，代理通过纯粹使用自然语言相互教导。
