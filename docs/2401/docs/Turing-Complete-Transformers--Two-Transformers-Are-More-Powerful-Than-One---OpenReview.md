<!--yml
category: 未分类
date: 2024-05-27 14:35:37
-->

# Turing Complete Transformers: Two Transformers Are More Powerful Than One | OpenReview

> 来源：[https://openreview.net/forum?id=MGWsPGogLH](https://openreview.net/forum?id=MGWsPGogLH)

**Primary Area:** general machine learning (i.e., none of the above)

**Code Of Ethics:** I acknowledge that I and all co-authors of this work have read and commit to adhering to the ICLR Code of Ethics.

**Keywords:** transformers, computational complexity, computation, generalization, agents, multi-model

**Submission Guidelines:** I certify that this submission complies with the submission instructions as described on https://iclr.cc/Conferences/2024/AuthorGuide.

**TL;DR:** We prove transformers are not Turing complete, propose a new architecture that is Turing complete, and empirically demonstrate that the new architecture can generalize more effectively than transformers.

**Abstract:** This paper presents Find+Replace transformers, a family of multi-transformer architectures that can provably do things no single transformer can, and which outperforms GPT-4 on several challenging tasks. We first establish that traditional transformers and similar architectures are not Turing Complete, while Find+Replace transformers are. Using this fact, we show how arbitrary programs can be compiled into Find+Replace transformers, potentially aiding interpretability research. We also demonstrate the superior performance of Find+Replace transformers over GPT-4 on a set of composition challenge problems. This work aims to provide a theoretical basis for multi-transformer architectures, and to encourage their further exploration.

**Anonymous Url:** I certify that there is no URL (e.g., github page) that could be used to find authors' identity.

**No Acknowledgement Section:** I certify that there is no acknowledgement section in this submission for double blind review.

**Submission Number:** 8781