<!--yml
category: 未分类
date: 2024-05-27 13:06:56
-->

# color image classification

> 来源：[https://www.rrighart.com/blog-color-tags.html](https://www.rrighart.com/blog-color-tags.html)

Tags or labels are very useful for finding pictures on the web. Color tags are used in e-commerce applications, where these tags can select between products, for example green or red shoes.

To make a search engine work, each image needs to have tags in its metadata. This can be done using machine learning. But why would you use an algorithm if you can label images by hand?

If you have only 100 photos, manual labelling is no problem.
However, if you have 100 new photos every day or every hour, the repeated task of labelling becomes cumbersome. 

Automating this process using machine learning would save a lot of human effort that instead can be spent on other relevant tasks. In addition, it will take away the subjective bias and errors in judging colors. Actually, individual and gender differences have been found in the way humans perceive, name and classify colors, and there are as well discrepancies in color classification between speakers of different languages

[1, 2]

. Not to mention that 8% of men and 0.5% of women suffer from color vision deficiency

[3]

.

Automating classification by a system is not a walk in the park. There are in fact a number of conditions that hinder product color classification by machine learning.

First, photos have different visual quality such as brightness, contrast and saturation. The training set should account for these differences. If the photos used for training were taken inside, there is a risk that the model will not recognize photos that were taken outside since several conditions  may differ (light, shadow). In other words, the testset (or data in production) should mirror those used during training.

Second, there are a wide variety of product classes such as cars, clothing, furniture etc. depending on the kind of e-commerce shop you run. The question is how much product variety a model should account for during color tagging.

Third, there is a large diversity of colors that exist in the world and one needs to select which classes need to be recognized by the system. For example, the principal colors green and blue seem obvious choices but what about the color called turquoise that is inbetween green and blue? Therefore, a careful examination of the production context is needed.

Even though it is not a walk in the park, we can come a long way if we persist.

Using the Vehicle color dataset

[5]

, I have been able to create a machine learning model that is pretty well able to classify colors of products. The script is freely available at Kaggle. The underlying model was trained using a basic Convolutional Neural Network and an EfficientNet, with the latter giving slightly better results.

As you will see, some of the tag names seem very specific for cars, such as tan and gold. On the other hand, testing it on other non-vehicle product categories

[6]

, the color classifier seems to be doing a pretty good job. I however need to test this more extensively.

To see the results in an interface, I have created an App where you can play with your own pictures (

[https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data](https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data)

). This App was built mainly to interact with the model. In production, probably it is best to integrate the model in a data pipeline. This would rapidly tag a large batch of new pictures and store the tags in a database.

Please feel free to share your ideas or send any questions!

**Contact**
Ruthger Righart
Self-employed data scientist in machine learning and computer vision
Email: rrighart@googlemail.com
Web: [https://www.rrighart.com](https://www.rrighart.com)**References**

1\.

In the eye of the beholder : is color classification consistent among human observers ?[https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.8093](https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.8093)

2\. Differences in color categorization manifested by males and females : a quantitative World Color Survey study.

[https://www.nature.com/articles/s41599-019-0341-7](https://www.nature.com/articles/s41599-019-0341-7)

3\. Colour blindness.

[https://www.nhs.uk/conditions/colour-vision-deficiency/](https://www.nhs.uk/conditions/colour-vision-deficiency/)

4\. Kaggle page.

[https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data](https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data)

.

5\. VCoR.

[https://www.kaggle.com/datasets/landrykezebou/vcor-vehicle-color-recognition-dataset](https://www.kaggle.com/datasets/landrykezebou/vcor-vehicle-color-recognition-dataset)

6\. Fashion Product Images.

[https://www.kaggle.com/datasets/paramaggarwal/fashion-product-images-small](https://www.kaggle.com/datasets/paramaggarwal/fashion-product-images-small)

App:

[https://huggingface.co/spaces/rrighart/color-tags](https://huggingface.co/spaces/rrighart/color-tags)

Kaggle notebook:

[https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags](https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags)