<!--yml
category: 未分类
date: 2024-05-27 14:38:08
-->

# Sensors | Free Full-Text | Text-Guided Image Editing Based on Post Score for Gaining Attention on Social Media

> 来源：[https://www.mdpi.com/1424-8220/24/3/921](https://www.mdpi.com/1424-8220/24/3/921)

## 1\. Introduction

With significant advances in Internet technologies, social media has become an important part of our daily lives. Social media provides a wide range of interactions such as communicating with other users, seeking and providing news, buying and selling products, and advertising. The number of users using social media is expected to increase further in the future [

[1](#B1-sensors-24-00921)

], and interactions within social media have the potential to have a significant impact on society. Instagram, which has grown particularly rapidly in recent years, has more than 1.3 billion monthly active users in 2023 [

[2](#B2-sensors-24-00921)

]. Instagram is a platform specialized in posting images and videos and allows users to “like” and comment on posts. The extraordinary success of Instagram confirms a recent report by Pew Research Center (

[https://www.pewresearch.org/](https://www.pewresearch.org/)

(accessed on 20 November 2023)) that photos and videos have become the primary social currency online [

[3](#B3-sensors-24-00921)

]. In other words, images posted on Instagram have significant worth. To enhance the worth, there are more opportunities to edit images, and research on automatic image editing is becoming important [

[4](#B4-sensors-24-00921)

,

[5](#B5-sensors-24-00921)

,

[6](#B6-sensors-24-00921)

].

Representative automatic image editing approaches include image colorization [

[7](#B7-sensors-24-00921)

], image inpainting [

[8](#B8-sensors-24-00921)

], and style transfer [

[9](#B9-sensors-24-00921)

], which are expected to eliminate the tedious human work involved in image editing. These approaches are usually constructed for one-pattern transformations. For example, the approach of style transfer automatically transforms a style in an input image to a specific artist’s style, and there are limitations in transforming it to different artist’s styles using one model and in reflecting a user’s request into the edited image. To address these problems, several user-friendly approaches, named text-guided image editing, have been proposed [

[10](#B10-sensors-24-00921)

,

[11](#B11-sensors-24-00921)

,

[12](#B12-sensors-24-00921)

,

[13](#B13-sensors-24-00921)

,

[14](#B14-sensors-24-00921)

,

[15](#B15-sensors-24-00921)

]. Given an input image and a text prompt describing the contents of image editing from a user, text-guided image editing aims to edit the text-related region in accordance with the text prompt and maintain text-unrelated regions of the input image. With the great success of generative models such as StyleGAN [

[16](#B16-sensors-24-00921)

] and the diffusion model [

[17](#B17-sensors-24-00921)

] in recent years, text-guided image editing has become a hot topic.

While text-guided image editing based on the diffusion model achieves particularly high performance, the user must expend effort devising how to give the text prompt to obtain the desired result. Specifically, as shown in

[Figure 1](#sensors-24-00921-f001)

, the results of text-guided image editing differ depending on the way the text prompt is represented, even if it has the same meaning. It is up to the user to decide which result best matches the intended use of the edited image. When the intended use of the edited image is to post to social media, it is difficult for the user without knowledge of social media marketing to decide which result will gain attention from a greater audience. As shown in the upper case of

[Figure 2](#sensors-24-00921-f002)

, it is unclear how much attention the result of the previous text-guided image editing method would gain when it is posted on social media. Moreover, as shown in the lower case of

[Figure 2](#sensors-24-00921-f002)

, by performing text-guided image editing while considering the attention on social media in advance, the edited image can be displayed to a greater audience. From the above, it is necessary to construct a text-guided image editing method that can generate an edited image to gain attention on social media.

Here, there is much research on post and influencer classifications for social media marketing [

[18](#B18-sensors-24-00921)

,

[19](#B19-sensors-24-00921)

,

[20](#B20-sensors-24-00921)

,

[21](#B21-sensors-24-00921)

]. In post classification, several works [

[18](#B18-sensors-24-00921)

,

[19](#B19-sensors-24-00921)

] focus on the categories (e.g., fashion, travel, and food) and the virality (a situation in which a post gets more interactions than others on Twitter (currently X) and thus gains the attention from a relatively large number of users [

[19](#B19-sensors-24-00921)

]) of posts on social media. In particular, the research classifying the virality of posts is useful in selecting edited images for posting on social media from the multiple results of text-guided image editing. However, since the conventional method [

[19](#B19-sensors-24-00921)

] predicts virality only from posted text, it is impossible to predict the attention of edited images on social media in advance. Therefore, it is necessary to construct a new model that predicts in advance the attention from the audience on social media based on the posted image in addition to the posted text.

In light of the above, we propose novel text-guided image editing considering the response in social media in this paper. The goal of the proposed method is to provide the user with images that are edited in accordance with the text prompt and will gain attention from the greater audience on social media. Here, the degree of attention from the audience is defined as the engagement rate calculated from the number of likes and comments on a post. The key idea of the proposed method is to newly introduce a model to predict post scores representing engagement rates on social media from posted images and text, thereby generating edited images that gain attention from the greater audience. To construct the novel model, based on previous works [

[22](#B22-sensors-24-00921)

,

[23](#B23-sensors-24-00921)

] that analyzed the relationship between the content or aesthetic of posted images and the engagement rate from the perspective of computer vision, the proposed method focuses on aesthetics and categories of the posted images in addition to features of posted images and texts and calculates post scores. Then, the proposed method obtains several other expressions similar to the text prompt given by the user, based on a large language model that has attracted much attention in recent years. We apply a pre-trained text-guided image editing method and generate edited images from each of these several text prompts. Using multiple text prompts increases the possibility of obtaining edited images that gain attention from a greater audience on social media while performing image editing desired by the user. Among these, by leveraging the novel model that predicts post scores representing engagement rates, we finally select the edited images that will gain more attention from the audience on social media. To the best of our knowledge, this is the first text-guided image editing method considering the response from the audience on social media. The proposed method can provide users with edited images that have the potential to obtain the highest engagement rate and is expected to reduce the burden of creating posts for users without knowledge of social media marketing.

The rest of this paper is organized as follows. We introduce related works on social media marketing, image–text matching, and text-guided image editing in

[Section 2](#sec2-sensors-24-00921)

. In

[Section 3](#sec3-sensors-24-00921)

, we then explain the proposed method that performs text-guided image editing considering the response in social media. In

[Section 4](#sec4-sensors-24-00921)

, as a preliminary validation, we verify the accuracy of the proposed model to predict post scores.

[Section 5](#sec5-sensors-24-00921)

demonstrates extensive experimental results for verifying the effectiveness of the proposed method. Finally, we conclude our work in

[Section 6](#sec6-sensors-24-00921)

.

## 3\. Proposed Text-Guided Image Editing Based on Post Score for Gaining Attention on Social Media

We show the architecture of the proposed method in

[Figure 3](#sensors-24-00921-f003)

. In the proposed method, we first obtain four paraphrase expressions for the text prompt

P

. Then, the proposed method applies the pre-trained text-guided image editing method (i.e., Instruct-Pix2Pix) and generates four edited images. Finally, we leverage the proposed model to predict the post score and select the edited image

<math display="inline"><semantics><msup><mi>I</mi> <mo>′</mo></msup></semantics></math>

with the highest post score from all results of the text-guided image editing method.

In the proposed model, we calculate the post score representing the engagement rate on social media from the posted image and text, as shown in

[Figure 4](#sensors-24-00921-f004)

. Specifically, based on previous works [

[22](#B22-sensors-24-00921)

,

[23](#B23-sensors-24-00921)

] that analyzed the relationship between the content or aesthetic of posted images and the engagement rate, we focus on aesthetics and categories of the posted images in addition to features of posted images and texts and calculate post scores. We describe the details of the proposed model to predict post scores in

[Section 3.1](#sec3dot1-sensors-24-00921)

. Then, the overall flow of the proposed method applying that model is described in

[Section 3.2](#sec3dot2-sensors-24-00921)

.

#### 3.1\. Calculation of Post Score Representing Engagement Rate on Social Media

This section explains the proposed model that takes an image

I

and a text

T

as inputs and predicts a post score representing the engagement rate when they are posted on social media. As shown in

[Figure 4](#sensors-24-00921-f004)

, to predict the post score, the proposed model uses three features (1) a multimodal feature extracted from the image

I

and text

T

, (2) an aesthetics feature of the image

I

, and (3) a category feature (e.g., fashion, travel, and food).

**Figure 4.** Details of the proposed model to predict post score. To calculate the post score, the proposed model predicts the class probability of the engagement rate using three integrated features <math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math> .

**Figure 4.** Details of the proposed model to predict post score. To calculate the post score, the proposed model predicts the class probability of the engagement rate using three integrated features <math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math> .

#### 3.1.1\. Calculation of Post Score Using Multiple Features

To obtain the multimodal feature, we utilize contrastive language-image pre-training (CLIP) [

[37](#B37-sensors-24-00921)

]. CLIP includes two neural networks of an image encoder

<math display="inline"><semantics><mrow><msub><mi>CLIP</mi> <mi>image</mi></msub> <mrow><mo>(</mo> <mo>·</mo> <mo>)</mo></mrow></mrow></semantics></math>

and a text encoder

<math display="inline"><semantics><mrow><msub><mi>CLIP</mi> <mi>text</mi></msub> <mrow><mo>(</mo> <mo>·</mo> <mo>)</mo></mrow></mrow></semantics></math>

. CLIP is trained on a very large image–text pair dataset, thus extracting image and text features in highly expressive spaces. The proposed method obtains features

<math display="inline"><semantics><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mi>D</mi></msup></mrow></semantics></math>

and

<math display="inline"><semantics><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>T</mi></msup> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mi>D</mi></msup></mrow></semantics></math>

extracted from the image

I

and text

T

in the CLIP space as follows:

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <mo>=</mo> <msub><mi>CLIP</mi> <mi>image</mi></msub> <mrow><mo>(</mo> <mi>I</mi> <mo>)</mo></mrow> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>T</mi></msup> <mo>=</mo> <msub><mi>CLIP</mi> <mi>text</mi></msub> <mrow><mo>(</mo> <mi>T</mi> <mo>)</mo></mrow> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

After deriving the image and text features, we concatenate these features and calculate the final multimodal feature

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">m</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mrow><mn>2</mn> <mi>D</mi></mrow></msup></mrow></semantics></math>

as follows:

This multimodal feature

<math display="inline"><semantics><mi mathvariant="bold-italic">m</mi></semantics></math>

is an important factor in predicting post scores because it provides a comprehensive representation of the image

I

and text

T

contained in a post.

With a previous work [

[23](#B23-sensors-24-00921)

] that clears the relationship between the aesthetic of posted images and the engagement rates, we calculate an aesthetics feature of the image

I

to predict post scores. To calculate the aesthetics feature, we apply a neural image assessment (NIMA) model [

[54](#B54-sensors-24-00921)

]. NIMA can predict the distribution of human opinion scores for image aesthetics using convolutional neural networks. The NIMA model takes an image as input and outputs a 10-dimensional softmaxed probability distribution representing the quality score. We use this distribution, which is output by inputting the image

I

into the NIMA model, as the aesthetic feature

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">n</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mn>10</mn></msup></mrow></semantics></math>

in the proposed model to predict the post score. This feature

<math display="inline"><semantics><mi mathvariant="bold-italic">n</mi></semantics></math>

can represent the aesthetics of the image

I

, which potentially has a significant impact on the overall quality of the posts.

Considering the relationship between post categories and the engagement rates [

[22](#B22-sensors-24-00921)

], we additionally focus on a category feature. Following [

[18](#B18-sensors-24-00921)

], we distinguish eight categories of posts: beauty, family, fashion, fitness, food, interior, pet, and travel. However, since manually labeling posts into those eight categories requires a great deal of effort, the proposed model leverages the power of CLIP. Specifically, we first calculate the text features of the eight texts

<math display="inline"><semantics><msubsup><mrow><mo>{</mo> <msub><mi>C</mi> <mi>i</mi></msub> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup></semantics></math>

representing the category names in the CLIP space as follows:

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msubsup><mrow><mo>{</mo> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup> <mo>=</mo> <msubsup><mrow><mo>{</mo> <msub><mi>CLIP</mi> <mi>text</mi></msub> <mrow><mo>(</mo> <msub><mi>C</mi> <mi>i</mi></msub> <mo>)</mo></mrow> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup> <mo>.</mo></mrow></mtd></mtr></mtable></semantics></math>

The proposed model calculates the cosine similarity

<math display="inline"><semantics><msub><mi>s</mi> <mi>i</mi></msub></semantics></math>

between the text feature

<math display="inline"><semantics><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup></semantics></math>

and the image feature

<math display="inline"><semantics><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup></semantics></math>

obtained in Equation (

[1](#FD1-sensors-24-00921)

) as follows:

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msub><mi>s</mi> <mi>i</mi></msub> <mo>=</mo> <mfrac><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <mo>·</mo> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup></mrow> <mrow><mrow><mo>|</mo> <mo>|</mo></mrow> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <msub><mrow><mo>|</mo> <mo>|</mo></mrow> <mn>2</mn></msub> <mrow><mo>|</mo> <mo>|</mo></mrow> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup> <msub><mrow><mo>|</mo> <mo>|</mo></mrow> <mn>2</mn></msub></mrow></mfrac> <mo>.</mo></mrow></mtd></mtr></mtable></semantics></math>

We obtain a vector

<math display="inline"><semantics><mi mathvariant="bold-italic">s</mi></semantics></math>

representing the category of the image

I

by concatenating the calculated cosine similarities

<math display="inline"><semantics><msubsup><mrow><mo>{</mo> <msub><mi>s</mi> <mi>i</mi></msub> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup></semantics></math>

as follows:

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi mathvariant="bold-italic">s</mi> <mo>=</mo> <mo>[</mo> <msub><mi>s</mi> <mn>1</mn></msub> <mo>;</mo> <msub><mi>s</mi> <mn>2</mn></msub> <mo>;</mo> <mo>…</mo> <mo>;</mo> <msub><mi>s</mi> <mn>8</mn></msub> <mo>]</mo> <mo>.</mo></mrow></mtd></mtr></mtable></semantics></math>

The proposed model finally obtains the category feature

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">c</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mn>8</mn></msup></mrow></semantics></math>

by applying the softmax function to the vector

<math display="inline"><semantics><mi mathvariant="bold-italic">s</mi></semantics></math>

. The feature

<math display="inline"><semantics><mi mathvariant="bold-italic">c</mi></semantics></math>

can represent the category to which the image

I

belongs without manual labeling.

The proposed model predicts the class of the engagement rate based on the three calculated features

<math display="inline"><semantics><mi mathvariant="bold-italic">m</mi></semantics></math>

,

<math display="inline"><semantics><mi mathvariant="bold-italic">n</mi></semantics></math>

, and

<math display="inline"><semantics><mi mathvariant="bold-italic">c</mi></semantics></math>

. Specifically, we use the three features and first calculate the concatenated feature

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">h</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mrow><mn>2</mn> <mi>D</mi> <mo>+</mo> <mn>18</mn></mrow></msup></mrow></semantics></math>

as follows:

This concatenated feature

<math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math>

can consider the aesthetics and categories of the image

I

while comprehensively representing the image

I

and text

T

contained in the post. The feature

<math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math>

is passed through a fully connected layer to obtain the class distribution

<math display="inline"><semantics><mover accent="true"><mi mathvariant="bold-italic">y</mi> <mo stretchy="false">^</mo></mover></semantics></math>

of the engagement rate. By using

<math display="inline"><semantics><mover accent="true"><mi mathvariant="bold-italic">y</mi> <mo stretchy="false">^</mo></mover></semantics></math>

, the proposed model calculates the post score as follows:

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>Post</mi> <mi>score</mi> <mo>=</mo> <munderover><mo>∑</mo> <mrow><mi>k</mi> <mo>=</mo> <mn>1</mn></mrow> <mi>K</mi></munderover> <mrow><mo>(</mo> <mi>k</mi> <mo>−</mo> <mn>1</mn> <mo>)</mo></mrow> <mo>×</mo> <msub><mover accent="true"><mi>y</mi> <mo stretchy="false">^</mo></mover> <mi>k</mi></msub> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

where

<math display="inline"><semantics><msub><mover accent="true"><mi>y</mi> <mo stretchy="false">^</mo></mover> <mi>k</mi></msub></semantics></math>

is the

k

th elements of the predicted class distribution

<math display="inline"><semantics><mover accent="true"><mi mathvariant="bold-italic">y</mi> <mo stretchy="false">^</mo></mover></semantics></math>

, and

K

is the number of classes for engagement rates. The more skewed the distribution is toward classes with high engagement rates, the higher the post score.

#### 3.1.2\. Loss Function

To predict the class distribution of the engagement rate, we apply cross entropy [

[55](#B55-sensors-24-00921)

] as a loss function and treat the classification task for the class of the engagement rate. The loss function

<math display="inline"><semantics><mi mathvariant="script">L</mi></semantics></math>

is described as follows:

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi mathvariant="script">L</mi> <mo>=</mo> <mo>−</mo> <munderover><mo>∑</mo> <mrow><mi>k</mi> <mo>=</mo> <mn>1</mn></mrow> <mi>K</mi></munderover> <msub><mi>y</mi> <mi>k</mi></msub> <mi>log</mi> <msub><mover accent="true"><mi>y</mi> <mo stretchy="false">^</mo></mover> <mi>k</mi></msub> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

where

<math display="inline"><semantics><msub><mi>y</mi> <mi>k</mi></msub></semantics></math>

is the

k

th elements of the ground truth class distribution

<math display="inline"><semantics><mi mathvariant="bold-italic">y</mi></semantics></math>

. By minimizing the loss calculated from this function

<math display="inline"><semantics><mi mathvariant="script">L</mi></semantics></math>

, it is possible to predict the class distribution of the engagement rate given the image

I

and text

T

as inputs.

#### 3.2\. Editing of Post Image Based on Post Score

To obtain multiple results of text-guided image editing, the proposed method first leverages a recently proposed large language model (i.e., GPT-3 [

[53](#B53-sensors-24-00921)

]). We create the sentence “Generate four different paraphrases of

<math display="inline"><semantics><mrow><mo>{</mo> <mi>P</mi> <mo>}</mo></mrow></semantics></math>

”. using the text prompt

P

to be used for image editing. The proposed method can obtain four representations similar to the text prompt

P

by giving the created sentence to the large language model. Using multiple text prompts increases the possibility of obtaining edited images that gain attention from a greater audience on social media while performing image editing desired by the user. The proposed method uses the four representations similar to the text prompt

P

and generates four edited images based on the text-guided image editing method [

[15](#B15-sensors-24-00921)

].

To select the edited images that will receive the attention from a greater audience on social media, we apply the proposed model to predict post scores constructed in

[Section 3.1](#sec3dot1-sensors-24-00921)

. The proposed method inputs the text to be posted and the four edited images, respectively, into that model and obtains four post scores. Finally, the proposed method selects the edited image

<math display="inline"><semantics><msup><mi>I</mi> <mo>′</mo></msup></semantics></math>

with the highest post score from all edited images. From the above, it is possible to generate the image

<math display="inline"><semantics><msup><mi>I</mi> <mo>′</mo></msup></semantics></math>

that is performed text-guided image editing and will gain attention on social media. By using the proposed method, the burden of using text-guided image editing can be reduced for users without knowledge of social media marketing.

## 6\. Conclusions

This paper has proposed a novel text-guided image editing method based on post scores for gaining attention on social media. The proposed method newly introduces the model to predict post scores on social media from posted images and text, thereby generating edited images that gain much attention from the audience. In the proposed method, we apply the pre-trained text-guided image editing method and obtain multiple edited images from the multiple text prompts generated from the large language model. Among these, leveraging the novel model that predicts post scores representing engagement rates, the proposed method selects the edited images that will gain the most attention from the audience on social media. Results of subjective experiments demonstrated that the edited images generated from the proposed method accurately reflect the content of the text prompts and provide a positive impression to the audience on social media. These results are supported by the subjective evaluation that subjects are most willing to give a “like” or comment when they find posts including edited images generated from the proposed method on social media.

In the future, we will improve the proposed model to predict the post score by devising a new integration technique for the image and text features. Also, the accuracy of image editing can be ensured by selecting edited images based on scores calculated from the evaluation metrics of text-guided image editing in addition to post scores, enhancing the overall performance of the proposed method.