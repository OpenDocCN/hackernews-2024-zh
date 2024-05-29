<!--yml
category: 未分类
date: 2024-05-29 12:28:57
-->

# The Modern Algorithmic Toolbox (CS168), Spring 2024

> 来源：[https://web.stanford.edu/class/cs168/index.html](https://web.stanford.edu/class/cs168/index.html)

# CS 168: The Modern Algorithmic Toolbox, Spring 2024

## **Announcements:**

*   5/28: [Mini-project #9](p9.pdf) is available. Here are the [close.csv](close.csv) and [tickers.csv](tickers.csv) data files. The project is due Thursday, June 6th at 11am.
*   5/23: Here is a [practice final exam](practice_final.pdf).
*   5/22: [Mini-project #8](p8.pdf) is available. Here is the [laurel_yanny.wav](laurel_yanny.wav) audio clip for Part 3, and a zipped folder [worldsYikes.zip](worldsYikes.zip) containing examples that we created illustrating that the 10-point research-oriented bonus part is not impossible : ) Also feel free to see the [research paper](https://theory.stanford.edu/~valiant/polyperceivable/index.html) that grew out of the work of two former CS68 students on this bonus part. The miniproject is due Thursday, May 30th at 11am.
*   5/14: [Mini-project #7](p7.pdf) is available. It is due Thursday, May 23rd at 11am. [Here](parks.csv) is the list of national parks, and [plot_route.py](plot_route.py) for Part 2\. For Part 3, the QWOP scripts are available for MATLAB [here](qwop.m) and Python [here](qwop.py).
*   5/8: [Mini-project #6](p6.pdf) is available. It is due Thursday, May 16th at 11am. [Here](cs168mp6.csv) is the data file for Part 2.
*   5/1: [Mini-project #5](p5.pdf) is available (this is one of my favorites!) It is due Thursday, May 9th at 11am. The data files you will need are: [dictionary.txt](dictionary.txt), [co_occur.csv](co_occur.csv), [analogy_task.txt](analogy_task.txt), and [p5_image.gif](p5_image.gif).
*   4/23: [Mini-project #4](p4.pdf) is available. The genomic dataset is available [here](p4dataset2024.txt), and the short file to decode the demographic tags is [here](p4dataset2024_decoding.txt). It is due Thursday, May 2nd (at 11am).
*   4/16: [Mini-project #3](p3.pdf) is available. It is due Thursday, April 25th (at 11am).
*   4/9: [Mini-project #2](p2.pdf) is available. The dataset is available [here](p2_data.zip). It is due Thursday, April 18th (at 11am).
*   Most CA office hours are now posted, though they might be slightly in flux over the next couple days. We may tweak the schedule in a few weeks after it is more clear which office hours are the most attended.
*   Here is [Mini-project LaTex solution template](sol_template.tex). Feel free to also use your own solution template, though in general the CAs would prefer you to not include the question prompts in your solutions, as it adds a lot of length/clutter.
*   Here is [Mini-project #1](p1.pdf). It is due Thursday, April 11th (at 11am). Please submit via Gradescope, entry code 2PN4J7. [Here](histogram.py) is some starter code for drawing histograms in Python, which might be helpful.
*   First lecture Tuesday, April 2nd, 1:30-2:50pm in NVIDIA Auditorium. See you there!
*   Lecture videos will be available via Canvas, though I strongly encourage you to attend lecture in person if possible---its more fun for all of us.
*   [**Here is a link to our Ed online discussion forum.**](https://edstem.org/us/join/ZYbyeA)

 **## Instructor:** 

*   [Gregory Valiant](http://theory.stanford.edu/~valiant/) (Office hours: Tues 3-4pm, Gates 162). Contact: email me at my last name at stanford.edu or via our course Ed page.

## **Course Assistants:**

*   Sidhant Bansal (Office hours: Mon 4:15-6:15pm, Huang Basement)
*   Luci Bresette (Office hours: Wed 10:30-11:30, Fri 10:30-11:30, Huang Basement)
*   Isaac Gorelik (Office hours: Mon 11-1, Huang Basement)
*   Meenal Gupta (Office hours: Wednesday 8-10pm, [Zoom link, entry password 856878](https://stanford.zoom.us/j/91570040226?pwd=b3hESUVKdVB6N01kNCtZTXYyTlFZQT09)))
*   Benson Kung (Office hours: Wed 11-1, [Zoom link](https://stanford.zoom.us/j/9649412299?pwd=eERaSGZnVU5NeVp4ZmVOSmZXZ3VHZz09))
*   Ali Malik (Office hours: Wed 6-8pm, Huang Basement)
*   Ligia Melo (Office hours: Mon 9-11am Huang Basement)
*   Joey Rivkin (Office hours: Tues 10:45-11:45 and Wed 10-11am, Huang Basement)
*   Luna Yang (Office hours: Wed 2-4pm, Huang Basement)

## **Lecture Time/location:**

Tues/Thurs, 1:30-2:50pm in NVIDIA Auditorium

[**Ed site for online discussion/questions.**](https://edstem.org/us/join/ZYbyeA) This link includes the access-code that you need the first time you sign up.

## **Prerequisites:**

CS107 and CS161, or permission from the instructor.

## Course Description

This course will provide a rigorous and hands-on introduction to the central ideas and algorithms that constitute the core of the modern algorithms toolkit. Emphasis will be on understanding the high-level theoretical intuitions and principles underlying the algorithms we discuss, as well as developing a concrete understanding of when and how to implement and apply the algorithms. The course will be structured as a sequence of one-week investigations; each week will introduce one algorithmic idea, and discuss the motivation, theoretical underpinning, and practical applications of that algorithmic idea. Each topic will be accompanied by a mini-project in which students will be guided through a practical application of the ideas of the week. Topics include modern techniques in hashing, dimension reduction, linear and convex programming, gradient descent and regression, sampling and estimation, compressive sensing, linear-algebraic techniques (principal components analysis, singular value decomposition, spectral techniques), and an intro to differential privacy.

## Proposed Lecture Schedule

*   ### Week 1: Modern Hashing

    *   **Lecture 1 (Tues 4/2):** Course introduction. ``Consistent'' hashing. Supplementary material:
    *   **Lecture 2 (Thurs 4/4):** Property-preserving lossy compression. From majority elements to approximate heavy hitters. From bloom filters to the count-min sketch. Supplementary material:
*   ### Week 2: Data with Distances (Similarity Search, Nearest Neighbor, Dimension Reduction, LSH)

    *   **Lecture 3 (Tues 4/9):** Similarity Search. (Dis)similarity metrics: Jaccard, Euclidean, Lp. Efficient algorithm for finding similar elements in small/medium (ie. <20) dimensions using k-d-trees. Supplementary material:
    *   **Lecture 4 (Thurs 4/11):** Curse of Dimensionality, kissing number. Distance-preserving compression. Estimating Jaccard similarity using MinHash. Euclidean distance preserving dimensionality reduction (aka the Johnson-Lindenstrauss Transform). Supplementary material:
        *   A nice survey of "kissing number", and some other strange phenomena from high dimensional spaces: [Kissing Numbers, Sphere Packings, and some Unexpected Proofs](http://www.ams.org/notices/200408/fea-pfender.pdf) (from 2000).
        *   Origins of MinHash at Alta Vista: Broder, [Identifying and Filtering Near-Duplicate Documents](http://cs.brown.edu/courses/cs253/papers/nearduplicate.pdf) (from 2000).
        *   Ailon/Chazelle, [Faster Dimension Reduction](https://www.cs.princeton.edu/~chazelle/pubs/fasterdim-ac10.pdf), CACM '10.
        *   Andoni/Indyk, [Near-Optimal Hashing Algorithms for Approximate Nearest Neighbor in High Dimensions](http://mags.acm.org/communications/200801/#pg119), CACM '08.
        *   For much more on Locality Sensitive Hashing, see [this chapter](http://infolab.stanford.edu/~ullman/mmds/ch3.pdf) of the CS246 textbook (by Leskovec, Rajaraman, and Ullman).
*   ### Week 3: Generalization and Regularization

    *   **Lecture 5 (Tues 4/16):** Generalization (or, how much data is enough?). Learning an unknown function from samples from an unknown distribution. Training error vs. test error. PAC guarantees for linear classifiers. Empirical risk minimization.
    *   **Lecture 6 (Thurs 4/18):** Regularization. The polynomial embedding and random projection, L2 regularization, and L1 regularization as a computationally tractable surrogate for L0 regularization, frequentist and Bayesian perspectives on regularization. Supplementary material:
        *   A 2016 paper arguing that, to understand why deep learning works, we need to rethink the theory of generalization. This paper was quite controversial, with one camp thinking that its conclusions are completely obvious, and the other camp thinking that it is revealing an extremely deep mystery. You decide for yourself! Paper is [here](https://arxiv.org/pdf/1611.03530.pdf).
*   ### Week 4: Linear-Algebraic Techniques: Understanding Principal Components Analysis

    *   **Lecture 7 (Tues 4/23):** Understanding Principal Component Analysis (PCA). Minimizing squared distances equals maximizing variance. Use cases for data visualization and data compression. Failure modes for PCA. Supplementary material:
        *   An exposition by 23andMe of the fact that the top 2 principal components of genetic SNP data of Europeans essentially recovers the geography of europe: [nice exposition w. figures](http://blog.23andme.com/news/a-different-kind-of-gene-mapping-comparing-genetic-and-geographic-structure-in-europe-the-return/). Original Nature paper: [Genes mirror geography in Europe](http://www.nature.com/nature/journal/v456/n7218/abs/nature07331.html), Nature, Aug. 2008.
        *   [Eigenfaces](http://www.cs.ucsb.edu/~mturk/Papers/jcn.pdf) (see also this [blog post](http://jeremykun.com/2011/07/27/eigenfaces/))
        *   There's tons of PCA tutorials floating around the Web (some good, some not so good), which you are also permitted to refer to.
    *   **Lecture 8 (Thurs 4/25):** How PCA works. Maximizing variance as finding the "direction of maximum stretch" of the covariance matrix. The simple geometry of "diagonals in disguise." The power iteration algorithm.
*   ### Week 5: Linear-Algebraic Techniques: Understanding the Singular Value Decomposition and Tensor Methods

    *   **Lecture 9 (Tues 4/30):** Low-rank matrix approximations. The singular value decomposition (SVD), applications to matrix compression, de-noising, and matrix completion (i.e. recovering missing entries). Supplementary material:
        *   For a striking recent application of low-rank approximation, check out the [LoRA paper](https://arxiv.org/abs/2106.09685) from 2021\. The punchline is that in many cases, when fine-tuning large language models, the updates are close to low rank. Hence once can explicitly train these fine-tuning updates to the original model in the low-rank factorized parameter space, reducing the number of parameters that need to be trained by 1000x or 10,000x!!
    *   **Lecture 10 (Thurs 5/2):** Tensor methods. Differences between matrices and tensors, the uniqueness of low-rank tensor factorizations, and Jenrich's algorithm. Supplementary material:
        *   A blog post discussing Spearman's original experiment and the motivation for tensor methods: [here](http://www.offconvex.org/2015/12/17/tensor-decompositions/).
        *   See chapter 3 of Ankur Moitra's course notes [here](http://people.csail.mit.edu/moitra/docs/bookex.pdf) for a more technical in depth discussion of tensor methods, and Jenrich's algorithm.
*   ### Week 6: Spectral Graph Theory

    *   **Lecture 11 (Tues 5/7):** Graphs as matrices and the Laplacian of a graph. Interpretations of the largest and smallest eigenvectors/eigenvalues of the Laplacian. Spectral embeddings, and an overview of applications (e.g. graph coloring, spectral clustering.) Supplementary material:
        *   Dan Spielman's [excellent lecture notes](http://www.cs.yale.edu/homes/spielman/561/) for his semester-long course on Spectral Graph Theory. The notes include a number of helpful plots.
        *   Amin Saberi offered a grad seminar a few years ago that covered some of the research frontier of Spectral Graph Theory. Hopefully he will offer this again soon...
    *   **Lecture 12 (Thurs 5/9):** Spectral techniques, Part 2\. Interpretations of the second eigenvalue (via conductance and isoperimetric number), and connections with the speed at which random walks/diffusions converge.
*   ### Week 7: Sampling and Estimation

    *   **Lecture 13 (Tues 5/14):** Reservoir sampling (how to select a random sample from a datastream). Basic probability tools: Markov's inequality and Chebyshev's inequality. Importance Sampling (how to make inferences about one distribution based on samples from a different distribution). Good-Turing estimate of the missing/unseen mass. Supplementary material:
        *   A paper of mine showing that one can accurately estimate the structure of the unobserved portion of a distribution---not just the total probability mass. [paper link here](https://dl.acm.org/citation.cfm?id=3125643).
    *   **Lecture 14 (Thurs 5/16):** Markov Chains, stationary distributions. Markov Chain Monte Carlo (MCMC) as approaches to solving hard problems by sampling from carefully crafted distributions. Supplementary material:
        *   A basic description of MCMC, [here](http://jeremykun.com/2015/04/06/markov-chain-monte-carlo-without-all-the-bullshit/).
        *   Lecture notes from Persi Diaconis on MCMC, including a description of the MCMC approach to decoding substitution-ciphers, [here](https://www.researchgate.net/publication/215446278_The_Markov_Chain_Monte_Carlo_Revolution).
        *   Example of MCMC used for fitting extremely complex biological models: [The human splicing code...](http://www.sciencemag.org/content/347/6218/1254806.full) Science, Jan. 2015.
        *   For those interested in computer Go: [here](http://www.nature.com/nature/journal/v529/n7587/full/nature16961.html) is the Jan, 2016 Nature paper from Google's DeepMind group.
*   ### Week 8: The Fourier Perspective (and other bases)

    *   **Lecture 15 (Tues 5/21):** Fourier methods, part 1\. Supplementary material:
        *   A very basic intro to Fourier transformations with some nice visualizations: [here](http://betterexplained.com/articles/an-interactive-guide-to-the-fourier-transform/).
        *   A book version of a Stanford course on the Fourier Transform, which has many extremely nice applications: [pdf here](http://see.stanford.edu/materials/lsoftaee261/book-fall-07.pdf).
    *   **Lecture 16 (Thurs 5/23):** Fourier methods, part 2\. Emphasis on convolution and its many applications (fast multiplication, physics simulations, etc.)
        *   Lecture [notes](l/l15.pdf) combined with Lecture 15.
*   ### Week 9: Online Learning with Multiplicative Weights, and Convex Optimization

    *   **Lecture 17 (Tues 5/28):** Online learning and the multiplicative weights algorithm, and applications. .

    *   **Lecture 18 (Thurs 5/30):** Linear and convex programming, and solving LPs via the multiplicative weights algorithm.
*   ### Week 10: Privacy Preserving Computation

    *   **Lecture 19 (Tues 6/4):** Differential Privacy in data analysis and machine learning.

## Coursework

*   **Assignments (70% of Grade)**: There will be 9 weekly mini-projects centered around the topics covered that week. Each mini-project contains both written and programming parts. You are strongly encouraged to work in teams of up to four students. If you work in a team **only one member** should submit all of the relevant files.

    For the written part, you are encouraged to use LaTeX to typeset your homeworks; we've provided a [template](homework.tex) for your convenience. We will be using the [GradeScope](https://gradescope.com/) online submission system. Please create an account on Gradescope using your Stanford ID and join CS168 using entry code 2PN4J7\.

    For the programming part, you are encouraged to use Numpy and Pyplot in Python ([Python tutorial](https://docs.python.org/3/tutorial/index.html), [Numpy tutorial](http://www.numpy.org/), [Matplotlib tutorial](https://matplotlib.org/stable/)), matlab ([tutorial](http://www.cyclismo.org/tutorial/matlab/)), or some other scientific computing tool (with plotting). [Here](https://colab.research.google.com/github/cs231n/cs231n.github.io/blob/master/python-colab.ipynb) is a comprehensive python tutorial using Google's colab that was put together for the CS231n course, with examples of plotting using matplotlib at the very end.

    Assignments will be released on Tuesdays, and are due at 11am on Thursday the following week. **No late assignments will be accepted**, but we will drop your lowest assignment grade when calculating your final grade.

*   **Final Exam (30% of Grade)**: There will be an in-class final exam, covering all the material with an emphasis on the high-level punchlines of each lecture. We will post a practice exam a few weeks before the end of the quarter.

## Collaboration Policy

You can discuss the problems at a high level with other groups and contact the course staff (via Ed or office hours) for additional help. And of course, you are encouraged to help respond to Ed questions .

You may refer to the course notes and research papers linked from the course webpage, and can also use other references that you find online. You may *NOT* consult solution sets or code from past offerings of this course. Of course, you are expected to understand everything that is written on any assignment turned in with your name; if you do refer to references, make sure you cite them appropriately, and make sure that all your words are your own.

You are also permitted to use general resources for whatever programming language you choose to use. None of the assignments require you to write much code, and *you may NOT reuse any code from friends/internet sources. If you use helper functions or code snippets that you did not write yourself (aside from standard python packages, etc.) you must clearly cite the source in your writeup.

Please follow the [honor code](https://communitystandards.stanford.edu/policies-and-guidance/honor-code).