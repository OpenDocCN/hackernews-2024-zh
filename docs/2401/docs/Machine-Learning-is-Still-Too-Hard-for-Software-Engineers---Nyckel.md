<!--yml
category: 未分类
date: 2024-05-27 15:02:33
-->

# Machine Learning is Still Too Hard for Software Engineers | Nyckel

> 来源：[https://www.nyckel.com/blog/machine-learning-difficulties/](https://www.nyckel.com/blog/machine-learning-difficulties/)

For a software engineer, the hardest thing about developing Machine Learning functionality *should* be finding clean and representative ground-truth data, but it often isn’t. If you already have a source of good quality data (perhaps because it is already gathered by your application), here are some obstacles that still lay ahead of you:

## Learning ML

ML concepts are hard to understand. The industry is still mostly in the research phase, with the beginnings of a movement towards making systems and learning resources consumable by the average developer. Books and libraries like [FastAI](https://www.fast.ai/) are examples of this movement, but it still takes days or weeks to learn how to do the most basic things.

To do something basic like [image classification](/blog/image-classification/), you have to:

*   Understand concepts like tensors, loss functions, transfer-learning, logistic regression, network fine-tuning, hyper-parameter search, over-fitting, active learning, regularization, and quantization.
*   Get familiar with one or more ML libraries like PyTorch, Tensorflow, FastAI, or scikit-learn. This is harder than getting familiar with a normal programming library because the concepts and paradigms are very different from what programmers are used to.
*   Find state-of-the-art (SOTA) deep neural networks for images from research and industry. Continue to search for new SOTA networks to keep up with industry improvements.
*   Make sure the deep networks are pre-trained on the appropriate corpus of data. Training networks from scratch is not a good idea, and the type of data used for pre-training really matters.

## Software

You’ll need software to for data exploration and management to do things like:

*   Visualize key data statistics, like relative frequency of classes
*   Find similar/different data samples
*   Mine for instances of rare classes
*   Debug data labeling mistakes
*   Implement ‘active learning’ - focus on labeling high-value samples to reduce the amount of data you have to label

You’ll also need software to track various experiments and resulting model accuracies, and to monitor the performance of your model after you start using it in production.

There are companies that solve parts of these - like [Aquarium Learning](https://www.aquariumlearning.com/) for data management and [Weights and Biases](https://wandb.ai/site) for experiment tracking. You’ll have to evaluate these and stitch together a solution that involves multiple vendors and systems. Also, as you can probably see from the data management list above, things like active learning and finding labeling mistakes require interfacing with the ML model being developed.

## Infrastructure and MLOps

Once you’ve figured out the ML and software bits, you’ll need cloud infrastructure expertise in order to:

*   Evaluate various hardware choices for cost-performance trade-offs for both training and inference.
*   Set up a fast, elastic, and cost-effective training pipeline. To provide an interactive and responsive experience when training and re-training during active learning, the speed of scaling infrastructure up and down is important.
*   Set up low-latency, elastic, highly-available, and cost-effective inference close to your data.
*   Monitor cloud systems and software for performance, availability, and utilization.
*   Have a way to annotate thousands to hundreds of thousands of samples for training. For instance, our [recycling tool](/pretrained-classifiers/recycling-identifier/) involved tagging 10,000 samples with their specific material and cross-referencing that with whether that material was recyclable or not.

These concerns are usually called “MLOps”. There are various companies, like [Valohai](https://valohai.com/) and [Determined](https://www.determined.ai/) that will solve some or all of it for you. You’ll have to evaluate them and figure out how they fit into your larger system.

## System Maintenance

If you’re lucky enough to get this far, you’re left with a large system stitched together from multiple components that you have to operate, maintain, and tweak. The maintenance applies not just to the software and infrastructure, but also to the ML pieces that have to be kept abreast of the latest ML techniques in order to get competitive performance.

## What This Means

Together, these present a formidable barrier of entry to developing ML functionality. Yes, you will scale that barrier if the incentives are sufficiently lucrative. But we’ve heard from ML teams at fairly large companies that even they don’t attempt some projects because the benefits don’t outweigh this cost. Even answering the question “*Can* ML solve my problem?” requires you to overcome half of the challenges outlined above. Imagine all the missed opportunities and problems left unsolved, especially at innovative small companies and startups.

## What the Future Looks Like

The story of software is about lowering barriers like these, leading to inventive developers solving problems that were not even considered before. Nyckel was born out of our frustration with trying to solve a relatively simple problem (custom curation of user-generated text) using ML, and we want to continue this story.