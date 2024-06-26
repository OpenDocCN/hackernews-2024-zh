<!--yml

分类：未分类

日期：2024-05-27 14:50:13

-->

# 神经网络训练生成美丽的分形 | 贾斯查的博客

> 来源：[https://sohl-dickstein.github.io/2024/02/12/fractal.html](https://sohl-dickstein.github.io/2024/02/12/fractal.html)

```

My five year old daughter came home from kindergarten a few months ago, and told my partner and I that math was stupid (!). We have since been working (so far successfully) to make her more excited about all things math, and more proud of her math accomplishments. One success we've had is that she is now very interested in fractals in general, and in particular enjoys watching deep zoom videos into [Mandelbrot](https://youtu.be/8cgp2WNNKmQ?si=PD7W2q4qDNY9AgzD) and [Mandelbulb](https://youtu.be/BLmAV6O_ea0?si=4iyAFMgzde0mTmsq) fractal sets, and eating [romanesco broccoli](https://en.wikipedia.org/wiki/Romanesco_broccoli). 
My daughter's interest has made me think a lot about fractals, and about the ways in which fractals relate to a passion of mine, which is artificial neural networks. 

I've realized that there are similarities between the way in which many fractals are generated, and the way in which we train neural networks. 
Both involve repeatedly applying a function to its own output. 
In both cases, that function has hyperparameters that control its behavior. 
In both cases the repeated function application can produce outputs that either diverge to infinity or remain happily bounded depending on those hyperparameters. 
Fractals are often defined by the boundary between hyperparameters where function iteration diverges or remains bounded.

Motivated by these similarities, I looked for fractal structure in the hyperparameter landscapes of neural network training. And I found it! The boundary between hyperparameters for which neural network training succeeds or fails has (gorgeous, organic) fractal structure. Details, and beautiful videos, below.

For a more technical presentation, see the short paper [*The boundary of neural network trainability is fractal*](https://arxiv.org/abs/2402.06184).

# Neural network training and hyperparameters

In order to train an artificial neural network, we iteratively update its parameters to make it perform better. We often do this by performing [gradient descent](https://en.wikipedia.org/wiki/Gradient_descent) steps on a loss function. The loss function is a measure of the neural network's performance. By descending the loss by gradient descent, we find values of the parameters for which the neural network performs well.

Training depends on *hyperparameters*, which specify details about how parameter update steps should be performed and how the network should be initialized. For instance, one common hyperparameter is the learning rate, which sets the magnitude of the update we make to the model’s parameters at every training step.

If the learning rate is too large, then the parameter update steps are too large. This causes the parameters to diverge (grow towards infinity) during training, and as a result causes the training loss to become very bad. If the learning rate is too small, the training steps are too short, and it takes a very large number of training steps to train the neural network. Requiring a very large number of training steps makes training slow and expensive. In practice, we often want to make the learning rate as large as possible, without making it so large that the parameters diverge.

# Visualizing the hyperparameter landscape

We can visualize how adjusting hyperparameters (like the learning rate) affects how quickly a neural network either trains or diverges. In the following image, each pixel corresponds to training the same neural network from the same initialization on the same data -- but with *different hyperparameters*. Blue-green colors mean that training *converged* for those hyperparameters, and the network successfully trained. Red-yellow colors mean that training *diverged* for those hyperparameters. The paler the color the faster the convergence or divergence

The neural network I used in this experiment is small and simple; it consists of an input layer, a $\operatorname{tanh}$ nonlinearity, and an output layer[^netdetails]. In the image, the x-coordinate changes the learning rate for the input layer’s parameters, and the y-coordinate changes the learning rate for the output layer’s parameters.

![Figure [p_ml]: **Hyperparameter landscape: A visualization of how neural network training success depends on learning rate hyperparameters.** Each pixel corresponds to a training run with the specified input and output layer learning rates. Training runs shown in blue-green converged, while training runs shown in red-yellow diverged.[^saturation] Hyperparameters leading to the best performance (lightest blue-green) are typically very close to hyperparameters for which training diverges, so the boundary region is of particular interest.](/assets/fractal/zoom_sequence_width-16_depth-2_datasetparamratio-1.0_minibatch-None_nonlinearity-tanh_phasespace-lr_vs_lr_step-0.png width="444px" border="1")

The best performing hyperparameters -- those that are shown with the palest blue-green shade, and for which the neural network trains the most quickly -- are near the boundary between hyperparameters for which training converges and for which it diverges. This is a general property. The best hyperparameters for neural network training are usually very near the edge of stability. For instance, as suggested above, the best learning rate in a grid search is typically the largest learning rate for which training converges rather than diverges.

# The boundary of neural network trainability is fractal

Because it is where we find the best hyperparameters, the boundary between hyperparameters that lead to converging or diverging training is of particular interest to us. Let’s take a closer look at it. Play the following video (I recommend playing it full screen, and increasing the playback resolution):

As we zoom into the boundary between hyperparameter configurations where training succeeds (blue) and fails (red), we find intricate structure at every scale. The boundary of neural network trainability is fractal! 
🤯 (If you watched the video to the end, you saw it turn blocky in the last frames. During network training I used the $\operatorname{float64}$ numeric type, which stores numbers with around 16 decimal digits of precision. The blockiness is what happens when we zoom in so far that we need more than 16 digits of precision to tell pixels apart.)

This behavior is general. We see fractals if we change the data, change the architecture, or change the hyperparameters we look at. The fractals look qualitatively different for different choices though. Network and training design decisions also have artistic consequences! 

![Figure [paper]: **Neural network training produces fractals in all of the experimental configurations I tried.** The figure is taken from the [companion paper](https://arxiv.org/abs/2402.06184), and shows a region of the fractal resulting from each experimental condition. Experimental conditions changed the nonlinearity in the network, changed the dataset size, changed between minibatch and full batch training, and changed the hyperparameters we look at.](/assets/fractal/fractal_tiles_midres.png width="444px" border="1")

Here are the remaining fractal zoom videos for the diverse configurations summarized in Figure [paper]. You can find code for these experiments in [this colab](https://colab.research.google.com/github/Sohl-Dickstein/fractal/blob/main/the_boundary_of_neural_network_trainability_is_fractal.ipynb)[^beware].

- **Changing the activation function to the identity function:** i.e. the network is a deep linear network, with no nonlinearity.

- **Change the activation function to $\operatorname{ReLU}$:** This is a neat fractal, since the piecewise linear structure of the $\operatorname{ReLU}$ is visually apparent in the straight lines dividing regions of the fractal.

- **Train with a dataset size of 1:** i.e. only train on a single datapoint. Other experiments have a number of training datapoints which is the same as the free parameter count of the model.

- **Train with a minibatch size of 16:** Other experiments use full batch training.

- **Look at different hyperparameters:** I add a hyperparameter which sets the mean value of the neural network weights at initialization. I visualize training success in terms of this weight initialization hyperparameter (*x-axis*) and a single learning rate hyperparameter (*y-axis*). Other experiments visualize training success in terms of learning rate hyperparameters for each layer. This fractal is **extra pretty** -- I like how it goes through cycles where what seems like noise is resolved to be structure at a higher resolution.

# This isn’t so strange after all

Now that I’ve shown you something surprising and beautiful, let me tell you why we should have expected it all along. In an academic paper I would put this section first, and tell the story as if I knew fractals would be there -- but of course I didn't know what I would find until I ran the experiment!

## Fractals result from repeated iteration of a function

One common way to make a fractal is to iterate a function repeatedly, and identify boundaries where the behavior of the iterated function changes. We can refer to these boundaries as bifurcation boundaries of the iterated function; the dynamics bifurcate at this boundary, in that function iteration leads to dramatically different sequences on either side of the boundary.

For instance, to generate the Mandelbrot set, we iterate the function $f( z; c ) = z^2 + c$ over and over again. The Mandelbrot fractal is the bifurcation boundary between the values of $c$ in the complex plane for which this iterated function diverges, and for which it remains bounded. The parameter $c$ is a (hyper)parameter of the function $f( z; c )$, similarly to how learning rates are hyperparameters for neural network training.

![Figure [mandelbrot fractal]: **The Mandelbrot fractal is generated by iterating a simple function, similar to the way in which update steps are iterated when training a neural network.** 
The image is color coded by whether iterations started at a point diverge (red-yellow colors) or remain bounded (blue-green colors). 
The boundary between the diverging and bounded regions is fractal. 
This image was generated by [this colab](https://colab.research.google.com/github/Sohl-Dickstein/fractal/blob/main/the_boundary_of_neural_network_trainability_is_fractal.ipynb).](/assets/fractal/mandelbrot_midres.png width="444px" border="1")

Other examples of fractals which are formed by bifurcation boundaries include [magnet fractals](https://paulbourke.net/fractals/magnet/), [Lyapunov fractals](https://en.wikipedia.org/wiki/Lyapunov_fractal), the [quadratic Julia set](https://mathworld.wolfram.com/JuliaSet.html), and the [Burning Ship fractal](Burning Ship fractal).

## Fractals can result from optimization

One particularly relevant class of bifurcation fractals are [Newton fractals](https://en.wikipedia.org/wiki/Newton_fractal). These are generated by iterating Newton's method to find the roots of a polynomial. 
[Newton's method is an optimization algorithm](https://en.wikipedia.org/wiki/Newton%27s_method_in_optimization). Newton fractals are thus a proof of principle that fractals can result from iterating steps of an optimization algorithm.

![Figure [newton fractal]: **Newton fractals, like the one shown, are formed by iterating Newton's method to find roots of a polynomial, and color coding initial conditions by the specific root the iterates converge to.** Newton fractals are a proof of principle that optimization can generate a fractal, since Newton's method is an optimization procedure. They motivate the idea of fractal behavior resulting from training (i.e. optimizing) a neural network.](/assets/fractal/Julia_set_for_the_rational_function.png width="444px" border="1")

## Artificial neural networks are trained by repeatedly iterating a function

When we train a neural network by iterating steps of gradient descent, we are iterating a fixed function, the same as for Mandelbrot, Newton, and other fractals. 
Like for Newton fractals, this fixed function corresponds to an optimization algorithm. 
Specifically, when we train a neural network using steepest gradient descent with a constant learning rate, we iterate the fixed function
$f(\theta; \eta ) = \theta( \eta ) - \eta\, g( \theta )$.
Here $\eta$ is the learning rate hyperparameter, $\theta$ are the parameters of the neural network, and $g( \theta )$ is the gradient of the loss function.

There are many differences between neural network training and traditional fractal generation. The fractals I just discussed all involve iterating a function of a single (complex valued) number. 
The equation defining the iterated function is short and simple, and takes less than a line of text to write down. 
On the other hand, neural network training iterates a function for all the parameters in the neural network. Some neural networks have trillions of parameters, which means the input and output of the iterated function is described with *trillions* of numbers, one for each parameter. The equation for a neural network training update is similarly far more complex than the function which is iterated for traditional fractals; it would require many lines, or possibly many pages, to write down the parameter update equations for a large neural network.

Nonetheless, training a neural network can be seen as a scaled up version of the type of iterative process that generates traditional fractals. 
We should not be surprised that it produces fractals in a similar way to simpler iterative processes.[^symmetry]

# Closing thoughts

## Meta-learning is hard

Meta-learning is a research area that I believe will transform AI over the next several years. In meta-learning we *learn* aspects of AI pipelines which are traditionally hand designed. For instance, we might meta-train functions to 
initialize, 
[optimize](https://github.com/google/learned_optimization/tree/main/learned_optimization/research/general_lopt), 
or regularize neural networks. If deep learning has taught us one thing, it's that with enough compute and data, trained neural networks can outperform and replace hand-designed heuristics; in meta-learning, we apply the same lesson to replace the hand-designed heuristics we use to train the neural networks themselves. 
Meta-learning is the reason I became interested in hyperparameter landscapes.

The fractal hyperparameter landscapes we saw above help us understand some of the challenges we face in meta-learning.
The process of meta-training usually involves optimizing hyperparameters (or meta-parameters) by gradient descent. 
The loss function we perform meta-gradient-descent on is called the meta-loss. 
The fractal landscapes we have been visualizing are also meta-loss landscapes; we are visualizing how well training succeeds (or fails) as we change hyperparameters. 
In practice, we often find the meta-loss atrocious to work with. It is often *chaotic* in the hyperparameters, which makes it [very difficult to descend](https://arxiv.org/abs/1810.10180)[^meta-descent]. 
Our results suggest a more nuanced and also more general perspective; meta-loss landscapes are chaotic because they are fractal. At every length scale, small changes in the hyperparameters can lead to large changes in training dynamics.

![Figure [meta landscape]: **Chaotic meta-loss landscapes make meta-learning challenging.** The image shows an example meta-loss landscape for a learned optimizer, with darker colors corresponding to better meta-loss. The two axes correspond to two of the meta-parameters of the learned optimizer (similar to the visualization in Figure [p_ml], where axes correspond to two hyperparameters). See [this paper](https://arxiv.org/abs/1810.10180) for details. This meta-loss landscape is difficult to meta-train on, since steepest gradient descent will become stuck in valleys or local minima, and because the gradients of the rapidly changing meta-loss function are exceptionally high variance.](/assets/fractal/meta-loss-landscape.png width="444px" border="1")

## Fractals are beautiful and relaxing

Recent AI projects I have collaborated on have felt freighted with historical significance. We are building tools that will change people's lives, and maybe bend the arc of history, for both [better and worse](/2023/09/10/diversity-ai-risk.html). This is incredibly exciting! But it is often also stressful.

This project on the other hand ... was just fun. I started the project because my daughter thought fractals were mesmerizing, and I think the final results are gorgeous. I hope you enjoy it in the same spirit!

-----

# Acknowledgements
Thank you to Maika Mars Miyakawa Sohl-Dickstein for inspiring the original idea, and for detailed feedback on the generated fractals.
Thank you to Asako Miyakawa for providing feedback on a draft of this post.

-----

[^netdetails]: In more detail, the baseline neural network architecture, design, and training configuration is as follows:

- Two layer fully connected neural network, with 16 units in the input and hidden layers, and with no bias parameters. The only parameters are the input layer weight matrix, and the output layer weight matrix.
- $\operatorname{tanh}$ nonlinearity in the single hidden layer
- Mean square error loss
- Fixed random training dataset, with number of datapoints the same as the number of free parameters in the network
- Full batch steepest descent training, with a constant learning rate
- **A different learning rate for each layer.** That is rather than training the input and output layer weight matrices with the same learning rate, each weight matrix has its own learning rate hyperparameter.

All experiments change one aspect of this configuration, except for the baseline experiment, which follows this configuration without change. If you want even more detail, see the [arXiv note](https://arxiv.org/abs/2402.06184) or the [colab notebook I used for all experiments](https://colab.research.google.com/github/Sohl-Dickstein/fractal/blob/main/the_boundary_of_neural_network_trainability_is_fractal.ipynb).

[^saturation]: The discerning reader may have noticed that training diverges when the output learning rate is made large, but that if the input learning rate is made large, performance worsens but nothing diverges. This is due to the $\operatorname{tanh}$ nonlinearity saturating. When the input learning rate is large, the input weights become large, the hidden layer pre-activations become large, and the $\operatorname{tanh}$ units saturate (their outputs grow very close to either -1 or 1). The output layer can still train on the (essentially frozen) $[-1, 1]$ activations from the first layer, and so some learning can still occur.

[^beware]: Like the fractals, the research code in the colab has vibes of layered organic complexity ... user beware!

[^symmetry]: Many fractals are generated by iterating simple functions, such as low order polynomials, or ratios of low order polynomials. Iterating these simple functions often generates simple symmetries, that are visually obvious when looking at the resulting fractals. The fractals resulting from neural networks are more organic, with fewer visually obvious symmetries. This is likely due to the higher complexity of the iterated functions themselves, as well as the many random parameters in the function definitions, stemming from the random initialization of the neural network and random training data.

[^meta-descent]: My collaborators and I have done more research into how to optimize a chaotic meta-loss. Especially see the papers: [*Unbiased Gradient Estimation in Unrolled Computation Graphs with Persistent Evolution Strategies*](https://icml.cc/virtual/2021/poster/10175), and [*Variance-Reduced Gradient Estimation via Noise-Reuse in Online Evolution Strategies*](https://openreview.net/forum?id=VhbV56AJNt).

```
