<!--yml
category: 未分类
date: 2024-05-27 14:32:35
-->

# Inverting PhotoDNA

> 来源：[https://anishathalye.com/inverting-photodna/](https://anishathalye.com/inverting-photodna/)

Microsoft [PhotoDNA](https://www.microsoft.com/en-us/photodna) creates a “unique digital signature” of an image which can be matched against a database containing signatures of previously identified illegal images like CSAM. The technology is [used](https://www.makeuseof.com/what-is-photodna-how-does-it-work/) by companies including Google, Facebook, and Twitter. Microsoft says:

> A PhotoDNA hash is not reversible, and therefore cannot be used to recreate an image.

[Ribosome](https://github.com/anishathalye/ribosome) inverts PhotoDNA hashes using machine learning.

This demonstration uses provocative images to make a point: rough body shapes and faces can be recovered from the PhotoDNA hash. The image in the top row is from Sports Illustrated, and the image in the bottom row is [not a real person](https://thispersondoesnotexist.com/). The first column shows a portion of the 144-byte PhotoDNA hash, and the center column shows the image that can be reconstructed from this hash using Ribosome.

Like any other lossy function, the PhotoDNA hash is not perfectly invertible, but the hash leaks plenty of information about the original input, as evidenced by these image recreations.

Neither details of the PhotoDNA algorithm nor an implementation is officially available to the public, but the algorithm has been [reverse-engineered](https://hackerfactor.com/blog/index.php?/archives/931-PhotoDNA-and-Limitations.html) based on public documents, and a [compiled library](https://github.com/jankais3r/pyPhotoDNA) for computing PhotoDNA hashes has been leaked around the time [collisions](https://github.com/anishathalye/neural-hash-collider) were found in Apple’s NeuralHash algorithm.

Likely due to the closed-source nature of PhotoDNA, there has not been much work studying the hash function. In 2019, [Nadeem et al.](https://kar.kent.ac.uk/77165/) in collaboration with Microsoft investigated the privacy protection capability of PhotoDNA by testing it against ML classification. The paper claimed that “PhotoDNA is resistant to machine-learning-based classification attacks”. More recently, in November 2021, [Prokos et al.](https://eprint.iacr.org/2021/1531) performed targeted second-preimage attacks on PhotoDNA.

Ribosome is an inversion attack on the PhotoDNA hash function and investigates the claim that a PhotoDNA hash “cannot be used to recreate an image.”

Ribosome treats PhotoDNA as a black box and attacks the hash function using machine learning. Because an implementation of PhotoDNA has been [leaked](https://github.com/jankais3r/pyPhotoDNA), it is possible to produce a dataset of image/hash pairs. Ribosome trains a neural net on such a dataset to learn to synthesize an image given its hash.

PhotoDNA hashes are 144-element vectors of bytes. Ribosome uses a neural network similar to the [DCGAN](https://arxiv.org/pdf/1511.06434/) generator and the [Fast Style Transfer](https://cs.stanford.edu/people/jcjohns/papers/fast-style/fast-style-supp.pdf) network, using residual blocks followed by fractionally-strided convolutions for learned upscaling, to turn this into a 100x100 image.

The choice of dataset used to train the model affects the results. Ideally, the images in the dataset should be drawn from the same distribution as the images whose hashes are being inverted. For example, when inverting a hash of an image that is expected to contain a person, training the model on the [Places](http://places.csail.mit.edu/) dataset may not produce optimal results.

Ribosome can produce good results even when the exact distribution that the image is drawn from is not known. The following figure illustrates hash inversions computed by models trained on different test sets:

In the above figure, held-out test images from each of the datasets, plus an extra image from the internet, are hashed and then reproduced using Ribosome models trained on different datasets. Training datasets include [CelebA](https://mmlab.ie.cuhk.edu.hk/projects/CelebA.html), [COCO](https://cocodataset.org/), and a dataset of 100K images scraped from SFW and NSFW subreddits. A fourth dataset was created by combining images from the three datasets (taking only 40K images from CelebA).

The figure illustrates some interesting phenomena:

*   Using a large and diverse dataset allows the model to produce good reproductions from a PhotoDNA hash (fourth column)
*   The better the prior, the better the result, e.g. reproducing a face with a model trained on CelebA (first column, second row)
*   Ribosome can handle some distribution shift, e.g. reproducing a face when trained on COCO (second column, second row)
*   Unsurprisingly, training the model on a narrowly-scoped dataset gives it a strong bias towards producing images like those in that dataset, e.g. training on CelebA biases the model to produce images containing faces (first column)
*   The model learns to recolor the image (PhotoDNA converts the image to greyscale before processing it)
*   The model sometimes has amusing failure behaviors, e.g. the Reddit dataset was not carefully curated and had many “image unavailable” images, and artifacts from this are visible in some of the results (third column)

The images above are manually selected examples, chosen for the quality of the result, diversity of images (e.g. different poses), and being SFW. The reconstruction is not always perfect. To give a sense of how the model works on average, here are 5 randomly-selected test datapoints from COCO shown along with their original images:

Ribosome shows that PhotoDNA does not perfectly hide information about the source image used to compute the signature, and that in fact, a PhotoDNA hash can be used to produce thumbnail-quality reproductions of the original image.

Code and pre-trained models are available on [GitHub](https://github.com/anishathalye/ribosome).