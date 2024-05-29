<!--yml
category: 未分类
date: 2024-05-27 14:35:39
-->

# Large language models can do jaw-dropping things. But nobody knows exactly why. | MIT Technology Review

> 来源：[https://www.technologyreview.com/2024/03/04/1089403/large-language-models-amazing-but-nobody-knows-why/](https://www.technologyreview.com/2024/03/04/1089403/large-language-models-amazing-but-nobody-knows-why/)

“These are exciting times,” says Boaz Barak, a computer scientist at Harvard University who is on secondment to [OpenAI’s superalignment team](https://www.technologyreview.com/2023/12/14/1085344/openai-super-alignment-rogue-agi-gpt-4/) for a year. “Many people in the field often compare it to physics at the beginning of the 20th century. We have a lot of experimental results that we don’t completely understand, and often when you do an experiment it surprises you.”

### **Old code, new tricks**

Most of the surprises concern the way models can learn to do things that they have not been shown how to do. Known as generalization, this is one of the most fundamental ideas in machine learning—and its greatest puzzle. Models learn to do a task—spot faces, translate sentences, avoid pedestrians—by training with a specific set of examples. Yet they can generalize, learning to do that task with examples they have not seen before. Somehow, models do not just memorize patterns they have seen but come up with rules that let them apply those patterns to new cases. And sometimes, as with grokking, generalization happens when we don’t expect it to. 

Large language models in particular, such as OpenAI’s GPT-4 and Google DeepMind’s Gemini, have an astonishing ability to generalize. “The magic is not that the model can learn math problems in English and then generalize to new math problems in English,” says Barak, “but that the model can learn math problems in English, then see some French literature, and from that generalize to solving math problems in French. That’s something beyond what statistics can tell you about.”

When Zhou started studying AI a few years ago, she was struck by the way her teachers focused on the how but not the why. “It was like, here is how you train these models and then here’s the result,” she says. “But it wasn’t clear why this process leads to models that are capable of doing these amazing things.” She wanted to know more, but she was told there weren’t good answers: “My assumption was that scientists know what they’re doing. Like, they’d get the theories and then they’d build the models. That wasn’t the case at all.”

The rapid advances in deep learning over the last 10-plus years came more from trial and error than from understanding. Researchers copied what worked for others and tacked on innovations of their own. There are now many different ingredients that can be added to models and a growing cookbook filled with recipes for using them. “People try this thing, that thing, all these tricks,” says Belkin. “Some are important. Some are probably not.”

“It works, which is amazing. Our minds are blown by how powerful these things are,” he says. And yet for all their success, the recipes are more alchemy than chemistry: “We figured out certain incantations at midnight after mixing up some ingredients,” he says.

### **Overfitting**

The problem is that AI in the era of large language models appears to defy textbook statistics. The most powerful models today are vast, with up to a trillion parameters (the values in a model that get adjusted during training). But statistics says that as models get bigger, they should first improve in performance but then get worse. This is because of something called overfitting.

When a model gets trained on a data set, it tries to fit that data to a pattern. Picture a bunch of data points plotted on a chart. A pattern that fits the data can be represented on that chart as a line running through the points. The process of training a model can be thought of as getting it to find a line that fits the training data (the dots already on the chart) but also fits new data (new dots).