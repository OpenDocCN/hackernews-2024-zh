<!--yml
category: 未分类
date: 2024-05-27 15:20:41
-->

# Chris Date and the Relational Model - Simple Talk

> 来源：[https://www.red-gate.com/simple-talk/opinion/opinion-pieces/chris-date-and-the-relational-model/](https://www.red-gate.com/simple-talk/opinion/opinion-pieces/chris-date-and-the-relational-model/)

Chris Date has long realized that his aim to convert everyone to his way of thinking will not prove harmonious. It is forty-five years this year since Date’s mentor and colleague Ted Codd, a former Royal Air Force pilot, Oxford educated mathematician and winner of the Turing Award, invented the relational model for database management and transformed the humble database.

 Looking back it seems such a simple idea to replace the hierarchical structures used to build databases with tables of rows and columns but Codd worked in a different world, when you had to deal with a huge amount of complexity to build a functioning database.

In 1970 Ted Codd published A Relational Model of Data for Large Shared Data Banks. It ushered in a global database market which in 2013 was estimated to be worth $28 billion and it is growing at approximately 7.5 per cent a year.

Codd’s paper was hugely influential to those working in the database field though a little more directly on Chris Date than a lot of other people.

Since his first meeting with Codd, Chris has become one of the leading voices specializing in relational database technology.

RM:

Chris, can you talk about the influence of Ted Codd on your work?

CJD:

Before I respond to this first question, I simply can’t resist the temptation to note that in my world, “RM” always stands for the relational model! So I’m truly honored to be interviewed by someone with such auspicious initials.

Anyway, what was Ted’s influence on my work? Well, of course it was huge. Let me explain how I got into this business. My background is in mathematics. When I left university, the only job that seemed to be available to mathematicians at the time that didn’t sound totally boring-I recall one option was *time and motion study*; can you imagine?-was this new thing called computing.

 (This was back in 1962, and we certainly didn’t have any computer science in our curriculum in those days; very few people even knew what a computer was.)

 So I became a computer programmer. And that was fun and interesting for a while, but quite frankly it didn’t seem to have much to do with the mathematics I learned at Cambridge.

And then, in 1970, two things happened: First, a colleague and myself were given the job of figuring out what the language PL/I should do about another new thing, namely databases (I was working for the IBM Hursley lab by this time, which was the home of PL/I).

“I got in touch with Ted,
and we became friends
and colleagues

Second, Ted Codd’s famous paper was published (“A Relational Model of Data for Large Shared Data Banks,” *CACM 13*, No. 6, June 1970). So I read that paper-in among a lot of other database stuff, I should quickly add-and I discovered it was all about set theory and logic, stuff I did know and love from my time at Cambridge.

 So at last I had found an aspect of computing, database technology, where my mathematical background was relevant. I got in touch with Ted, and we became friends and colleagues-first within IBM, later in a consulting company that we started, along with Sharon Weinberg, in 1985 or so. And I’ve been working in that field (database technology, that is) all through that time, for almost 45 years now.

To get back to your question, I don’t think I can do better than quote from the tribute piece I wrote for Ted when he died, back in 2003\. (Of course, it’s very conceited to quote from oneself, but in the circumstances I hope you’ll forgive me.)

Dr. Codd, known universally to his colleagues and friends-among whom I was proud to count myself-as Ted, was the man who, singlehanded, put the field of database management on a solid scientific footing. The entire relational database industry, now worth many billions of dollars a year, owes the fact of its existence to Ted’s original work, and the same is true of all of the huge number of relational database research and teaching programs under way worldwide in universities and similar organizations.

Indeed, all of us who work in this field owe our career and livelihood to the giant contributions Ted made during the period from the late 1960s to the early 1980s. We all owe him a huge debt. This tribute to Ted and his achievements is offered in recognition of that debt.

In other words, Ted’s influence on me was, as I’ve said, huge, but it wasn’t just me-it was all of us in the database field. Though perhaps it was a little more directly on me than it was on a lot of other people.

“Ted and I certainly
didn’t see eye to
eye on everything
-we did have our
disagreements

That said, I should say too that Ted and I certainly didn’t see eye to eye on everything-we did have our disagreements. I say this because the idea seems to exist in certain circles that Ted’s writings are and must forever be the final arbiter on all things relational. (“This must be the way it is because it’s what Ted Codd said.”) I think such an attitude is quite dangerous.

Science is a social activity; it’s also an activity that’s always in an ongoing state of development; and I don’t think scientists in general-true scientists, that is-would ever claim that their own opinion is the absolute last word on any scientific subject. Critical thinking is just as important in the database field as it is everywhere else.

RM:

Do you think we’re taken a huge step backwards in terms of how SQL has developed? Do you think people have the intellectual curiosity about where the idea of relational databases came from? Are people aware of the history?

CJD:

Oh dear. There’s so much I want to say here! Let me see if I can put it all into some kind of sensible order. Perhaps the first item to address is: Why did SQL happen at all? I mean, why was it developed in the first place?   

The answer to this question can be found, I think, in the very first SQL paper (Donald D. Chamberlin and Raymond F. Boyce, “SEQUEL: A Structured English Query Language,” Proc. 1974 ACM SIGMOD Workshop on Data Description, Access, and Control, Ann Arbor, Mich., May 1974; the name SEQUEL was subsequently changed to SQL for legal reasons).

In that paper, the authors claimed, in effect, that their language was more user friendly than predicate calculus, and in particular that it didn’t need what they called “the mathematical sophistication” of predicate calculus.

 You see, Ted’s 1970 paper had proposed, among other things, that predicate calculus might reasonably be used as a basis for a data language (or data sublanguage, as Ted called it), and it gave examples of what expressions in such a language might look like.

Unfortunately, it did so using the formal notation of predicate calculus, notation that many people in the computing world at the time-now too, probably-were quite unfamiliar with. So the idea arose that predicate calculus was just too difficult for ordinary mortals, and something simpler was needed. Ergo, SQL.

Now I think this was all a confusion on the part of the SQL folks. It’s true that the notation-the syntax-of predicate calculus might look a little strange and complicated; but the concepts expressed by that notation-the semantics-are actually quite simple, and can be explained quite simply too. In other words, it’s not difficult to wrap some nice syntactic sugar around those concepts and make them very palatable indeed.

“QBE, like so many
other proposals,
was subsequently
overwhelmed by
the SQL juggernaut

Do you happen to be familiar with Query-By-Example (QBE)? QBE is exactly predicate calculus! In my opinion, it’s a very user friendly syntactic sugar coating of the semantic ideas embodied in predicate calculus. (Of course, QBE, like so many other proposals, was subsequently overwhelmed by the SQL juggernaut. I know one of your later questions has to do with such matters, so we’ll come back to this issue later.)

Anyway, the SQL folks didn’t do that (I mean, they didn’t do what QBE did)-instead, they invented something different, based on what they called a mapping (basically, the ability to use a subquery in the WHERE clause). I don’t want to get into details of those “mappings” here; I just want to note that they’re actually 100% redundant!

By which I mean, there’s nothing relational you can do using them that you can’t also do without them. It’s ironic, really-the single biggest innovation in the SQL language could actually be thrown away without any serious loss of function.

What’s more, I really question whether SQL is more user friendly than predicate calculus, anyway. In fact I think it’s what might reasonably be described as user hostile … In my seminars (which are mostly intended these days for SQL professionals specifically) I make a point of showing a set of SQL queries, or would-be queries, and asking the class which ones are legal and which illegal. And *nobody* gets them all right the first time around.

“I really question whether
SQL is more user friendly
than predicate calculus

As a matter of fact I can never remember the answers myself either, because the pertinent syntax rules are just so convoluted and intertangled. You know, it’s very easy to make something complicated; it’s very hard to make something simple. (Parenthetically, part of the beauty of the relational model is precisely that it’s fundamentally so simple.)

There’s more, a lot more. SQL isn’t just user hostile, it involves some very serious departures from relational theory. I don’t think this is the place to get into specifics-I’ve written about those problems at great length elsewhere (as indeed other people have too, including in particular my friend and colleague Hugh Darwen).

Suffice it to say that those departures are so serious that I honestly believe SQL has no real right to be called relational at all. As a consequence, SQL DBMSs have no real right to be called relational at all, either. The truth is, there never has been a mainstream DBMS product that’s truly relational.

“I honestly believe
SQL has no real
right to be called
relational at all

I honestly believe SQL has no real right to be called relational at all

There’s something else I’d like to say in connection with all this. People who don’t accept our criticisms of SQL often describe the points on which we disagree as “religious issues”-implying, I suppose, that a proper resolution of those issues is a matter of faith, not science.

Our position is exactly the opposite: Proper resolution is a matter of science, not mere faith. For example, take nulls. The simple *scientific* fact is that an SQL table that contains a null isn’t a relation; thus, relational theory doesn’t apply, and all bets are off. (Of course there’s a lot more I could say here, but for present purposes I think this should be enough.

Well, I’ll just say one thing: If you’d prefer to replace the words *science* and *scientific* in the foregoing by *logic* and *logical*, respectively, then I won’t argue with you. I mean, I don’t think it makes any difference.)

You also ask whether people are aware of the history here, and whether they have the intellectual curiosity to want to be aware. Well, I don’t think I should generalize too much here. At least I can say that the people who attend my seminars do seem to take an interest in this stuff.

On the other hand, we live for better or worse in a world where the market is king, and a lot of people seem to take it as an article of faith that if something’s new it must be better than what we had before: a position that seems to me to demonstrate a considerable *lack* of historical perspective, or interest. In connection with this point, I note that Lou Gerstner-IBM chairman at the time-was once quoted in *Informationweek* (February 9th, 1998) as saying “All large companies know today that speed and being early to market are often more important than being right.” Make of that what you will.  

RM:

What was key to SQL becoming the standard language for relational databases in the mid- 1980s? Was all down to good marketing?  

CJD:

In other words, why did SQL became so popular? Especially given all its faults? Well, I think this is rather a sorry story. I said earlier that there has never been a mainstream DBMS product that’s truly relational. So the obvious question is: Why not? And I think a good way for me to answer your questions here is to have a go at answering this latter question in their place, which I’ll do by means of a kind of Q&A dialog. Like this:

Q:

Why has no truly relational DBMS has ever been widely available in the marketplace?

A:

Because SQL gained a stranglehold very early on, and SQL isn’t relational.

Q:

Why does SQL have such a stranglehold?

A:

Because SQL is “the standard language for RDBMSs.”

Q:

Why did the standard endorse SQL as such and not something else-something better?

A:

Because IBM endorsed SQL originally, when it decided to build what became DB2\. IBM used to be more of a force in the marketplace than it is today. One effect of that state of affairs was that-in what might be seen as a self-fulfilling prophecy-competitors (most especially Relational Software Inc., which later became Oracle Corp.) simply assumed that SQL was going to become a big deal in the marketplace, and so they jumped on the SQL bandwagon very early on, with the consequence that SQL became a kind of de facto standard anyway.

Q:

Why did DB2 support SQL?

A:

Because (a) IBM Research had running code for an SQL prototype called System R and (b) the people in IBM management who made the decision to use System R as a basis on which to build DB2 didn’t understand that there’s all the difference in the world between a running prototype and an industrial strength product. They also, in my opinion, didn’t understand software (they certainly didn’t understand programming languages). They thought they had a bird in the hand.

Q:

Why did the System R prototype support SQL?

A:

My memory might be deficient here, but it’s my recollection that the System R implementers were interested primarily in showing that a relational-or “relational”-DBMS could achieve reasonable performance (recall that “relational will never perform” was a widely held mantra at the time). They weren’t so interested in the form or quality of the user interface. In fact, some of them, at least, freely admitted that they weren’t language designers as such. I’m pretty sure they weren’t all totally committed to SQL specifically. (On the other hand, it’s true that at least one of the original SQL language designers was a key player in the System R team.)

Q:

Why didn’t “the true relational fan club” in IBM-Ted and yourself in particular-make more fuss about SQL’s deficiencies at the time, when the DB2 decision was made?

A:

We did make some fuss but not enough. The fact is, we were so relieved that IBM had finally agreed to build a relational-or would-be relational-product that we didn’t want to rock the boat too much. At the same time, I have to say too that we didn’t realize how truly awful SQL was or would turn out to be (note that it’s much worse now than it was then, though it was pretty bad right from the outset). But I’m afraid I have to agree, somewhat, with the criticism that’s implicit in the question; that is, I think I have to admit that the present mess is partly my fault.

“Certainly I did my bit
in the early days to
help push SQL on to
an unsuspecting public

Certainly I did my bit in the early days to help push SQL on to an unsuspecting public. In my defense, let me say that the battles inside IBM to get management to commit to building a product at all were both bloody and exhausting, and I don’t think we had much energy left to fight the battle that said “Well, we’re glad you’ve agreed to build a product, but don’t you see you’re still not doing it right.” As I’ve said, we made some fuss but not enough, and in fact I don’t think the people who were making the decisions at that time cared very much what we thought, anyway. (I could cite some concrete examples here but probably shouldn’t.)

Anyway, you see what I mean by a sorry story. What makes it even sorrier is that, even at the time, a vastly preferable alternative to SQL existed *inside IBM!* I’m referring here to ISBL (“Information Systems Base Language”), which was developed by Stephen Todd and others at the IBM Peterlee Scientific Centre in England (P. A. V. Hall, P. Hitchcock, and S. J. P. Todd, “An Algebra of Relations for Machine Computation,” Conf. Record of the 2nd ACM Symposium on Principles of Programming Languages, Palo Alto, Calif., January 1975).

The trouble was, California was the center of the world for database work at the time. Certainly all mainstream database work in IBM was being done in California. Thus, through no fault of their own, the work being done by the ISBL team was marginalized at best, if not overlooked entirely, by the IBM powers that be. I don’t think it’s an exaggeration to say that the database community in particular, and in fact society at large, are still paying the price for this state of affairs.

RM:

You’ve said that Ted was a genius but he wasn’t too good at communicating his ideas to ordinary mortals, so you took over this area. Your writing style is very coherent. Did you learn to write in this style or do you write as you talk?

CJD:

I didn’t exactly “take over this area”!-it would be more accurate to say it fell into my lap. So much always depends on happenstance, doesn’t it-the chance, or good luck, of being in the right place at the right time. I’ve explained how I first came across Ted’s ideas in 1970, and I’ve said, or at least implied, that they seemed right to me because of their mathematical foundation. I’ve also talked about how we (my colleague in IBM Hursley and myself) were thinking about how to incorporate Ted’s ideas into PL/I. What I didn’t say was that Ted at the time was fighting very much of a lone battle in IBM; he was up against some rather powerful vested interests.

“Ted at the time was fighting very
much of a lone battle in IBM;
he was up against some rather
powerful vested interests.

 So in order to show that he wasn’t completely alone, Ted invited IBM Hursley to send someone out to California, where he was, to talk to various people about what we were doing. And that was me. And I flatter myself that my presentations went down well-I mean, I was able to make people understand what we were doing-and I began to get more requests to talk about relational technology, both to IBM folks and to IBM customers (and to technical audiences outside IBM as well, come to that). Then I got an IBM assignment to California (this was in 1974) and, well, one thing led to another, and Ted and I found ourselves increasingly being seen as a team.

Thank you for saying my writing style is “coherent”! But I think I can explain that, too. The truth is, I’m a rather slow learner. As a consequence, I think I can be a good teacher, because I can identify places where students are likely to have trouble. The truth is, all of my books are essentially written for me! What I mean is, I try to write the book I would have liked to have read when I was learning the subject at hand myself. But I’ve never taken any formal lessons in how to write. (It’s relevant to add that I’ve always loved books anyway, and I’ve always been interested in words, and language, and languages. Not just or even primarily computer books, I hasten to add; and not just or even primarily computer languages, either.)

As for “Do I talk the same way?” well, I certainly try to when I’m teaching a class. Probably not, in unprepared conversation! In fact, certainly not. I need time to get my thoughts in order. So I’m often unable to answer a question coherently right away, for example. But then again, I tend to be rather suspicious of people who have an immediate opinion on everything and are never at a loss for words.

RM:

There’s a common application language in D. Why, in your view isn’t this more widely used? Is there a possibility that in say 5 years, SQL will become what COBOL is to programmers, very much the last resort? If SQL is demoted to a language of last resort what will have been its legacy?

CJD:

First let me correct a misconception. **D**-the name is always set in bold face, by the way-isn’t a language; rather, it’s a generic name for any language that conforms to the principles laid down by *The Third Manifesto*. *The Third Manifesto* in turn is a set of prescriptions for future data and database management systems; it consists essentially of a precise though somewhat terse definition of the relational model and a supporting type theory (including a comprehensive model of type inheritance). It’s described in detail in the book *Databases, Types, and the Relational Model: The Third Manifesto* (3rd edition), by Hugh Darwen and myself (Addison-Wesley, 2007). Now, Hugh and I needed a generic name for a language that we could reference in the prescriptions of the *Manifesto*, just so that we could save ourselves a great deal of circumlocution in writing those prescriptions, and for reasons of our own we chose the name **D**. Any number of distinct languages could qualify as a valid **D**, though it goes without saying that SQL unfortunately isn’t one of them.

Now, we also needed a concrete language that we could use in the *Manifesto* book in examples and so forth, and so we invented a concrete **D** that we called, for obvious reasons, **Tutorial D**. (**Tutorial D** is, of course, a valid **D**; in fact, it was expressly designed to be suitable as a vehicle for illustrating and teaching the ideas of *The Third Manifesto*, though Hugh and I have since used it as the basis for examples, and the like, in numerous other books and papers and presentations.) And implementations of **Tutorial D**, or something very close to it, do exist, though not in product form. One in particular I’d like to mention is *Rel*, by Dave Voorhis, which you can access by going to the website *www.thethirdmanifesto.com*.

So now let me reinterpret your question: Why isn’t **Tutorial D** more widely used? Well, I think I’ve already answered that when I talked earlier about the SQL “stranglehold.” But it’s also true that there’s something you might call “the *Third Manifesto* community,” which consists of people all over the world-I’m afraid I don’t really have any idea how many-who do indeed use **Tutorial D** or something very like it for various practical and theoretical purposes.

“Tutorial **D** is, in a sense,
only a toy language …
it has no I/O, and it has
no exception handling

That said, I should also explain that **Tutorial D** is, in a sense, only a toy language. To be specific, it has no I/O, and it has no exception handling. On the other hand, it *is* well designed, as far as it goes, and one of the things we’d like to do (in among a very long list of such things!) is beef it up to become what you might call **Industrial D**. And yes, we’d like **Industrial D** to be considered-under a much better name, I would assume!-for implementation in an industrial context. (As a matter of fact, we’ve already given some thought to the question of implementing **Tutorial D**, at least, on top of SQL.) But even if **Industrial D** is implemented in product form, will it displace SQL? Well, obviously not for many years, if ever (certainly more than five years, I’m quite sure of that). But we’re optimists; we’re in this for the long haul; we take the view that if we do nothing, then nothing will happen, but if we do something, then something might happen.

RM:

The World Wide Web Consortium (W3C) enthuses about XQuery saying it’s replacing proprietary languages, is much simpler to work with and easier to maintain than many other alternatives. Is XQuery a better alternative to SQL?

RM:

No. But you probably wanted more than that, so I’ll give it a go. First of all, Xquery is designed to work with XML data, of course. Now, I haven’t taken a close look at either XML or XQuery for some years now, so the following remarks might no longer be totally applicable. If they aren’t, then I apologize. But here’s what I believe. First of all, XML structures are fundamentally hierarchic; thus, all of the intrinsic difficulties with hierarchies that we experienced all those years ago-with IBM’s IMS product in particular-are rearing their ugly head again. (I often characterize XML, perhaps a little unfairly, as “IMS warmed over.”) At least two serious problems arise immediately. First, the innate lack of symmetry in hierarchies means the structure might be suitable for some problems but will certainly be unsuitable for others. Second, many to many relationships will have to be dealt with in some ad hoc and probably unsatisfactory fashion.

“XQuery is necessarily
much more complicated
than a relational language

In fact, XML structures aren’t just hierarchies, they’re linearizable hierarchies-there’s a linear sequence to the nodes (I think I’m right in saying it’s top down, left to right sequence).   As a consequence, when W3C tried to define an XML algebra (allegedly as a basis for XQuery, though I think it was defined after XQuery was defined), they had to build on the sequence abstraction. But sequences aren’t a *good* abstraction! I mean, they’re unnecessarily complicated. Sets (which the relational model is based on) are so much better. One immediate consequence is that XQuery is necessarily much more complicated than a relational language. In particular, it includes support for iteration, which relational languages don’t need, or have.

To sum up: I’m obviously no fan of SQL, but no, I don’t think XQuery is any better. In fact, I think it suffers from some of the same problems that SQL does. At least SQL, with all of its faults, can be used-with a *lot* of discipline, like avoiding duplicates and nulls-almost as if it were relational; but the same clearly can’t be said of XQuery.

RM:

Can you identify any big changes in the way you think about relations and relational databases since your early days? Do you still enjoy your work as much as you used to when you first started?

CJD:

“A database, in order to be
a truly general purpose
database within the meaning
of the act, must be relational

In answer to your first question, yes. One big change is that in the early days, when we were still trying to convince people that relational was real (or at least could be), we-and especially I-necessarily got into the business of comparing the relational model with the systems (primarily hierarchic and network systems) that existed at the time. Typically, we would take sample problems and show how and why the relational solutions were so much better than the hierarchic and network solutions. But in doing that, we unwittingly did ourselves and the cause a huge disservice: We effectively spread the idea, or perception, that relations were “the same kind of animal” as hierarchies and networks, and in particular by the idea that even if relations did replace hierarchies and networks as we said they would, so relations in turn would sometime be replaced by something better. But I’ve come to realize that that perception was very wrong. I now believe-and I’ve explained this belief in detail in writing, as well as in my live lectures-that a database, in order to be a truly general purpose database within the meaning of the act, *must* be relational.  Relational really is different.

I’ve lost count of the number of times someone has come up to me and said “I’ve got something that’s better than the relational model.” (For a good example of the kind of thing I mean, take a look at the letters column in the January 2013 issue of *CACM*. I might also mention the truly appalling preamble to a book I read recently extolling the virtues and “advantages” of graph databases. And look at all the nonsense we were hearing a few years ago about how object databases were going to take over the world. At least we don’t hear so much about this last one any more, do we?) It seems to me that if you want to replace Technology A by Technology B, then it’s incumbent on you, first, to understand Technology A properly; second, to show that Technology B can solve all of the problems that Technology A solves; and third, to show there’s some problem that Technology B solves and Technology A doesn’t. And I can state categorically that, in my experience, people always fail at the first of these hurdles: They don’t understand the relational model in the first place. In fact, I’d like to offer a challenge: Show me one problem the relational model ought to be able to solve but can’t.

In connection with the foregoing, I’d like to mention one of my current research interests, temporal data. There’s been a lot of research into this subject over the years, and a lot of solutions proposed-but most of those solutions are nonrelational. I don’t want to get into details here of just exactly how they’re nonrelational; let me just say that the researchers often don’t seem to realize what they’re proposing is indeed nonrelational, because their papers have titles like “Temporal Extensions to the Relational Model,” or “Adding the Time Dimension to the Relational Model.” I believe this is more evidence of the general lack of understanding of the relational model in the database community at large. In strong contrast to the foregoing, I’m very pleased to be able to tell you that the relational model actually needs no extension, and no subsumption, and above all no perversion, in order to incorporate temporal support.  In fact, Hugh Darwen, Nikos Lorentzos, and I have just (July 2014) published a book that clearly demonstrates this very point (*Time and Relational Theory: Temporal Data in the Relational Model and SQL*, Morgan Kaufmann, 2014).

I have another reason for answering yes to your question, too (“Have I changed the way I think about relations and relational databases since the early days?”)-and here I must credit several people: my friends David McGoveran, Hugh Darwen again, and a late colleague Adrian Larner. Together, these friends and colleagues made me realize that the best way to think of a relation is not just as an abstraction of the classical concept of a file (with its records and fields), but rather as the extension of a predicate. Once again, I don’t think this is the place to get into details-I’ll just to have to point you to any of my recent writings (e.g., the book on temporal databases just mentioned), where this perception is explained in depth (and the reason why it’s important is explained in depth, too).

By the way, as a kind of PS to both of the foregoing responses, I’d like to mention that the SQL standard does now include some support for temporal data. But it seems to us (i.e., my coauthors and myself) that the support in question involves yet another rather large violation of relational principles. To be more specific: Suppose we create a table EMP, with columns EMPNO, EFROM, and ETO, with an intended interpretation along the lines of *Employee ENO was employed from date EFROM to date ETO*. Then it turns out, in general, that-despite what I just said about “intended interpretation”-that table actually has no predicate! At least, we haven’t been able to find a predicate that works. (Maybe there is one, but as I say we haven’t found it, and I can assure you it certainly wasn’t for want of trying.)   I don’t know what the consequences of this departure from relational theory are likely to be, but as far as I’m concerned it looks like yet another nail in the “SQL is relational” coffin.

“The relational model, although
it’s fundamentally so simple,
has amazing depths

Turning to your second question (“Do you still enjoy your work as much as you used to when you first started?”), my answer here is yes as well. In many ways, in fact, I enjoy it more, especially since I now work for myself instead of for some corporation. But the real point is this: The relational model, although it’s fundamentally so simple (you can explain the basic idea in five minutes), has amazing depths. You can peel away layer after layer of meaning and keep on discovering more and more. Thus, I’m still learning things about it myself. (Indeed, if I wasn’t, I would probably have given up this career years ago and become a park ranger or something.)

RM:

Do you feel there were times in your life where your passion for work ran contrary or to the detriment of other parts of your life?

CJD:

Well, I think I need to dispute the premise of your question, slightly. I do enjoy my work, very much; it’s challenging and creative and (I believe) useful. But I don’t live to work, I work to live. I have absolutely no difficulty in switching off work-related thinking when (for example) I’m hiking the desert-which, parenthetically, is something I really enjoy doing. I have many interests outside of work: birdwatching, wildlife in general, music, politics, hiking, poetry, beer and wine, cosmology, art, science fiction, and on and on. Work is one of my interests, but it’s not number one in the list. (I don’t think I *have* a number one.)

RM:

Do you consider yourself a scientist, an engineer, a writer or a craftsman?

CJD:

Scientist: Yes (at least if you consider mathematics a science; I know some people don’t, but I do). Engineer: No. Writer: Yes (though I might prefer “teacher”). Craftsman: Don’t know! I’m not sure what you’re getting at here.

RM:

Do you have any recommendations for people who want to follow in your footsteps?

CJD:

I don’t know about “following in my footsteps,” but if you’re talking about people who want to work in the relational field-let me stress that *relational*, though!-then, well, I don’t think I can do better than paraphrase some remarks I made in another recent interview. First, if people are serious about going into this field, then I congratulate them; the subject is intellectually stimulating, and pragmatically important, and it can be *fun*.  As for advice, one thing I can say is this (and I’m sorry if this sounds a little self-serving-I don’t mean it to be):

  1\. Learn the relational model, by reading the right books and/or attending the right courses.  

  2\. Go out and get your hands dirty working on a real project for a year or three.

  3\. Come back and read those books and/or attend those courses again.

“Learn the relational
model first,SQL second

A related piece of advice is:  Learn the relational model first, SQL second (doing it the other way around is *hard*).  Finally, read either or both of Ted Codd’s first two papers every year. (Those papers are “Derivability, Redundancy, and Consistency of Relations Stored in Large Data Banks”, IBM Research Report RJ599, August 19th, 1969, reprinted in *ACM SIGMOD Record 38*, No. 1 (March 2009), and “A Relational Model of Data for Large Shared Data Banks”, *CACM 13*, No. 6, June 1970, which you can find online.)

RM:

Is there anything I haven’t asked about your work that you thought I might?

CJD:

This is a dangerously open ended question! But I think I’ve been quite self-indulgent enough already, about quite enough different topics, so I think I’d better just stop here. Thank you for the opportunity to air my opinions.