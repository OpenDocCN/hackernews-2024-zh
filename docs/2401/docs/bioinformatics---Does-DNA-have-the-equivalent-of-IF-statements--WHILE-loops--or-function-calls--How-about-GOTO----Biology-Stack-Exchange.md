<!--yml
category: 未分类
date: 2024-05-27 14:40:01
-->

# bioinformatics - Does DNA have the equivalent of IF-statements, WHILE loops, or function calls? How about GOTO? - Biology Stack Exchange

> 来源：[https://biology.stackexchange.com/questions/30116/does-dna-have-the-equivalent-of-if-statements-while-loops-or-function-calls-h](https://biology.stackexchange.com/questions/30116/does-dna-have-the-equivalent-of-if-statements-while-loops-or-function-calls-h)

Just to add to previous answers, but **transcriptional interference** (see e.g. [Shearwin et al., 2005](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC2941638/)) can be seen as a form of IF-statement (or WHILE) in the sense that:

```
if(x transcribed){not y transcribed} 
```

The interference does not have to be binary though, and more common are graded responses. Transcriptional interference can also take place at the RNA stage (see e.g. [Xue et al, 2014](http://www.nature.com/nature/journal/v514/n7524/full/nature13671.html)), using [*antisense RNA*](http://en.wikipedia.org/wiki/Antisense_RNA) and often providing a negative feedback loop, but the interference is then removed from the DNA, and does not represent a direct IF-statement analog at the DNA stage.

To me, GOTO mainly makes sense for sequential code execution, and this is not the case for DNA (lots of transcription is happening all the time in parallel). More generally, the parallel "execution" of DNA along with the continuous interactions and feedback loops between DNA, transcripts and proteins (among other things) also means that cellular processes are far less clear-cut and traceable than computer code, which means that computer code is a very poor metaphor for cellular processes and the functioning of DNA.