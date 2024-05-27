<!--yml
category: 未分类
date: 2024-05-27 13:25:45
-->

# AIQUAL.80[AM,DBL]1 - www.SailDart.org

> 来源：[https://www.saildart.org/AIQUAL.80[AM,DBL]1](https://www.saildart.org/AIQUAL.80[AM,DBL]1)

perm filename AIQUAL.80[AM,DBL]1 blob

[sn#568256](AIQUAL.80%5BAM,DBL%5D1_blob)

filedate 1981-02-26 generic text, type C, neo UTF8

```
COMMENT ⊗   VALID 00008 PAGES
C REC  PAGE   DESCRIPTION
C00001 00001
C00002 00002	.journal
C00006 00003	.sec |Mechanics of the Examination|
C00012 00004
C00038 00005	.sec |Readings|
C00041 00006	.ss |References|
C00063 00007	@Hendrix, G.
C00082 00008	Nilsson, N. J. (1974) [OVERVIEW] Artificial Intelligence,
C00103 ENDMK
C⊗;
.journal
.every heading (Stanford AI Syllabus - 1980,,Page {page!})
.every footing (,,)

.page frame 60 high 80 wide;
.area text lines 4 to 58;
.title area heading lines 1 to 3;
.title area footing line 60;
.place text;

.single space
.underlinebetween (<<,>)
.noContents ← true;
.blankline;
.skip 5
.begin center
Syllabus for Qualifying Exam
in Artificial Intelligence
.skip 2
Department of Computer Science
Stanford University
Spring 1980
.end

.skip 7
.indent1 ← 5

The syllabus is organized to present a picture of the range of
knowledge expected of Ph.D. candidates in Artificial Intelligence, rather
than specifying a fixed list of readings.  There are a number of different
dimensions along which we could divide up the material.  The attempt in
the earlier version to establish a thorough categorization has been
replaced this year with a less formal, more realistic organization.  We
have listed a number of "topics" with a short paragraph describing the
necessary reading for each.  These topics overlap in various ways, and
reflect idiosyncratic views of how things divide up without any attempt to
provide a consistent classification. Hopefully the set of references
included with each item will make it possible for students to select a
reasonable number of readings which will fill any knowledge gaps.  The
long reading list is intended as a source of details on individual
references, not as a necessary set of things to read.  It includes a rough
indication of what sort of understanding is most important for each
reference -- whether there is a general perspective, one or more specific
concepts, and/or a body of detail with which students are expected to be
fluent.  These indications are of course based on the prejudices and
peculiarities of the committee making up the syllabus, and should not be
taken as representing the views of anyone else (including the members of
individual exam committees).

Please send any comments or suggestions on the syllabus to
Doug Lenat (LENAT@SUMEX).
We are hoping to get lots of feedback, and continue building toward
a syllabus which really describes what there is to know [and what
is important to know] about AI, to be built on year after year.
.sec |Mechanics of the Examination|

The examination will be an individually-scheduled oral, before a
committee consisting of three members, chosen from the faculty,
adjunct faculty, senior research staff, and possibly appropriate
senior researchers from AI facilities in the area like SRI and Xerox
PARC.  Each committee will include at least one faculty member and at
least one person in a potential area of specialization of the
candidate.  The candidate can request a particular person in his or
her area, but the Qual committee has final choice of examining
committees.   We plan to hold the examinations during the first
two weeks of June.  If there are special reasons why someone cannot
take it at that time, we will try to make other arrangements.

At least two weeks before a student's examination, he or she will be 
given either:  (i) a problem to be worked on an open-book basis; or (ii)
a research paper on which he or she will write a critique.
Those students who prepared papers for the qual last year, but
did not pass the oral exam can use the same papers as the basis
for this year's exam if they wish.  The papers are to be
handed in to the qual committee (Doug Lenat or Terry Winograd)
no later than one week before the exam.  The members of the individual exam
committee will be given copies of the solution or paper, and the first section
of the exam (up to an hour) will center around issues raised by the
work done.  The rest of the exam will be on any
questions the examiners consider appropriate. 
The purpose of the exam is to demonstrate that the student has done
sufficient reading and thinking to fit his or her individual research
into a perspective of other work in AI.  This includes detailed
knowledge of some other existing work, both in the sub-area in which
the student intends to do research, and in other sub-areas.  The
committee should be satisfied that the student already has a
sufficient grasp of both the general issues and of a reasonable
amount of technical detail.
It would be unwise to assume that it is only necessary to know
a subset of the topics listed below by matching them to the
individual examiners.  The purpose of the exam is to look for bredth,
not for conformity to the particular committee, and questions from
all areas are fair game.
The exam is not intended as a device to
decide who will and will not be able to continue in the program, but
rather a way of focussing effort on a comprehensive study of AI, and a way of
providing students with specific diagnostics for gaps in their
knowledge or understanding.  The possible outcomes of the exam are:

.crown (5,10,5)
Pass unconditionally: The student has a satisfactory knowledge of all
areas.

Pass conditionally: The student has some lacks which can be made up
by directed work, such as the completion of specific course(s) or
specific research project(s).  The committee will set both the scope
of the work and a time period in which it must be completed in order
for the examination to count as passed. 

Continuation of the examination: The student has a lack which demands
a moderate amount of further study in one or more areas, and a second
oral examination (with the same committee) will be scheduled within
the next two quarters. 

Defer: The student has significant gaps in knowledge which cannot be
made up by limited correctives.  Candidate is required to take the
exam another time it is offered, under whatever system is in effect
then. 
.endcrown

.ss |Topics to be studied|

As mentioned above, this is not intended as a complete or structured
classification.  It is a list of answers to the vague question "What
kinds of things should AI students know about?".  There is no significance
to the ordering.
.stoptext

	General Perspective
	Weak Methods
	Epistemological problems of AI and the use of formal logic
	Knowledge Engineering - Expert Systems
	Knowledge Representation Formalisms
	Game Playing
	Planning and Common-sense Reasoning
	Mathematical theorem proving and discovery
	Natural Language
	Speech understanding
	Vision
	Physical manipulation
	Automatic Programming and Program Verification
	Learning and Inductive Inference
	Psychological Models
	Automata and Formal Language Theory
	Programming Languages for AI
	Philosophical Implications
	Political and Social Implications
	History and politics of the field
.starttext
.SS |General Perspective|

There are several recent books on AI which attempt to provide an
overview.   Of these, [Boden AI] and [Winston AI] are the best
starting point. One recent book ([McCorduck AI]) detailes the early history 
and sociology of the field.
Shorter articles which provide long-term perspective
are [Minsky STEPS], [Feigenbaum IFIP], [Lenat UBIQ], and [Nilsson OVERVIEW].
In addition, the AI handbook will provide an overview of lots
of AI issues. [Nilsson AI2] is now available, and its novel organization cuts
across most of the categories on our list above.

.SS |Weak Methods|

Classicaly, AI has been associated with a set of methods for 
problem solving and search which have been called "weak methods".
These include early work such as GPS [Newell, Shaw, and Simon],
notions of heuristic search as described in [Nilsson
AI] and [Handbook SEARCH], and more theoretical ideas abot
problem spaces discussed in [Newell ILL] and at length in
[Newell & Simon HPS].  Students are expected to be familiar enough
with the technical details to demonstrate how the techniques
operate, but will not be asked to prove theorems or remember complex
results.

.SS |Epistemological problems of AI and the use of formal logic|

The problem of describing facts about the world including the effects
of actions has been studied apart from specific problem solving
programs.  This work has used first order logic to express facts
about the world.  These issues are discussed in [McCarthy and Hayes]
and in [Hayes DEFENCE].  Many current issues are discussed in [McCarthy
5IJCAI].  A new approach is described in [Weyrauch PROLEGOMENA].
Truth maintenance is covered by de Kleer, Doyle, and others in
[Winston & Brown].  The relevance of theorem proving to problem
solving is discussed in [Green TP], and the techniques of
resolution theorem proving are described in [Nilsson AI].  As
with weak methods, students are expected to be familiar with
the basic mechanisms (e.g. be able to demonstrate a simple proof
by resolution, or explain the issues in unification) but
will not be required to know sophisticated technical results
(e.g. prove the completeness of resolution with the X heuristic).
A tutorial on some of the relevant mathematics is in [Manna MTC].

.SS |Knowledge Engineering - Expert Systems|

Much of current AI work is being subsumed under the heading of "knowledge
engineering".  [Bernstein KBS] is a general survey of knowledge based
systems.  [Winston AI] gives a general idea of systems of this kind done
at MIT, and [Feigenbaum IJCAI5] describes the general approach.  A number
of expert systems have been built in the past few years.  Students should
be familiar with the general capabilities and design.  
[Handbook APPLICATIONS] provides some details (and many pointers) for most
of the recent efforts.  Applications to Music [Moorer][Zaripov] and Art
[Gips][Cohen IJCAI6] should also be looked over.  The MIT perspective is
covered in [Winston & Brown], V.1, Sec.1.

.ss |Knowledge Representation Formalisms|

A number of current AI research projects are centered around the
development of knowledge representation languages.  Some of the early
issues in representation are discussed in [Amarel ACTIONS] and [Bobrow
DIMENSIONS].  More recent discussions include [Winograd FRAME], [Hayes
DEFENCE], and [Winograd EXTENDED].  Students should be familiar with at
least the following general approaches: Procedural Embedding,
Semantic Networks, Conceptual Dependency, Frames (Scripts, etc.),
Production Systems,  and Description Languages.
These are treated well in [Handbook REPRESENTATION].

.ss |Game Playing|

One of the earliest and most publicized areas of AI research has been game
playing programs, such as those for checkers [Samuel in C&T] and Chess
[Greenblatt FJCC].  Students should be familiar with the basic techniques
(e.g. Minimax and alpha-beta) and some of the more subtle problems (e.g.
the horizon effect [Berliner IJCAI3] and the use of patterns [Simon
1973]).  Berliner's articles in IJCAI5 and IJCAI6 also merit attention.
Basic techniques are taught in [Nilsson AI] and [Slagle AI].

.ss |Planning and Common-sense Reasoning|

Much of the AI work related to robotics dealt with the planning of action
sequences.  GPS [Newell, Shaw and Simon] was the early classic, and other
well known systems are STRIPS [Fikes, Hart, Nilsson] and NOAH [Sacerdoti
NONLINEAR].  An early discussion of the problems is in [McCarthy
ADVICE-TAKER].

.ss |Mathematical theorem proving and discovery|

Mathematics has always been an important domain for AI.  The Geometry
theorem prover [Gelernter] was an early program.  More recent efforts in
mathematical theorem proving are discussed in [Bledsoe MAN-MACHINE].  The
mechanization of mathematical discovery is discussed in [Lenat IJCAI5].
Non-Computer Science treatments of some interest are [Polya] and [Lakatos].

.ss |Natural Language|

Much of the research on natural language is summarized in the handbook
articles [Handbook NL] and the drafts of [Winograd LANGUAGE].  Students
should be aware of the general content of these and familiar with the
issues which arise in parsing and reasoning with natural language.  This
includes a level of understanding of parsing techniques (CFG, ATN, TG,
etc.) similar to that of heuristic search and theorem proving discussed in
sections above.  Early work in NL is described in [Simmons SURVEY 1965]
[Simmons SURVEY 1970], [Minsky SIP (browse)], and [Weizenbaum ELIZA].
[Handbook NL] gives a summary of work in Machine Translation.  Many papers
reflecting current research interests are found in [TINLAP] 1 and 2.

.ss |Speech understanding|

Work in Speech systems is well summarized in [Handbook SPEECH].

.ss |Vision|

[Winston VISION] contains a good introduction to some areas of scene analysis.
[Winston & Brown], V2, Sec1, present more recent MIT vision efforts,
notably Marr's work on representation.
[Waltz] presents work on analysis of line drawings.
[Thomas and Binford] give an early comparison of machine perception and natural
perception.
[Nevatia and Binford] describe recognition of complex objects.
[Moravec], [Gennery], and [Marr and Poggio] deal with stereo vision.
[Brooks, Greiner, and Binford] present a model-based vision system.
[Barrow & Tenenbaum] describe their efforts at SRI.
[Land] describes color visual perception.

.ss |Physical manipulation|

Stanford AI films make a good introduction to robotics.  
[Bolles and Paul] describe the first computer-controlled assembly.
[Finkel, Bolles, and Taylor] present AL, the Stanford language
for mechanical assembly.  [Lieberman] gives a description of a
very high level language for robotics.
[Binford et al] give an overview of robotics at Stanford.
[IJCAI6] contains many brief articles which give a picture of current
Japanese projects and achievements (unfortuanately, said picture could
stand some image enhancement, and we refrain from recommending any
particular articles.)

.ss |Automatic Programming and Program Verification|

There has been work in AI devoted to automating the process of
writing programs.  Some surveys of the field are given in [Biermann APPROACHES]
[Green-INFORMAL], and in the last section of [Manna and Waldinger DREAMS].
Also see [Handbook AP] for a detailed, up-to-date survey.
Approaches range from formal theorem-proving techniques [Manna and
Waldinger DEDUCTIVE] to knowledge-based systems [Green IJCAI5] to systems 
that attempt to simulate human reasoning [Lenat BEINGS], 
[Shrobe MONOLOGUE].  Students should
be familiar with these approaches, but do not need to know the details of
any particular system.

A closely related area is program verification -- ascertaining that a
program does, indeed, do what it is supposed to. Some perspectives on
current verification research and discussion of opportunities for
extension are found in [Gerhart-1980s].  Other surveys of the field of
verification are [Luckham PV & VOP] and [London PERSPECTIVES].

.ss |Learning and Inductive Inference|

Much of the work in AI can be view as an effort to get a program to
'learn' -- typically, to view a set of examples, and induce the common
concept uniting them.  An excellent survey of this work is in [Smith et al
IJCAI5].  Winston [STRUCTURAL] provided a seminal contribution,
emphasizing both the necessity for adequate description, and the use of
negative examples to more precisely circumscribe the concept desired.
Students should be familiar with these papers, and with the issues
involved.  They should also understand the application of learning
techniques to various domains (e.g., [Buchanan THEORY-FORMATION], [Fikes
et al GENERALIZED], [Sussman SKILL]), but need not know the technical
details of all these cases.

.ss |Psychological Models and Human Problem Solving|

Many AI programs have been intended as models of human information
processing.  Information processing psychology in general is described in
[Newell and Simon HPS (Sections 1 and 5 especially)].  An early program
which modelled human verbal behavior was [Feigenbaum EPAM].  More recent
models are found in [Anderson & Bower HAM (chapters 4 and 7)],
[Norman & Rumelhart EXPLORATIONS], and [Collins & Quillian USER].  A
famous early program which took one form of intelligence tests was [Evans
ANALOGY], and a controversial model of paranoia is described in [Colby
SIMULATIONS].  Relevant classics from psychology include [Bartlett
REMEMBERING] and [Miller MAGICAL].  Some good books on human problem solving
are those by Wickelgren, Polya, and Lakatos; see also [Sloman AIJ].  Students
should be familiar with (at least) the first and last chapters of
[Newell&Simon HPS].

.ss |Automata and Formal Language Theory|

Although this is not AI per se, it forms an important part of the
background. Relevant work is in [Minsky COMPUTATION], [Manna MTC] and the
material on perceptrons in [Hunt AI] (extra detail in [Minsky & Papert
PERCEPTRONS]).

.ss |Programming Languages for AI|
.stoptext

	List Processing -- LISP
	String processing -- SNOBOL
	Associative mechanisms -- LEAP/SAIL
	Active data structures -- SIMULA/SMALLTALK/ACTORS
	Pattern Matching		[Bobrow & Raphael]
	Data Structures			[Knuth Vol.I]
	PLANNER, CONNIVER, QA4, etc.	[Bobrow & Raphael]
	Production Systems		[Davis & King OVERVIEW]

.starttext

The candidate is expected to be familiar enough with some AI language (e.g.
LISP or SAIL) to demonstrate the ability to write simple programs.  He or she
should also know enough about the features of the more specialized languages
(MicroPlanner, QA4, the LEAP and multiple process features in SAIL)
to discuss
the kinds of problems for which they are useful, and the limitations they
force the programmer into.  It is not necessary
to know syntactic details of these features.

.ss |Note:|

Last year there was some controversy about including the following topics
on the reading list.  McCarthy stated,
"I think the syllabus should downplay philosophical and social issues,
because the we certainly don't want to grade students on their own
views, and the field is fluffy enough as it is without letting or
requiring students to be able to regurgitate various people's views
on the issues.  If the faculty wants students to have exposure to
these issues it should require attendance at a seminar or lecture
series and take attendance but not examine."
Winograd said, "The purpose of the qualifying examinations is to ensure
that students graduating from our department have a sufficient background
to work as qualified researchers and teachers in the field.  I believe that
a perspctive on what we are doing and why is a critical part of the needed
background, and that it would be irresponsible for us as a teaching institution
to leave it out.  Since our general philosophy is to judge competence in all
areas by exams rather than required courses, this is no different.  Indeed
nobody is expected to endorse or regurgitate anyone's views, but (as in
all areas) to demonstrate that they are aware of the important issues and have
thought about them."

.ss |Philosophical Implications|

There has been a continuing discussion about the philosophical implications
of intelligent machines.  The original paper often cited is
[Turing TEST].  The classical book criticizing AI from a philsophical
point of view is [Dreyfus CAN'T].  Many of the general
issues are discussed in the fascinating [Anderson MINDS] and
in [Dennett BRAINSTORMS].  There is an accessible discussion of the issue in
[Boden AI].  A recent controversy dealing with AI and the philosophy
of science is in [Dresher & Hornstein SUPPOSED], [Winograd CONTESTED],
and [Dresher and Hornstein RESPONSE].  [Sloman REVOLUTION] is a new 
important reference (JMC has a copy and the library
has one).  [Hofstater GODEL] is bizarre, whimsical, at times provocative, and
at times informative.

.ss |Political and Social Implications|

One of the pioneers in the field discussed these issues in [Weiner HUMAN],
and more recently there has been a discussion raised by [Weizenbaum
REASON], and responses like [Buchanan, Lederberg, and McCarthy REVIEWS].
[Boden AI] also discusses some
of the important questions.  [Firschein & Coles IJCAI3 survey] attempts to
predict some of the future applications of AI.  An important controversy
in British support for AI is in [Lighthill AI] and responses in the same
booklet and in [McCarthy LIGHTHILL].

.ss |History and Politics of the Field|

A critical part of being able to do and evaluate research in a field
is having some perspective on what things have been done and why --
in particular, what lessons have been learned and how can they
be applied to keep from repeating mistakes.  In addition, students
need to know where and how things are published in the field if
they hope to keep up with current work.  It is difficult to give specific
references, but the general books on AI [e.g. Boden, Winston, McCorduck,
Raphael] each give some perspective.  It is also critical to look at
some of the early collections of papers, including a
careful reading of most of [Feigenbaum & Feldman].

Students should have some familiarity with the early
history of AI -- its connections to cybernetics and machine translation in
the 50's.  It is interesting to note some of the early optimistic attempts
at "self-organizing systems" and work on perceptrons and neural nets.
Today, there is much discussion about the emergence of a discipline called
"cognitive science" which includes work now considered artificial
intelligence, psychology, linguistics, and philosophy.  Students should
have done some thinking about the relationships between these disciplines,
especially where they adopt differing methodologies in looking at the same
phenomena.  

The following journals (and conferences) present material relevant to 
AI, and it is useful to have a general idea of what kinds of things each 
of them contains. Given a paper, you should be able to discuss where it 
ought (not) to be submitted.
.stoptext

	Journal of AI
	SIGART 
	SIGCAS 
	Machine Intelligence (1 - 9)
	IJCAI proceedings (1 - 6)
	TINLAP proceedings (1 - 2)
	CACM 
	JACM 
	Cognitive Psychology 
	American Journal of Computational Linguistics
	Cognitive Science Journal
	Cognitive Science Society
	The Behavioral & Brain Sciences
	AAAI
	AISB
	Special interest conferences: cybernetics, natural language, 
		robotics

.starttext

.sec |Readings|
.ss |Explanation:|

The following list is NOT a list of required readings, but a
guide to interesting, available materials in each subfield.
Some of the papers are not yet published but will be made
available.

A simple annotation system has been used to give some feeling for the
level and importance (measured idiosyncratically by the syllabus
preparers) of each paper.  The level is indicated as:

.stoptext

	ESS:  An essay giving general perspective and discussion of basic 
	        issues
	SURV: A survey of research work in some area
	SUM:  A summary at a non-detailed level of a specific piece of 
	        research
	DR:   A detailed research report
	TEXT: A textbook

.starttext

The importance is based on whether the paper is important for
understanding a general perspective (P), whether it has one or
more specific important ideas (I), or whether it gives a set of
details over which students are expected to have fluent command (D).
We repeat: these are opinions, hastily arrived at! 

A basic background can be acquired via the books marked "@@"; gaps
may be filled with those papers marked "@".  These together can
be thought of as an initial reading list.

Many of the references below point to more specialized sources for those
highly interested; it is expected that each candidate will be highly 
interested in some areas (hint).  We also hope that each student will 
have  only a few gaps (at study-time, not at exam-time) and it will be 
possible to glance over the outline, choosing to read in those areas 
where the references pointed to are not familiar.

.ss |References|
.crown (3,0,0)
.skip 1

Adams, J. <<Conceptual Blockbusting>, W.H.Freeman.

@Agin, G. and Binford, T.  (1973) Computer Description of Curved
Objects. <<Proc IJCAI3>, 1973, pp 629-640 (SUM,I:the basic representation)

@@<<Artificial Intelligence Handbook>;  [AIHandbook], a comprehensive 
encyclopedia of work in AI, prepared by HPP; copies are available in
the library (or by bothering Barr or Feigenbaum).    (SURV!!)

@Amarel, S. (1968) On Representations of Problems of Reasoning About
Actions, in <<Machine Intelligence 3>, pp. 131-171 (eds Meltzer and
Michie), New York: American Elsevier Publishing Company. 
(DR,P,I:effects of change of repr. on problem solubility, )

Anderson, J. R., and Bower, G. H., <<Human Associative Memory>, V.
H. Winston and Sons, Washington, D.C., 1973\. Especially Chapters 4, 5, & 7.
(SURV & DR,P)

Anderson,(ed), <<Minds and Machines>, Contemporary Perspectives in
Philosophy Series, Prentice-Hall, Inc, N.J., 1964.
(Collection of essays, each of which is rated at least: (ESS, ))

Arnold, R. D.; "Local Context in Matching Edges for Stereo Vision",
<<Proc ARPA Image Understanding Workshop>", Cambridge, May 1978, 65-72\. (DR)

Balzer, Robert, Neil Goldman, and David Wile, "On the
Transformational Implementation Approach to Programming", <<Proceedings
Second International Conference on Software Engineering>, Computer
Society, Institute of Electrical and Electronics Engineers, Inc., Long
Beach, California, October 1976, pages 337-344.

Barr, A. and Feigenbaum, E., <<Artificial Intelligence Handbook>.  Please
see [AIHandbook] above in this listing.

Barrow,H.G., J.M.Tenenbaum; "MSYS: A System for Reasoning about Scenes";
<<SRI AI Center Tech Note 121>, April 1976\. (SUM)

Barrow, H.G. and Tenenbaum, J.M.;
"Recovering Intrinsic Scene Characteristics from Images";
<<SRI International AI Center>, Apr 1978, Tech note 157\. (ESS)

Barstow, David R., A Knowledge Base Organization for Rules about 
Programming, <<Proc. IJCAI5>, 1977, 382-388 (DR)

Bartlett, Frederick, <<Remembering: A Study in Experimental and Social 
Psychology>, Cambridge, University Press, 1932  (DR,I: memory is active, 
not passive)

Baumgart, B.; "Geometric Modeling for Computer Vision", 
Stanford AI Memo AIM-249, CS-463, 1974\. (DR)

Berliner, Hans, Some Necessary Conditions for a Master Chess Program,
<<Proc IJCAI3>, 1973, pp. 77-85.
(SUM,I)

Berliner, Hans, Experiences in Evaluation with BKG - A Program that Plays
Backgammon, <<Proc IJCAI5>, 1977\. (SUM,)

Berliner, Hans, On the Construction of Evaluation Functions for Large Domains,
<<Proc IJCAI6>, 1979\. (SUM,I)

@Bernstein, M.I., <<Knowledge-Based Systems: A Tutorial>, TM-(L)-5903/000/00A, SDC,
 June 1977 (SURV,P)

Biermann, Alan W., "Approaches to Automatic Programming," in
<<Advances in Computers>, Volume 15, Academic Press, Inc., New York,
New York, 1976\. (SURV,P)

@Binford, T.O; "Visual Perception by Computer"  
Invited paper for IEEE Systems and Control, Miami, 1971\. 
(ESS, I: generalized cones)

Binford, T.O., "Computer Integrated Assembly Systems,"
<<Proc NSF Adv Prod Techn Grantees Conf>, Nov, 1978.

Bledsoe, W.W., and Bruell (1974) A Man-Machine Theorem-Proving
System, <<Journal of AI>, 5, 1974, pp 51-72\. 
(SUM,P)

@Bobrow, Dan, and Allan Collins, editors, <<Representation and
Understanding>, New York: Academic Press, 1975
(a collection of papers, each at least (SUM,I))  See esp. "Dimensions of
Repr." and Woods' "What's in a Link?"

@Bobrow and Raphael, (1973) New Programming Languages for AI Research,
<<Computing Surveys>, 6, 1974, 155-174\. (SURV, I)  Still quite relevant, 
after all these years.

@Bobrow, D. and T. Winograd
An Overview of KRL, a Knowledge Representation Language,
<<Cognitive Science>, 1, 1977, 3-46 (SUM,I: controlled processing and matching)

Boden, Margaret A. <<Artificial Intelligence and Natural Man>, New York,
Books, Inc., 1977 (TEXT & SURV,P) Probably the best comprehensive
nontechnical introduction to AI.

Bolles,R.C.; "Verification Vision for Programmable Assembly";
<<Proc. IJCAI5>, 1977, 569-575\. (DR)

Bolles,R.and R.Paul,
"The use of Sensory Feedback in a Programmable Assembly Systems",
Stanford AI Lab Memo AIM-220, CS-396, AD772064/2WC, 1973.

@Brachman, R.J., What's in a Concept: Structural Foundations for Semantic Networks,
<<Int'l J. Man-Machine Studies>, 9, 1977, 127-152 (DR,P)

Braid,I.C.; <<Designing with Volumes>,
Univ of Cambridge, Cantab Press, Cambridge, England, 1973\. (ESS & DR)

Brooks, R., R. Greiner, and T.O. Binford;
"ACRONYM: A Model-Based Vision System"; <<Proc IJCAI6>, 1979.

Bruce, Bertram, Case systems for natural language, <<Artificial
Intelligence>, 6:4, Winter 1975, 327-360\. (SURV,P)

Buchanan, B., Feigenbaum, and Sridharan (1972) Heuristic Theory Formation,
<<Machine Intelligence 7>, pp.267-280 (the appendix may be omitted). 
(SUM, I:choosing proper domain is important)

Buchanan, Bruce, Joshua Lederberg, and John McCarthy, Three
reviews of J. Weizenbaum's <<Computer Power and Human Reason,>
Stanford AIM-291, STAN-CS=76-577, November 1976.

Chandrasekharan and Reeker (1974) AI: A Case for Agnosticism, in <<IEEE
Transactions on Systems, Man, and Cybernetics>; January, 1974, pp.88-94\. 
(ESS,P)

Chase, W.G., editor, <<Visual Information Processing>, Academic
Press, New York, 1973\. 
(Collection, each of which is at least (SUM, ))  See esp. Newell "Production
Systems: Models of Control Structures".

Cohen, Harold, What is an Image?, <<Proc IJCAI6>, 1979\. (DR, just glance over;
SUM, P, I: Intelligence is in the eye of the beholder)

Cohen, Philip
On Knowing What to Say: Planning Speech Acts, Tech Rpt 118, U. of 
Toronto CSD, January 1978 (DR,I: use of planning to recognize/generate 
speech)

Darlington and Burstall (1973) A System Which Automatically Improves
Programs, <<Proc. IJCAI3>, pp. 479-485\. 
(SUM, I:encoding knowledge into schemata rewriting rules)

@Davis, Randall
Generalized Procedure Calling and Content-Directed Invocation, <<Proc. ACM Symposium
AI & PL>, 1977, 45-54 (SURV,I: procedures should state their effects)

Davis, Randall,
Meta-Level Knowledge: Overview and Applications, <<Proc IJACI5>, 1977, 920-927 
(SUM,I: meta-knowledge)

@Davis, Randall, and Jonathan King
An Overview of Production Systems, in <<Machine Intelligence 8>, E. Elcock and 
D. Michie (eds), Chichester, Ellis Horwood, 1977 (SURV,P)

Davis, Randall, Bruce Buchanan, and Edward Shortliffe,
Production Rules as a Representation for a Knowledge-Based consultation Program,
<<Artificial Intelligence>, 8, 1977, 15-45
(DR,I: organization and use of knowledge, computer-based consultation)

Dresher, B.E., and N. Hornstein.
On Some Supposed contributions of Artificial Intelligence to The Scientific
Study of Knowledge, <<Cognition>, 4, 1976 (ESS,P)

Dresher, B.E., and N. Hornstein.
Reply to Winograd, <<Cognition>, 5, 1977, 379-392 (ESS,P)

Dreyfus, Hubert, <<What Computers Can't Do>, Harper and Row, 1972 (and
later edition).  (ESS,P)

@Erman, L.D. and V.R. Lesser
A Multi-level Organization for Problem Solving Using Many Diverse Cooperating
Sources of Knowledge, <<Proc IJCAI4>, 1975, 483-490 (DR,I: modular knowledge sources, blackboard)

Ernst, George W., and Newell, Allen, <<GPS: A Case Study in
Generality and Problem Solving>, Academic Press, New York, New
York, 1969\. Not central, but many good ideas are inside.
(DR, )

Fahlman, Scott E.
A System for Representing and Using Real-World Knowledge, AI-TR-450, MIT AI Lab, 
December 1977 
(DR,I: certain inferences should be made quickly and easily, using hardware)

@Falk, G. (1972) Interpβetation of Imperfect Line Data as a
Three-dimensional Scene, <<Artificial Intelligence>, 3, 1972, pp. 101-144\. 
(SUM, I:good when nature falls into a few tens of categories, so task is hard but
doable.)

Feigenbaum, E. et al (1971) [DENDRAL] On Generality and Problem Solving: A Case
Study Using The DENDRAL Program. (eds Meltzer and Michie) <<Machine
Intelligence 6>, pp 165-190\. More detail than [AIHandbook].
(SUM, I:A useful application; Choose domain carefully)

@@Feigenbaum, Edward A., and Feldman, Julian, editors, <<Computers and
Thought>, [C&T], McGraw-Hill Book Company, New York, New York, 1963.
Old but almost each article is still quite worth a careful reading; to name
a few: Feigenbaum, Gelernter, Newell et al, Turing, Armer, Minsky.
(Collection; some (ESS,), most (SUM,I))

@Feigenbaum, E.A.
The Art of Artificial Intelligence: Themes and Case Studies of Knowledge Engineering,
<<Proc IJCAI5>, 1977, 1014-1029 (SURV,I: use of large amounts of domain-specific knowledge)

Feldman, Pingle, Binford, Falk, Kay, Paul, Sproull, Tenenbaum. (1971)
The Use of Vision and Manipulation to Solve the Instant Insanity
Puzzle, <<Proc IJCAI2>.
(SUM, )

Feldman, Jerome A. & Robert F. Sproull
Decision Theory and Artificial Intelligence II: The Hungry Monkey, <<Cognitive Science>,
1, 1977, 158-192 (DR,I: use of decision theory to allocate scarce resources)

@Fikes, R. E. et al (1972) Learning and Executing Generalized Robot
Plans.  <<Artificial Intelligence>, Vol. 3 (Winter 1972). 
(SUM, I:Triangle Tables; handling of Frame problem,plans. D: Triangle tables)

@Fikes, R. and G. Hendrix
A Network-based Knowledge Representation and its Natural Deduction System,
<<Proc IJCAI5>, 1977, 235-246 (SUM,I)

Findler, N.V., and Meltzer, B., editors, <<Artificial Intelligence
and Heuristic Programming>, Edinburgh University Press, Edinburgh,
Scotland, 1971, ? + ? pages, $15.00\. 
(SUM, a collection of papers)

Finkel, R., Russel Taylor, Robert Bolles, Richard Paul, Jerome Feldman,
"AL, A Programming System for Automation", AIM-243, CS-456, 130 pages, November 1974.
(DR, )

@Firschein, O., and Coles, S. (1973) Forecasting and Assessing The
Impact of Artificial Intelligence on Society. <<Proc IJCAI3>; 105-120
(SUM, Questionaire results: Look this over, at least.)

@Floyd, R. W. (1971) Toward Interactive Design of Correct Programs,
<<IFIP 71>, (ed., C.V. Freeman), Volume 1, pp. 7-10\. 
(ESS: An extended example. Brief, but contains an idea I: Auto.pgmming via dialogue)

Fogel, Lawrence J., Owens, Alvin J., and Walsh, Michael J.,
<<Artificial Intelligence through a Simulation of Evolution>, John
Wiley and Sons, New York, New York, 1966\.  Just glance over this and
notice what its significance is.
(DR, P:title of the book)

@Funt, Brian
WHISPER: A Problem-Solving System Utilizing Diagrams, <<Proc IJCAI5>, 1977, 459-464
(SUM,I: analogical reasoning may buy us some nice properties)

Gennery,D.B.; "A Stereo Vision System for An Autonomous Vehicle";
<<Proc IJCAI5>, 1977, 576-582\. (DR)

Gerhart, Susan L., <<Program Verification in the 1980s:  Problems,
Perspectives, and Opportunities>, Research Report ISI/RR-78-71,
Information Sciences Institute, University of Southern California, Marina
del Rey, California, August 1978.

Gips, J. "Shape grammars and their uses",  AI memo 231,  last third is on
aesthetics.  Recently published (Stiny & Gips): <<Algorithmic Aesthetics>,
UC Berkeley Press, 1978.
(DR, I:picture grammar, aesthetics related to simplicity)

@Goldman, Neil, Sentence Paraphrasing from a Conceptual Base, <<CACM>, 18:2
February 1975, pp. 96-107.
(SUM, I:integration of decision tree control with conceptual dependency).

Goldman, N., R. Balzer, and D. Wile, The Inference of Domain Structure from
Informal Process Descriptions, <<Proc. Workshop on Pattern-Directed Inference
Systems>, SIGART Newsletter, June 1977, 75-82 (SUM)

Goldstein, Ira, and Seymour Papert
Artificial Intelligence, Language, and the Study of Knowledge, <<Cognitive Science>,
1, 1977, 84-120 (SURV,P)

@Green, C. C. (1969) [TP] The Application of Theorem Proving to QA Systems,
Stanford Technical Report CS 138, SAIL Memo AI-96\. 
(DR, I:use of Resolution to perform deduction & answer questions)

Green, Cordell, "An Informal Talk on Recent Progress in
Automatic Programming", <<Lectures on Automatic Programming and List
Processing>, PIPS-R-12, Electrotechnical Laboratory, Tokyo, Japan, November
1976, pages 1-69\. (SURV,)

Green, Cordell, and Barstow, David, "On Program Synthesis Knowledge",
<<Artificial Intelligence>, Volume 10, Number 3, November 1978, pages
241-279\. (DR, I)

Greenblatt, R.B., D. Eastlake and S. Crocker, The Greenblatt Chess Program
<<Proceedings of the 1967 Joint computer conference>, 30:801-810, 1967.
(SUM, I:use of a variety of game playing techniques)

Grosz, Barbara J.,
Utterance and Objective: Issues in NL Communication, <<Proc IJCAI6>, 1979.
(SURV,P) If interested, examine her excellent IJCAI5 paper also.

Guard, J.R., et al. (1969) Semi-Automated Mathematics, <<JACM> 16,
January, 1969, pp. 49-62.
(SUM, D:A new result in Math has been established by a computer program: SAM's LEMMA)

Hayes, Patrick J.
Computation and Deduction, <<Proc MFCS Symposium>, Czech. Acad. Sciences, 1973 (ESS,P)

Hayes, Patrick J.
Some Problems and Non-Problems in Representation Theory, <<AISB Summer 
Conference>, 1974 (ESS & SURV, D)

@Hayes, Patrick J.
In Defence of Logic, <<Proc IJCAI5>, 1977, 559-565 (ESS,P)

Heidorn, George, Automatic Programming Through Natural Language Dialogue:
A Survey, <<IBM Journal of Research and Development>, Volume 20, Number 4,
July 1976\. (SURV, P)
@Hendrix, G.
Expanding the Utility of Semantic Networks through Partitioning, <<Proc IJCAI4> 1975,
115-121 (SUM, I: use of spaces in nets, to achiev scoping, etc)

Hewitt, Carl
Viewing Control Structures as Patterns of Passing Messages, 
<<Artificial Intelligence>, 8, 1977, 323-364 
(DR,I: actors; Hewitt's only comprehensible paper)

Hunt, E. and S. Poltrock
The Mechanics of Thought, in <<Human Information Processing: Tutorials in
Performance and Cognition>, B. Kantowitz (ed), Hillsdale, Erlbaum, 1974 (SURV,P)

Hunt, Earl B., <<Artificial Intelligence>, Academic Press, Inc.,
New York, New York, 1975\. 
(TEXT, D:good coverage of pattern recog. and perceptrons)

<<International Joint Conferences in Artificial Intelligence>, held biannually
since 1969; proceedings available from program chairmen; best indicator of
current research trends

Jackson, Philip C., Jr., <<Introduction to Artificial
Intelligence>, Petrocelli Books, New York, New York, 1974.
Elementary; if you feel lost in some subfield, consult this.
(TEXT, )

Julesz, B., "Experiments in the Visual Perception of Texture";
<<Scientific American>, April 1975\. (SUM)

Kanade, T.; "A Theory of Origami World";
Dept of Comp Sci, Carnegie-Mellon Univ, 1978, CMU-CS-78-144\. (DR)

Kant, Elaine, "The Selection of Efficient Implementations for a
High-Level Language", <<Proceedings of the Symposium on Artificial
Intelligence and Programming Languages>, <<SIGPLAN Notices>, Volume 12,
Number 8, <<SIGART Newsletter>, Number 64, August 1977, pages 140-146.

Kellogg, Charles, Philip Klahr, and Larry Travis.
Deductive Methods for Large Data Bases, <<Proc IJCAI5>, 1977, pp 203-209 (SUM,I:
ABSTRIPS-like skeletal plans can help deduction)

de Kleer, Johan, Jon Doyle, Guy L. Steele Jr, & Gerald Jay Sussman.
AMORD: Explicit Control of Reasoning, <<Proc ACM Sym AI PL>, SIGART #64, August
1977, 116-125 (SUM,I)

@Kling, Robert E., A Paradigm for Reasoning by Analogy,
<<Artificial Intelligence>, 2, 1971, pp. 147-178.
(SUM, I:similar to Evans' idea, but analyzed further.)

Kuhn, Thomas, <<The structure of scientific revolutions>,
Chicago: University of Chicago press, 1972\. (ESS,P,I: Paradigm shifts)

Lakatos, Imre, <<Proofs and Refutations>,  
(ESS,I: sprial of criticism and improvement of conjectures)

Land, E.H.; "The Retinex Theory of Color Vision";
<<Scientific American>, Dec 1977\. (SUM)

Larson, J.B., and R.S. Michalski
Inductive Inference of VL Decision Rules, <<Proc Wkshp Patt-Dir Inf Systems>, 
SIGART 63, 1977, 38-44 (SUM)

Lehnert, Wendy,
Human and Computational Question Answering, <<Cognitive Science>, 1, 1977, 47-73 (SUM)

Lenat, Douglas B., BEINGS: Knowledge as Interacting Experts, <<Proc IJCAI4>, 1975, 126-133
(DR, I:beings)

@Lenat, Douglas B.,
Automated Theory Formation in Mathematics, <<Proc IJCAI5>, 1977, 833-842 (DR,I: heuristics 
to generate search)

Lenat, Douglas B., and John McDermott,
Less Than General Production System Architectures, <<Proc IJCAI5>, 1977, 928-932
(SURV, I: gain power by sacrificing generality)

Lesser, Victor R., and Lee E. Erman,
A Retrospective View of the HEARSAY-II Architecture, <<Proc IJCAI5>, 1977, 790-800 (SUM,P)

@Victor Lesser, Richard Fennell, Lee Erman and D. Raj Reddy, Organization
of the Hearsay II Speech Understanding System, <<IEEE Symposium on Speech
Recognition>, Computer Science Dept., Carnegie Mellon Univ., 1974.
(SUM, I:modular system organiztion)

Lettvin, J., H.R. Maturana, W.S. McCulloch, and W.H. Pitts, What
the frog's eye tells the frog's brain, <<Proceedings of the
IRE> 47(1959), pp. 1940-1951\. (SUM)

Lieberman, L., 
AUTOPASS, A Very High Level Programming Language for Mechanical Assembler
Systems,  IBM Watson Research Center Report RC 5599, No. 24205, 1975.

Lighthill, Sir J., and Sutherland, Needham, Longuet-Higgins, and
Michie (1973) AI: A Paper Symposium; by the British Science Research
Council, April, 1973\. A pro/con AI debate.  
Try to see the McCarthy, Michie vs. Lighthill debate on videotape.
(ESS, P, a general survey giving Lighthill's view on AI.  See McCarthy's response)

Lindsay, Peter H., and Norman, Donald A., <<Human Information
Processing: An Introduction to Psychology>, Academic Press, Inc.,
New York, New York, 1972\.  
(TEXT, P a comprehensive elementary introduction to cognitive psychology from a
view congenial to AI).

London, R. L., "Perspectives on Program Verification", in Yeh, R. T.,
(ed), <<Program Validation>, <<Current Trends in Programming
Methodology>, Volume 2, Prentice-Hall, Inc., 1977, pages 151-172.

Lozano-Perez, Tomas, and Patrick H. Winston, LAMA: A Language for Automatic
Mechanical Assembly, <<Proc. IJCAI-5>, 710-716\. (SUM)

Low, James, and Rovner, Paul, "Techniques for the Automatic Selection of
Data Structures", << Third ACM Symposium on Principles of Programming
Languages>, January 1976; also TR4, Computer Science Department,
University of Rochester, Rochester, New York, November 1975.

Luckham, D. C., "Program Verification and Verification Oriented
Programming", invited paper, in Gilchrist, B., editor, <<Information
Processing 77:  Proceedings of IFIP Congress 77>, North-Holland
Publishing Company, Amsterdam, The Netherlands, 1977, pages 783-793.

@McCarthy, J., [ADVICE-TAKER] "Programs With Common Sense",
Stanford AI Memo AIM-7,,AD785044, 7 pages, September 1963.
For details, also look at also the following memo:
(ESS, I:what we need is...)

McCarthy, J., "Situations, Actions, and Causal Laws", Stanford AI Memo 2,July 1963.
(ESS/DR: I:You can formalize these notions)

@McCarthy, J. and Hayes, P. (1969) Some Philosophical Problems from
The Standpoint of AI.  <<Machine Intelligence 4> (eds Meltzer and
Michie) pp. 463-502\. Edinburgh University Press. 
(ESS/DR: I/D: More of the same as the last reading. Further developed.)

McCarthy, John, Review of Lighthill debate, <<Artificial Intelligence>,
5, 1974, 317-322 (ESS,P) 

McCarthy, John, <<Mechanization of Thought Processes>, Her Majesty's Stationery
Office, 1958\.   Contains early McCarthy papers.

@McCarthy, J.,
Epistemological Problems of Artificial Intelligence, <<Proc. IJCAI5>, 
1977, 1038-1044 (ESS,I: what is AI still missing?)

@McDermott, Drew V.,
Vocabularies for Problem Solver State Descriptions, <<Proc. IJCAI5>, 
1977, 229-234 (SUM,I)

Manna, Zohar, <<Mathematical Theory of Computation>.
(TEXT, I/D: Read some book or article to gain familiarity with Prop 
and Pred Calc)

Manna, Zohar, and Richard Waldinger,
Synthesis: Dreams => Programs, AIM-302, Stanford, November 1977 (DR)

Manna, Zohar, and Richard Waldinger,
Knowledge and Reasoning in Program Synthesis, <<Artificial Intelligence>, 6,
1975, 175-208 (ESS,I/P)

Manna, Zohar, and Richard Waldinger, "A Deductive Approach to Program
Synthesis," SRI AI Center Tech. Note 177, Dec. 1978.

@Manna, Zohar, Six Lectures on the logic of computer proramming, Stanford
AIM-318, Nov 1978 (SURV, I/P)

@Marr,D. and T.Poggio; "Cooperative Computation of Stereo Disparity";
<<Science>, 194, Oct 1976, 283-287\. (SUM)

Marr,D., "Analysis of Occluding Contour",
MIT AI Memo AIM 372, Oct 1976\. (SUM)

Martin and Fateman (1971) The MACSYMA System, in (S. Petrick, ed.)
<<2nd Symposium on Symbolic and Algebraic Manipulation>. NY: ACM SIGSAM.
pp 59-75\. 
(SUM,I: application of AI techniques to a specific domain area)

Meltzer, Bernard, and Michie, Donald, editors, <<Machine
Intelligence>, volumes 1-6, American Elsevier
Publishing Company, New York, New York, volumes 7-8, Halstead
Press, New York; volumes 9- , John Wiley & Sons, New
York. Almost annually (since 1967).
Needless to say, don't study every article.
(Collection of articles, each at least (SUM, )).

@Meltzer, Bernard, and Bobrow, Daniel, editors, <<Artificial
Intelligence: An International Journal>, North-Holland Publishing
Company, Amsterdam, The Netherlands, quarterly (since 1970).
(Collection of articles, each at least (SUM, )).

Michie, Donald, <<On Machine Intelligence>, John Wiley and Sons,
New York, New York, l974.
(SURV, P)

@Miller, G.,
The magical number 7, plus or minus 2, <<Psychological Review>, 63, 1956, 
81-97 (ESS,I: memory can hold a fixed number of chunks, regardless of 
their complexity)

Minsky, Marvin, <<Computation: Finite and Infinite Machines>, Prentice
Hall, 1968.
Not in AI but you should know at least this much anyway.
(TEXT, I/D: Know at least this much about the theory of computation)

Minsky, Marvin, editor, [SIP]  <<Semantic Information Processing>, The
MIT Press, Cambridge, Massachusetts, 1968.
(Collection of MIT natl. lang. dissertations, all at least (SUM/DR, ))
See esp. Evans 
(SUM, I:power of using even a crude version of a single simple heuristic for
analogy) and 
Quillian (DR,I: network flow model for conceptual linking)
and Minsky (ESS, P: makes explicit many of the underlying notions of AI models).

Minsky and Papert  <<Perceptrons>, MIT 1969.
A sufficient expertise can be gained
from the appropriate section of [Hunt AI].
In looking at this book, try to read through 
page 25; then look through the rest; especially note the
concluding remarks, pp. 227-246\.  
(DR, I:applying rigourous mathematics to what AI-systems of different types
can theoretically achieve)

Mitchell, T.M.,
Version Spaces: A Candidate Elimination Approach to Rule Learning, 
<<Proc IJCAI5>, 1977, 305-310 (DR,I: the title)

Moore, Jim and Allen Newell, How can MERLIN understand?,
in Gregg (ed.) <<Knowledge and Cognition>,  New Jersey: Lawrence
Erlbaum Associates, 1973.
(SUM, I:Beta structures; criteria for understanding systems)

Moore, Robert Carter,
Reasoning about Knowledge and Action, <<Proc IJCAI5>, 1977, 223-227 (SUM,I)

Moorer, James A., "Music and Computer Composition", <<Comm. ACM>,
January 1972.
(SURV)

Moravec,H.P., "Towards Automatic Visual Obstacle Avoidance"
<<Proc. IJCAI5>, 1977, 584\. (SUM)

Mujtaba, S., and R. Goldman. "AL User's Manual";
Stanford AI Lab Memo, 1979\. (SUM, P)

Nash-Webber, Bonnie L., and Schank, Roger C. (eds.), <<Theoretical
Issues in Natural Language Processing, an interdisciplinary workshop
in computational linguistics, psychology, linguistics, and artificial
intelligence>, MIT, June 1975\. Preprints distributed by the Association
for Computational Linguistics. (a collection of papers, mostly SUM)

@Nevatia,R. and T.O.Binford; "Description and Recognition of Curved Objects";
<<Artificial Intelligence>, 8, p 77, Feb 1977; (DR)

@Newell, A. (1969) [ILL] Heuristic Programming: Ill-Structured Problems, in
(ed. Aronofsky, A.) <<Progress in Operations Research III>, John Wiley and Sons. 
(ESS, P)

Newell, A. (1965) Limitations of The Current Stock of Ideas about
Problem:Solving.  <<Proceedings of a Conference on Electronic
Information Handling>, pp. 195-208\. (eds Kent and Taulbee) 
New York: Spartan.  (ESS, interesting reading)

Newell, A. (1970) Remarks on The Relationship Between AI and
Cognitive Psychology, in (Banerji and Mesarovic, eds.) <<Theoretical
Approaches to Non-Numerical Problem Solving>, pp 363-400\.  New York:
Springer-Verlag Pub. 
(ESS, P)

Newell, A., Barnett, Jeffrey, Forgie, James W., Green, Cordell,
Klatt, Dennis, Licklider, J.C.R., Munson, John, Reddy, D. Raj, and
Woods, William A., [SPEECH]  <<Speech Understanding Systems: Final Report of a
Study Group>, American Elsevier Publishing Company, New York, New
York, 1973, xiv + 137 pages, $6.75\.  Read especially: Chaps. 1,4; Appendix A2\.  
(SURV, I:Evaluating research goals and guiding research toward them)

@Newell, A., and Simon, Herbert A., [HPS]  <<Human Problem Solving>,
Prentice-Hall, Englewood Cliffs, New Jersey, 1972, xvi + 920 pages. 
Read the first and last chapters, look over Chaps. 3,4,8\. 
(DR/ESS/SUM, D:know what LT  and GPS were, what a PBG is, production systems)

Nilsson, N. J. (1974) [OVERVIEW] Artificial Intelligence,
SRI Technical Note 89 (March, 1974), 
and also <<Information Processing 74>, North-Holland: Amsterdam, 1975
(SURV)

Nilsson, Nils J., [AI]  <<Problem-solving Methods in Artificial
Intelligence>, McGraw-Hill Book Company, New York, New York, 1971
(TEXT, but I/D:Good presentation of resolution, searching, αβ, etc.)

Nilsson, Nils J., <<Principles of Artificial Intelligence>, Tioga, Palo Alto,
1980\. (TEXT, attempts a new synthesis of much of the field)

Norman, Donald, D. Rumelhart, and the LNR Research Group, 
<<Explorations in Cognition,> Freeman, 1975.
(SUM, P: a collection of papers done from Norman's Psychology/AI viewpoint)

Park, W.T.; "Minicomputer Software Organization for Control of Industrial Robots";
<<Proc Int Jt Auto Control Conf>, San Francisco, 1977, p164, TA21\. (A survey
of existing systems) (ESS, SURV, P)

Pettigrew,J.D.; "The Neurophysiology of Binocular Vision"
<<Scientific American>, August 1972\. (SUM)

@Polya, G. Three representative books are listed here; you should be
acquainted with the kinds of principles Polya tries to impress, his
studies of heuristics. It is not necessary to study the detailed
contents of these books.
	<<How to Solve It,> Doubleday Anchor Books, 1945\. 
	<<Induction and Analogy in Mathematics,> Princeton U. Press, 1954\. 
	<<Patterns of Plausible Inference>, Princeton U. Press, 1968\. 
(DR, I:heuristics and how to use them)

Pople, H., Meyers, J., and Miller, R., DIALOG, a model of diagnostic
logic for internal medicine, <<Proc. IJCAI4>,
1975, 848-855\. (SUM)

Raphael, Bertram, <<The Thinking Computer>, W.H. Freeman, 1979.
(ESS, )

[IEEE] Reddy, D. Raj, editor, <<Speech Recognition: Invited Papers
Presented at the 1974 IEEE Symposium>, Academic Press, Inc., New
York, New York, 1975\. 
(Collection of articles, all at least (SUM, ))  See esp. Reddy & Erman,
pp. 457-480.

Reiter, Raymond, On reasoning by default, <<Theoretical Issues
in Natural Language Processing-2>, Urbana, Illinois: Association
for Computing Machinery, 1978, 210-218\. (SURV,I: non-monotonic logic)

Rieger, Charles J., III,
An Organization of Knowledge for Problem-solving and Language Comprehension,
<<Artificial Intelligence>, 7, 1976, 89-128 (SUM,I)

Rovner, Paul D., <<Automatic Representation Selection for Associative Data
Structures>, Ph.D. thesis, Harvard University, Cambridge, Massachusetts,
TR10, Computer Science Department,
University of Rochester, Rochester, New York, September 1976.

Rubin, S.; "The ARGOS Image Understanding System";
Dept of Comp Sci, Carnegie-Mellon Univ, Nov 1978, Ph.D.thesis (DR),
or <<Proc ARPA Image Understanding Workshop>,
Carnegie-Mellon, Nov 1978, 159-162\. (SUM)

Rumelhart, D., P. Lindsay, and D. Norman,
A Process Model for Long-term Memory, in <<The Organization of Memory>, E. Tulving
and W. Donaldson (ed), N.Y., Academic Press, 1972 (SUM,P)

Rustin, R., editor, <<Natural Language Processing>, Algorithmics
Press, New York, New York, 1973.
(Collection of articles, all at least (SUM, ))
See esp. W. Woods' ATN article.

Ruth, Gregory R., "PROTOSYSTEM I:  An Automatic Programming System
Prototype", in Ghosh, Sakti P., and Liu, Leonard Y., editors, <<AFIPS
Conference Proceedings:  1978 National Computer Conference>, Volume 47,
AFIPS Press, Montvale, New Jersey, June 1978, pages 675-681.

Rychener, M.,
Control Requirements for the Design of Production System Archtectures, <<Proc. ACM
Symposium AI & PL>, 1977, 37-44 (SUM)

@Sacerdoti, Planning in a Hierarchy of Abstraction Spaces,
<<Proc IJCAI3>, 1973, 412-422
(SUM, I:planning is just searching a sparser, more abstract space)

@Sacerdoti, E.,
The Nonlinear Nature of Plans, <<Proc IJCAI4>, 1975, 206-214 (DR,I: the title)

@[S&C] Schank, R. and Colby, K. <<Computer Models of Thought and
Language>, San Francisco: Freeman, 1973
(Collection; Note especially chapters 1,4,5,6)

Schank, Roger C.,  <<Conceptual
Information Processing>, Amsterdam: North Holland, 1975, See esp.
Rieger (pp. 157-288; DR,I: uncontrolled
forward inferencing is necessary in some situations) and Riesbeck
(DR,I: use of predictions in parsing).

Schank, Roger, Neil Goldman, Charles Rieger, and Chris Riesbeck,
MARGIE: Memory, Analysis, Response Generation, and Inference on English,
<<3IJCAI>, 1973, pp. 255-261.
(SUM, I: conceptual dependency for system integration)

Schank, Roger, and the Yale AI Project, SAM -- A story understander,
Yale University Computer Science Research Report #43, August,
1975\. (SUM)

@Schank, Roger, and Robert Abelson. Scripts, Plans, and Knowledge, <<Proc. IJCAI4>,
1975, 151-157 (SUM, I: scripts (frames, schemata))

Schank, Collins, and Charniak, eds., <<Cognitive Science>, journal, 
published quarterly since 1977 by Ablex Publishing Co, New Jersey

Schatz, B.; "The Computation of Immediate Texture Discrimination"
MIT Artificial Intelligence Laboratory, AI Memo 426, August 1977\. (DR)

Schmidt, C.F., and N.S. Sridharan,
Plan Recognition Using a Hypothesize and Revise Paradigm: An Example, 
<<Proc IJCAI5>, 1977, 480-486 (SUM,I)

Schubert, L.,
Extending the Expressive Power of Semantic Networks, <<Proc IJCAI4>, 1975,
158-164 (DR,I: how to represent quantification, etc, in semantic nets)

Schwartz, Jacob T., <<On Programming:  An Interim Report on
the SETL Project>, revised, Computer Science Department, Courant
Institute of Mathematical Sciences, New York University, New York, New
York, June 1975.

Shortliffe, Edward, <<MYCIN: Computer-Based Medical Consultations>,
New York: American Elsevier, 1976\. (DR,I: automated diagnosis)

@Shortliffe, Davis, Axline, Buchanan, Green, and Cohen,
Computer-based Consultations in Clinical Therapeutics:
Explanation and Rule Acquisition Capabilities of the MYCIN
System, preprint for article in Volume 8 of the <<Journal for
Computers in Biomedical Research>, June, 1975.
(SUM, I:Medical application of production systems; communication with experts)

Shrobe, Howard, Richard Waters, and Gerald Sussman, A Hypothetical
Monologue Illustrating the Knowledge Underlying Program Analysis, MIT LCS
Memo 506, January 1979\. (ESS,)

<<SIGART Newsletter>,
Special Interest Group on Artificial Intelligence (SIGART),
Association for Computing Machinery, New York, New York, quarterly (if you
believe that, you probably will have trouble passing the qual).
(SURV/SUM, )

Simmons, R. (1965) Answering English Questions by a Computer: A
Survey, <<CACM> 8, 1; January, 1965, pp. 53-70\. 
(SURV, gives reasonable picture of state of art at that time)

Simmons, R. (1970) Natural Language QA Systems. <<CACM> 13, 1; Jan.,
1970, pp. 15-30\. 
(SURV, gives reasonable picture of state of art at that time)

Simon, Herbert A.,
How Big is a Chunk?, <<Science>, 1974, 183, 482-488 (ESS)

@Simon, H (1973) Lessons from Perception for Chess-Playing Programs
(and vice versa), CMU Computer Science Research Review 1972-1973, pp.35-40\. 
(ESS I)

Simon, Herbert A., and Siklossy, Laurent, editors, <<Representation
and Meaning: Experiments with Information Processing Systems>,
Prentice-Hall, Englewood Cliffs, New Jersey, 1972, xx + 440 pages,
(collection of CMU dissertations each (SUM or DR))
See esp. Simon's "On Reasoning About Actions" (SUM,P)

Sloman, Aaron (1971) Interactions Between Philosophy and Artificial
Intelligence: The Role of Intuition and Non-Logical Reasoning in
Intelligence, <<Journal of AI>, 2, 1971, pp. 209-225\.  Provocative.
(ESS, P looking for philosophical implications of AI work)

Sloman, Aaron, <<The Computer Revolution in Philosophy>, Humanities
Press, 1979.

Smith, Brian C., Levels, Layers, and Planes: The Framework of a System of
Knowledge Representation Semantics,
unpublished Masters thesis, Dept. of E.E. & C.S.,
M.I.T., 1978\. (DR,I: new directions in representation and meaning)

@Smith, R.G., T.M.Mitchell, R.A. Chestek, and B.G.Buchanan,
A Model for Learning Systems, <<Proc IJCAI5>, 1977, 338-343 (SURV, P)

Soloway, Elliot M. and Edward M. Riseman,
Levels of Pattern Description in Learning, <<Proc IJCAI5>, 1977, 801-811 (DR,I)

Sussman, G., and D.V.McDermott,
From Planning to Conniving: A Genetic Approach, <<Proc ACM FJCC>, 1972 (SUM,
I:where Planner went wrong, and Conniver  doesn't)

Szolovits, P., L.B. Hawkinson, and W.A. Martin, An Overview of
OWL, an language for knowledge representation, M.I.T. LCS-TM-86,
1977\. (SUM)

Thomas,A.J. and T. O. Binford; 
"Information Processing Analysis of Visual Perception: A Review";
Stanford AI Lab Memo AIM-227, CS-408, 1974\. (SURV, ESS)

Vere, Steven A.,
Induction of Relational Productions in the Presence of Background Information,
<<Proc. IJCAI5>, 1977, 349-355 (DR,)

@Walker, Donald E., William H. Paxton, et al.,
Procedures for Integrating Knowledge in a Speech Understanding System, <<Proc IJCAI5>,
1977, 36-42 (SUM,I)

Waltz, David (ed.), <<TINLAP-2: Theoretical
Issues in Natural Language Processing, an interdisciplinary workshop
in computational linguistics, psychology, linguistics, and artificial
intelligence>, University of Illinois, June 1977.
(a collection of papers, mostly SUM)

Waterman, D., and F. Hayes-Roth, <<Pattern-Directed Inference Systems>, New York,
Academic Press, 1978 (a collection of papers, mostly SUM; good coverage of
state-of-the-art work in production systems) 
See esp. Duda et al; Hayes-Roth, Waterman, & Lenat.

Weiner, N.  <<The Human uses of Human Beings: Cybernetics and Society>,
Anchor 1954\. (ESS, P)

Weizenbaum, J., ELIZA, <<CACM>
1966, 9, 36-45.
(SUM, I:It's easy to pretend intelligence by reflective listening, or, more
generally, by very careful selection of the task to be performed)

@Weizenbaum, Joseph,
<<Computer Power and Human Reason>, San Francisco, W.H.Freeman, 1976
(ESS, I: we should be aware of the larger implications of our work)

Weyhrauch, R. Prolegomena to a theory of mechanized formal reasoning.
Stanford A.I. Memo, 1979 (DR,P)

Wickelgren, Wayne A.,  <<How to Solve Problems: Elements of a
Theory of Problems and Problem-solving>.  San Francisco:
W.H. Freeman and Company, 1974\.   Integrates Newell and Polya's ideas.
(DR, I:Reconciling Polya and one of his students, A. Newell)

Wilks, Y.,
Natural Language Understanding Systems within the AI Paradigm: A Survey and
Some Comparsions, AIM-237, Stanford U., 1974 (SURV,P)

@Winograd, Terry, "Five Lectures on Artificial Intelligence",
Stanford AIM-246, CS459, ADA000085/1WC, 93 pages, September 1974.
(ESS, P)

Winograd, Terry, <<Understanding Natural Language>, Academic
Press, Inc., New York, New York, 1972, viii + 191 pages, $10.00\. 
See paper in Schank and Colby or in 5 Lectures for summary.
(DR, I:Procedural knowledge in an integrated system).

Winograd, Terry, Frame representations and the declarative/procedural
controversy, in Bobrow & Collins (ed), <<Representation & Understanding>, 1975
(ESS, I:modularity of knowledge structures, frames)

Winograd, T.,
On some contested suppositions of generative linguistics about the scientific
study of language, <<Cognition>, 5, 1977 (reply to earlier article by Drescher
and Hornstein) (ESS,P)

Winograd, Terry, Towards a Procedural Understanding of Semantics,
<<Revue Internationale de Philosophie>, 1976 fasc. 3-4 (117-118). (ESS,I:
procedural semantics in linguistics)

Winston, P. H. (1972) The M.I.T. Robot, <<Machine Intelligence 7>,
American Elsevier Pub. 
(SURV, I:Heterarchical systems)

@Winston, Patrick H., editor, <<The Psychology of Computer Vision>,
McGraw-Hill Publishing Company, New York, New York, 1975, 
$18.00\.   Mostly MIT vision work. 
See Minsky's FRAMES paper, Shirai's, Waltz's, Winston's.
(Collection, each at least (DR/SUM, ))

@@Winston, P. and R. Brown, eds., <<Artificial Intelligence: An MIT Perspective>,
(2 volumes), M.I.T., 1979\.  Copies available from MIT, library, and TW.
This subsumes the 1974 memo "New Progress in Artificial Intelligence".
(SURV, Collection of summaries)

@Winston, Patrick H.,
<<Artificial Intelligence>, Reading: Addison-Wesley, 1977 (TEXT, P)
(one of the best overviews of the field)

Woodham, R.J.; "A Cooperative Algorithm for Determining Surface Orientation
from a Single View"; <<Proc IJCAI5>, 1977, 635-641\. (DR)

Woods, W. A. and Makhoul, J. (1973) Mechnical Inference Problems in
Continuous Speech Understanding. <<Proc IJCAI3>, pp.  200-207\.  This describes a
partly-implemented system. For the final story, see Woods' article in
IEEE Transactions on ASSP, February, 1975.
(SUM, I:Incremental simulation)

Woods, W., M. Bates, B. Bruce, and B.L.Nash-Webber,
Uses of Higher Level Knowledge in a Speech Understanding System, 
<<SIGART>, April 1976 (SUM,I)

Yakimovsky, Y. and Feldman, J. (1973) A Semantics-Based Decision
Theory Region Analyzer. <<Proc IJCAI3>, Advanced Papers pp 580-8\. 
(SUM, I:pruning using real-world constraints)

.endcrown

```