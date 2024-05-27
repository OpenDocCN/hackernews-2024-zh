<!--yml
category: 未分类
date: 2024-05-27 12:58:06
-->

# Algebraic Geometry for Computer Graphics

> 来源：[https://courses.cs.washington.edu/courses/cse590b/13au/](https://courses.cs.washington.edu/courses/cse590b/13au/)

# Algebraic Geometry for Computer Graphics

 James F. Blinn  Computer graphicists model shapes with polynomials. Simple linear polynomials generate flat polygons; higher order polynomials generate curved lines and surfaces. In order to develop algorithms for modeling and rendering such shapes we must have an intimate understanding of how the coefficients of the polynomials relate to the geometric shapes they generate. The answers to various geometric questions are expressed in terms of new polynomials built out of the coefficients of the original polynomial. The problem is that, for fairly simple shapes, these new polynomials can get incredibly complicated. For example, the condition that a cubic curve has a double point results in a test expression that is a 12^(th)  order polynomial in the cubic’s coefficients, and this test expression has over 10000 terms! Fortunately there is a better way to deal with this problem. The research I am doing builds on a graphical notation technique that was originally proposed by Sylvester and Clifford over 100 years ago. I have been translating many of the results of classical algebraic geometry into this notation. This has led to a much better visualization of the relation between a polynomial’s shape and its coefficients and has produced new algorithms for solving and factoring polynomials. My ultimate goal is a new catalog of all the interesting algebraic relations between polynomial coefficients and their shape, describing curves and surfaces of orders up to 4 or 5 in a way that their processing can be accurately computed by parallel processors such as modern GPU’s.

Algebraic Geometry as studied by Mathematicians often deals with very abstract and general issues that I admit that I don’t completely understand myself. Instead, my talks will focus on that subset of Algebraic Geometry that I find most relevant to computer graphics. They will only require a knowledge of simple linear algebra and homogeneous coordinates. I am still researching some of the topics so the presentation will show open questions as well as answers.

Click [here](topics.html) for a list of topics covered in the seminar.

## Lecture 8 – Quartics and Groups

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_8.pdf).

## Lecture 7 – Cubic Curves

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_7.pdf).

## Lecture 6 – Cubic Polynomials

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_6.pdf).

**Reference:**
[How To Solve a Cubic Equation - Part 2 the 11bar case.](lecture_notes/solvecubic_p2.pdf)
[How To Solve a Cubic Equation - Part 3 General Depression and a new Covariant.](lecture_notes/solvecubic_p3.pdf)
[How To Solve a Cubic Equation - Part 4 The 111 case.](lecture_notes/solvecubic_p4.pdf)
[How to solve a Cubic Part 5 - Back to Numerics.](lecture_notes/solvecubic_p5.pdf)

## Lecture 5 – Cataloging Bezier Curves

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_5.pdf).

In this, the last lecture for this term I will try something a bit different. So far I have been working “bottom up” by building up elementary structures of more and more complexity. This time I will start “top down” with a more complex goal: Determining when and where Bezier curves have loops, and work toward the tensor diagram structures that answer this question. People who have missed a few lectures should still be able to get something useful from this talk. I will finish by teasing some of the more advanced things we might get into if there is interest in continuing this next term.

**Reference:** [How To Solve a Cubic Equation - Part 1 The Shape of the Discriminant.](lecture_notes/cubic.pdf).

## Lecture 4 –Transforming transformations

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_4.pdf).

November 20, 2013 at 3:30pm in room 045.

See how Nilpotent, Idempotent, and Involution matrices relate in colorful 3D plots, and see how to represent all these with tensor diagrams in a way that makes their properties apparent. We will then look at a new way to generate Exemplary Transformations and have an unconventional look at cross products.

## Lecture 3: Quadratics - Resultants and Factoring, The world of transformation matrices

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_3.pdf).

November 6, 2013 at 3:30pm in room 045\.

In this talk we will finish up the discussion of quadratic polynomials in P1 by showing how tensor diagrams can represent some basic operations like resultants, functional determinants, syzygies, and polynomial division. We can then find our missing invariant that allows us to completely categorize the space of equivalence classes. And we will see an amusing relation between 2x2 matrices in P1 and 3D vectors in P2\.

Then we will launch into the world of transformation matrices. See how Nilpotent, Idempotent, and Involution matrices relate in colorful 3D plots. And see how to represent all these with tensor diagrams in a way that makes their properties apparent.

Our next topics will be Exemplary Transformations and an unconventional look at cross products, but I don’t expect we will get to these this time.

## Lecture 2: Projective 1-space

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_2a.pdf).

October 23, 2013 at 3:30pm in room 045. 

In this talk we will go down a dimension and look at objects in projective 1-space, P¹. First I will show the (simpler) form that tensor diagrams take in this space and show how they generate some basic identities. In P¹, 2x2 matrices are used to represent quadratic polynomials and to represent linear transforms. The different transformation properties of these matrices lead to different internal structures. I will show various visualizations of these properties and how they lead to some exercises in projective thinking. These will form the foundations of generalization to higher dimensions and higher order polynomials.  

## Lecture 1: Algebraic Geometry A Personal View

**See the slides from the lecture** [**here**](lecture_notes/UW_CSE590B_Lecture_1.pdf).

### Publications referenced in this lecture

#### Key Historical Papers

J. J. Sylvester,
On an application of the new atomic theory to the graphical representation of the invariants and covariants of binary quantics, with three appendices
Amer. J. Math 1 (1878), 64-125.

W. Clifford,
Extract of a letter to Mr. Sylvester from Prof. Clifford of University College, London
Amer. J. Math. 1 (1878), 126-218.

A. B. Kempe,
On the application of Clifford’s graphs to ordinary binary quantics,
Proc. London Math. Soc., 17 (1878)107-121.

#### My introduction to diagrams

Geoffrey E. Stedman,
Diagram Techniques in Group Theory,
Cambridge University Press, 1990.

#### Best summary of historical notational techniques

Peter Olver & Chehrzad Shakiban,
Graph Theory and Classical Invariant Theory,
Advances in Mathematics 75, 212-245 (1989)
[http://www.math.umn.edu/~olver/a_/gtcit.pdf](http://www.math.umn.edu/~olver/a_/gtcit.pdf)
[http://www.ima.umn.edu/2006-2007/seminars/shakiban/Invarant2.ppt](http://www.ima.umn.edu/2006-2007/seminars/shakiban/Invarant2.ppt)

#### Also about the notation I use

Jurgen Richter-Gebert
Perspectives on Projective Geometry
Springer 2011
Chapters 13,14

#### Nice example of pictures but on a subject I will not be covering

Ralph H. Abraham & Christopher D. Shaw,
Dynamics- The Geometry of Behaviour,
Aerial Press, Inc. 1984.

#### A book I don’t understand yet, but hope to someday

Robin Hartshorne,
Algebraic Geometry,
Springer 1977\.  Jim Blinn made his first computer generated pictures in 1968 while an undergraduate at the University of Michigan. From 1974 to 1977 he was a graduate student at the University of Utah where he did research in realistic rendering. The results of this research have become standard techniques in today’s computer animation systems. They include realistic specular lighting models, bump mapping and environment/reflection mapping. In 1977 he received a Ph.D. and moved to the Jet Propulsion Laboratory where he produced computer graphics animations for various space missions to Jupiter, Saturn and Uranus. These animations were shown on many news broadcasts as part of the press coverage of the missions and were the first exposure to computer animation for many people in the industry today. Also at JPL he produced animation for the PBS series COSMOS and for the Annenberg/CPB funded project “The Mechanical Universe”, a 52 part telecourse to teach college level physics. During these productions he developed several other standard computer graphics techniques including work in cloud simulation and a modeling technique variously called blobbies or metaballs. From 1989 to 1995 he worked at Caltech producing animations to teach High School mathematics for “Project Mathematics!” From 1987 to 2007 he had a regular column called *Jim Blinn’s Corner* in the IEEE Computer Graphics and Applications journal where he described mathematical techniques used in computer graphics. These have been collected into three books. From 1995 to 2009 he worked at Microsoft Research as a Graphics Fellow developing a new mathematical notation scheme that greatly simplifies the algebraic description and manipulation of curves and surfaces. He is currently retired