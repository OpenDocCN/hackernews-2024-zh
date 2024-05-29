<!--yml
category: 未分类
date: 2024-05-27 14:53:52
-->

# IBM and NASA build language models to make scientific knowledge more accessible - IBM Research

> 来源：[https://research.ibm.com/blog/science-expert-LLM](https://research.ibm.com/blog/science-expert-LLM)

In a new collaboration, IBM and NASA created a suite of efficient language models by training on scientific literature. Based on the transformer architecture, these models can be used in a variety of applications, from classification and entity extraction to question-answering and information retrieval. These models achieve high performance across a variety of domains and can respond promptly. We have open-sourced the models on Hugging Face for the benefit of the scientific and academic community.

Transformer-based language models — which include [BERT](https://aclanthology.org/N19-1423/), [RoBERTa](https://arxiv.org/abs/1907.11692), and IBM’s Slate and Granite family of models, are invaluable for a range of natural language understanding tasks. What powers these models is a statistical understanding of how language works. They are trained on masked language modeling tasks, which learns by reconstructing sentences with words that have been obscured. Tokenizers, which break down words into units for the model, play a critical role in learning a vast vocabulary. While general-purpose text training is effective with popular tokenizers trained on datasets like Wikipedia or BooksCorpus, scientific domains require specialized tokenizers for terms like "phosphatidylcholine."

We trained our models on 60 billion tokens on a corpus of astrophysics, planetary science, earth science, heliophysics, and biological and physical sciences data. Unlike a generic tokenizer, the one we developed is capable of recognizing scientific terms such as "axes" and "polycrystalline." More than half of the 50,000 tokens our models processed were unique compared to the open-source RoBERTa model on Hugging Face.

The IBM-NASA models, trained on domain-specific vocabulary, outperformed the open RoBERTa model by 5% on the popular [BLURB](https://microsoft.github.io/BLURB/) benchmark, which evaluates performance on biomedical tasks. It also showed a 2.4% F1 score improvement on an internal scientific question-answering benchmark and a 5.5% improvement on internal Earth science entity recognition tests.

Our trained encoder model can be fine-tuned for many non-generative linguistic tasks and can generate information-rich embeddings for document retrieval through [retrieval augmented generation](https://arxiv.org/abs/2005.11401) (RAG). RAG commonly follows a two-step framework: a retriever model first encodes the question and retrieves relevant documents from a vector database. These documents are then passed to a generative model to answer the question while ensuring fidelity to the retrieved document.

We built a retriever model on top of our encoder model to produce information-rich embeddings that map the similarity between pairs of text. Specifically, we optimize on a contrastive loss function, pushing the embeddings of an anchor text closer to those of a relevant (“positive”) document, and farther away from a random (“negative”) document.

These models used about 268 million text pairs, including titles and abstracts, and questions and answers. As a result, they excel at retrieving relevant passages in a test set of about 400 questions that NASA curated. This is evidenced by a 6.5% improvement over a similarly fine-tuned RoBERTa model, and a 5% improvement over [BGE-base](https://huggingface.co/BAAI/bge-base-en-v1.5), another popular open-source model for embeddings.

The significant enhancements achieved by our models can be attributed to the specialized training data, custom tokenizer, and training methodology. Consistent with IBM and NASA’s commitment to open and transparent AI, both models are available on Hugging Face: the [encoder model](https://huggingface.co/nasa-impact/nasa-smd-ibm-v0.1) can be further finetuned for applications in the space domain, while the [retriever model](https://huggingface.co/nasa-impact/nasa-smd-ibm-st) can be used for information retrieval applications for RAG. We are also collaborating with NASA to enhance science search engine using these models.