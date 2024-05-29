<!--yml
category: 未分类
date: 2024-05-27 14:25:45
-->

# Images altered to trick machine vision can influence humans too - Google DeepMind

> 来源：[https://deepmind.google/discover/blog/images-altered-to-trick-machine-vision-can-influence-humans-too/](https://deepmind.google/discover/blog/images-altered-to-trick-machine-vision-can-influence-humans-too/)

Research

# Images altered to trick machine vision can influence humans too

Published

2 January 2024

Authors

Gamaleldin Elsayed and Michael Mozer

New research shows that even subtle changes to digital images, designed to confuse computer vision systems, can also affect human perception

Computers and humans see the world in different ways. Our biological systems and the artificial ones in machines may not always pay attention to the same visual signals. Neural networks trained to classify images can be completely misled by subtle perturbations to an image that a human wouldn’t even notice.

That AI systems can be tricked by such adversarial images may point to a fundamental difference between human and machine perception, but it drove us to explore whether humans, too, might—under controlled testing conditions—reveal sensitivity to the same perturbations. In a series of experiments published in Nature Communications, we found evidence that human judgments are indeed systematically influenced by adversarial perturbations.

Our discovery highlights a similarity between human and machine vision, but also demonstrates the need for further research to understand the influence adversarial images have on people, as well as AI systems.

## What is an adversarial image?

An adversarial image is one that has been subtly altered by a procedure that causes an AI model to confidently misclassify the image contents. This intentional deception is known as an adversarial attack. Attacks can be targeted to cause an AI model to classify a vase as a cat, for example, or they may be designed to make the model see anything except a vase.

Left: An Artificial Neural Network (ANN) correctly classifies the image as a vase but when perturbed by a seemingly random pattern across the entire picture (middle), with the intensity magnified for illustrative purposes – the resulting image (right) is incorrectly, and confidently, misclassified as a cat.

And such attacks can be subtle. In a digital image, each individual pixel in an RGB image is on a 0-255 scale representing the intensity of individual pixels. An adversarial attack can be effective even if no pixel is modulated by more than 2 levels on that scale.

Adversarial attacks on physical objects in the real world can also succeed, such as causing a stop sign to be misidentified as a speed limit sign. Indeed, security concerns have led researchers to investigate ways to resist adversarial attacks and mitigate their risks.

## How is human perception influenced by adversarial examples?

Previous research has shown that people may be sensitive to large-magnitude image perturbations that provide clear shape cues. However, less is understood about the effect of more nuanced adversarial attacks. Do people dismiss the perturbations in an image as innocuous, random image noise, or can it influence human perception?

To find out, we performed controlled behavioral experiments.To start with, we took a series of original images and carried out two adversarial attacks on each, to produce many pairs of perturbed images. In the animated example below, the original image is classified as a “vase” by a model. The two images perturbed through adversarial attacks on the original image are then misclassified by the model, with high confidence, as the adversarial targets “cat” and “truck”, respectively.

Next, we showed human participants the pair of pictures and asked a targeted question: “Which image is more cat-like?” While neither image looks anything like a cat, they were obliged to make a choice and typically reported feeling that they were making an arbitrary choice. If brain activations are insensitive to subtle adversarial attacks, we would expect people to choose each picture 50% of the time on average. However, we found that the choice rate—which we refer to as the perceptual bias—was reliably above chance for a wide variety of perturbed picture pairs, even when no pixel was adjusted by more than 2 levels on that 0-255 scale.

From a participant’s perspective, it feels like they are being asked to distinguish between two virtually identical images. Yet the scientific literature is replete with evidence that people leverage weak perceptual signals in making choices, [signals that are too weak for them to express confidence or awareness](https://link.springer.com/article/10.3758/BF03206939) ). In our example, we may see a vase of flowers, but some activity in the brain informs us there’s a hint of cat about it.

Left: Examples of pairs of adversarial images. The top pair of images are subtly perturbed, at a maximum magnitude of 2 pixel levels, to cause a neural network to misclassify them as a “truck” and “cat”, respectively. A human volunteer is asked “Which is more cat-like?” The lower pair of images are more obviously manipulated, at a maximum magnitude of 16 pixel levels, to be misclassified as “chair” and “sheep”. The question this time is “Which is more sheep-like?”

We carried out a series of experiments that ruled out potential artifactual explanations of the phenomenon for our Nature Communications paper. In each experiment, participants reliably selected the adversarial image corresponding to the targeted question more than half the time. While human vision is not as susceptible to adversarial perturbations as is machine vision (machines no longer identify the original image class, but people still see it clearly), our work shows that these perturbations can nevertheless bias humans towards the decisions made by machines.

## The importance of AI safety and security research

Our primary finding that human perception can be affected—albeit subtly—by adversarial images raises critical questions for AI safety and security research, but by using formal experiments to explore the similarities and differences in the behaviour of AI visual systems and human perception, we can leverage insights to build safer AI systems.

For example, our findings can inform future research seeking to improve the robustness of computer vision models by better aligning them with human visual representations. Measuring human susceptibility to adversarial perturbations could help judge that alignment for a variety of computer vision architectures.

Our work also demonstrates the need for further research into understanding the broader effects of technologies not only on machines, but also on humans. This in turn highlights the continuing importance of cognitive science and neuroscience to better understand AI systems and their potential impacts as we focus on building safer, more secure systems.