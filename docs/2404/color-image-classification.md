<!--yml

category: 未分类

date: 2024-05-27 13:06:56

-->

# 彩色图像分类

> 来源：[https://www.rrighart.com/blog-color-tags.html](https://www.rrighart.com/blog-color-tags.html)

标签或标签在网上查找图片时非常有用。颜色标签在电子商务应用中使用，这些标签可以在产品之间进行选择，例如绿色或红色的鞋子。

要使搜索引擎工作，每张图像的元数据都需要有标签。这可以通过机器学习来实现。但是，如果可以手动标记图像，为什么还要使用算法呢？

如果只有100张照片，手动标记不成问题。

然而，如果每天或每小时有100张新照片，那么重复标记的任务变得繁琐。

使用机器学习自动化此过程将节省大量人力，并可用于其他相关任务。此外，这将消除主观偏见和颜色判断中的错误。实际上，已发现个体和性别在人类感知、命名和分类颜色方面存在差异，并且不同语言的讲话者在颜色分类上也存在差异。

[1, 2]

. 更不用说8%的男性和0.5%的女性患有色觉缺陷

[3]

。

通过系统自动分类并不是一帆风顺的事情。事实上，有许多条件阻碍了机器学习对产品颜色进行分类。

首先，照片的视觉质量有所不同，如亮度、对比度和饱和度。训练集应考虑这些差异。如果用于训练的照片是在室内拍摄的，那么有可能模型无法识别在室外拍摄的照片，因为条件可能不同（光线、阴影）。换句话说，测试集（或生产数据）应反映训练中使用的数据。

第二，根据您运营的电子商务店铺类型，产品类别种类多种多样，如汽车、服装、家具等。问题是在颜色标记过程中模型应考虑多少种产品种类。

第三，世界上存在大量的颜色，需要选择系统需要识别的哪些类别。例如，主要的颜色绿色和蓝色似乎是显而易见的选择，但是介于绿色和蓝色之间的颜色称为绿松石色，那该如何处理？因此，需要对生产环境进行仔细的检查。

虽然这不是一帆风顺的事情，但如果我们坚持下去，我们可以走得更远。

使用车辆颜色数据集

[5]

, 我已经能够创建一个能够相当好地分类产品颜色的机器学习模型。该脚本可以在Kaggle上免费获取。底层模型是使用基本的卷积神经网络和EfficientNet进行训练的，后者的效果略好一些。

正如您将看到的，一些标签名称对汽车非常具体，例如浅棕色和金色。另一方面，在其他非汽车产品类别上进行测试

[6]

，颜色分类器似乎做得相当不错。但是，我需要更广泛地测试这一点。

要在界面中查看结果，我创建了一个应用程序，您可以在其中使用自己的图片（

[https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data](https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data)

）。这款应用程序主要是为了与模型互动而构建的。在生产中，最好将模型集成到数据管道中。这样可以快速为大批新图片打标签并将标签存储在数据库中。

请随时分享您的想法或发送任何问题！

**联系方式**

Ruthger Righart

自由职业的机器学习和计算机视觉数据科学家

电子邮件：rrighart@googlemail.com

网站：[https://www.rrighart.com](https://www.rrighart.com)**参考文献**

1\.

在观察者眼中：颜色分类在人类观察者中是否一致？[https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.8093](https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.8093)

2\. 男性和女性在颜色分类上的差异：量化的世界颜色调查研究。

[https://www.nature.com/articles/s41599-019-0341-7](https://www.nature.com/articles/s41599-019-0341-7)

3\. 色盲。

[https://www.nhs.uk/conditions/colour-vision-deficiency/](https://www.nhs.uk/conditions/colour-vision-deficiency/)

4\. Kaggle 页面。

[https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data](https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags/data)

。

5\. VCoR。

[https://www.kaggle.com/datasets/landrykezebou/vcor-vehicle-color-recognition-dataset](https://www.kaggle.com/datasets/landrykezebou/vcor-vehicle-color-recognition-dataset)

6\. 时尚产品图像。

[https://www.kaggle.com/datasets/paramaggarwal/fashion-product-images-small](https://www.kaggle.com/datasets/paramaggarwal/fashion-product-images-small)

应用程序：

[https://huggingface.co/spaces/rrighart/color-tags](https://huggingface.co/spaces/rrighart/color-tags)

Kaggle 笔记本：

[https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags](https://www.kaggle.com/code/rrighart/the-prediction-of-color-tags)
