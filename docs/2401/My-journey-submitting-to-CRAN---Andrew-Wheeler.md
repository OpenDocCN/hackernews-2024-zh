<!--yml
category: 未分类
date: 2024-05-27 15:05:21
-->

# My journey submitting to CRAN | Andrew Wheeler

> 来源：[https://andrewpwheeler.com/2022/07/22/my-journey-submitting-to-cran/](https://andrewpwheeler.com/2022/07/22/my-journey-submitting-to-cran/)

So my R package [ptools is up on CRAN](https://cran.r-project.org/package=ptools). CRAN obviously does an important service – I find the issues I had to deal with pedantic – but will detail my struggles here, mostly so others hopefully do not have to deal with the same issues in the future. Long story short I knew going in it can be tough and CRAN did not disappoint.

Initially I submitted the package in early June, which it passed the email verification, but did not receive any email back after that. I falsely presumed it was in manual review. After around a month I sent an email to cran-sysadmin. The CRAN sysadmin promptly sent an email back with the reason it auto-failed – examples took too long – but not sure why I did not receive an auto-message back (so it never got to the manual review stage). When I got auto-fail messages at the equivalent stage in later submissions, it was typically under an hour to get that stage auto-fail message back.

So then I went to fixing the examples that took too long (which on my personal machine all run in under 5 seconds, I have a windows $400 low end “gaming” desktop, with an extra $100 in RAM, so I am not running some supercomputer here as background). Running devtools `check()` is not the same as running `R CMD check --as-cran path\package.tar.gz`, but maybe `check_built()` is, I am not sure. So first note to self just use the typical command line tools and don’t be lazy with devtools.

Initially I commented out sections of the examples that I knew took too long. Upon manual review though, was told don’t do that and to wrap too long of examples in `donttest{}`. Stochastic changes in run times even made me fail a few times at this – some examples passed the time check in some runs but failed in others. Some examples that run pretty much instantly on my machine failed in under 10 seconds for windows builds on CRAN’s checks. (My examples use plots on occasion, and it may be `spplot` was the offender, as well as some of my functions that are not fast and use loops internally.) I have no advice here than to just always wrap plot functions in `donttest{}`, as well as anything too complicated for an abacus. There is no reliable way (that I can figure) to know examples that are very fast on my machine will take 10+ seconds on CRAN’s checks.

But doing all of these runs resulted in additional Notes in the description about spelling errors. At first it was last names in citations (Wheeler and Ratcliffe). So I took those citations out to prevent the Note. Later in manual review I was asked to put them back in. Occasionally a DOI check would fail as well, [although it is the correct DOI](https://onlinelibrary.wiley.com/doi/10.1002/jip.1449).

One of the things that is confusing to me – some of the Note’s cause automatic failures (examples too long) and others do not (spelling errors, DOI check). The end result messages to me are the same though (or at least I don’t know how to parse a “this is important” Note vs a “whatever not a big deal” Note). The irony of this back and forth related to these spelling/DOI notes in the description is that the description went through changes only to get back to what is was originally.

So at this point (somewhere around 10+ submission attempts), 7/16, it finally gets past the auto/human checks to the point it is uploaded to CRAN. Finished right – false! I then get an automated email from Brian Ripley/CRAN later that night saying it is up, but will be removed on 8/8 because `Namespace in Imports field not imported from: 'maptools'`.

One function had `requireNamespace("maptools")` to use the conversion functions in maptools to go between sp/spatspat objects. This caused that “final” note about maptools not being loaded. To fix this, I ended up just removing maptools dependency altogether, as using unexported functions, e.g. `maptools:::func` causes a note when I run `R CMD check` locally (so presume it will auto-fail). There is probably a smarter/more appropriate way to use imports – I default though to doing something I hope will pass the CRAN checks though.

I am not sure why this namespace is deal breaker at this stage (after already on CRAN) and not earlier stages. Again this is another Note, not a warning/error. But sufficient to get CRAN to remove my package in a few weeks if I don’t fix. This email does not have the option “send email if a false positive”.

When resubmitting after doing my fixes, I then got a new error for the same package version (because it technically is on CRAN at this point), so I guess I needed to increment to 1.0.1 and not fix the 1.0.0 package at this point. Also now the DOI issue in the description causes a “warning”. So I am not sure if this update failed because of package version (which doesn’t say note or warning in the auto-fail email) or because of DOI failure (which again is now a warning, not a Note).

Why sometimes a DOI failure is a warning and other times it is a note I do not know. At some later stage I just take this offending DOI out (against the prior manual review), as it can cause auto-failures (all cites are in the examples/docs as well).

OK, so package version incremented and namespace error fixed. Now in manual review for the 1.0.1 version, get a note back to fix my errors – one of *my tests* fails on noLD/M1Mac (what is noLD you may ask? It is “no long doubles”). These technically failed on prior as well, but I thought I just needed to pass 2+ OS’s to get on CRAN. I send an email to Uwe Ligges at this point (as he sent an email about errors in prior 1.0.0 versions at well) to get clarity about what exactly they care about (since the reason I started round 2 was because of the Namespace threat, not the test errors on Macs/noLD). Uwe responds very fast they care about my test that fails, not the DOI/namespace junk.

So in some of my exact tests I have checks along the line `ref <- c(0.25,0.58); act <- round(f,2)` where `f` is the results scooped up from my prior function calls. The note rounds the results to the first digit, e.g. `0.2 0.5` in the failure (I suspect this is some behavior for `testhat` in terms of what is printed to the console for the error, but I don’t know how exactly to fix the function calls so no doubles will work). I just admit defeat and comment out the part of this test function that I think is causing the failure, any solution I am not personally going to be able to test in my setup to see if it works. Caveat Emptor, be aware my exact test *power* calculation functions are not so good if you are on a machine that can’t have long doubles (or M1 Mac’s I guess, I don’t fricken know).

OK, so that one test fixed, upon resubmission (the following day) I get a *new* error in my tests (now on Windows) – `Error in sp::CRS(...): PROJ4 argument-value pairs must begin with +`. I have no clue why this is showing an error now, for the first time going on close to 20 submissions over the past month and a half.

The projection string I pass definitely has a “+” at the front – I don’t know and subsequent submissions to CRAN even after my attempts to fix (submitting projections with simpler epsg codes) continue to fail now. I give up and just remove that particular test.

Uwe sends an updated email in manual review, asking why I removed the tests and did not fix them (or fix my code). I go into great detail about the new SP error (that I don’t think is my issue), and that I don’t know the root cause of the noLD/Mac error (and I won’t be able to debug before 8/8), that the code has pretty good test coverage (those functions pass the other tests for noLD/Mac, just one), and ask for his grace to upload. He says OK patch is going to CRAN. It has been 24 hours since then, so cannot say for sure I will not get a ‘will be removed’ auto-email.

To be clear these issues back and forth are on me (I am sure the `\donttest{}` note was somewhere [in online documentation](https://r-pkgs.org/) that I should have known). About the only legit complaint I have in the process is that the “Note” failure carries with it some ambiguity – some notes are deal breakers and others aren’t. I suspect this is because many legacy packages fail these stringent of checks though, so they need to not auto-fail and have some discretion.

The noLD errors make me question reality itself – does `0.25 = 0.2` according to M1 Mac’s? Have I been living a lie my whole life? Do I **really** know my code works? I will eventually need to spin up a Docker image and try to replicate the noLD environment to know what is going on with that one exact test power function.

For the projection errors, I haven’t travelled much recently – does Long Island still exist? Is the earth no longer an ellipsoid? At our core are we just binary bits flipping the neural networks of our brain – am I no better than the machine?

There is an irony here that people with better test code coverage are more likely to fail the auto-checks (although those packages are also more likely to be correct!). It is intended and reasonable behavior from CRAN, but it puts a very large burden on the developer (it is not easy to debug noLD behavior on your own, and M1 Mac’s are effectively impossible unless you wish to pony up the cash for one).

* * *

CRAN’s model is much different than python’s PyPI, in that I could submit something to PyPI that won’t install at all, or will install but cause instant errors when running `import mypackage`. CRANs approach is more thorough, but as I attest to above is quite a bit on the pedantic side (there are no “functional” changes to my code in the last month I went through the back and forth).

The main thing I really care about in a package repository is that it does not have malicious code that does suspicious `os` calls and/or sends suspicious things over the internet. It is on me to verify the integrity of the code in the end (even if the examples work it doesn’t mean the code is correct, I have come across a few packages on R that have functions that are obviously wrong/misleading). This isn’t an open vs closed source thing – you need to verify/sanity check some things work as expected on your own no matter what.

So I am on the fence whether CRAN’s excessive checking is “worth it” or not. Ultimately since you can do:

```
library(devtools)
install_github("apwheele/ptools")
```

Maybe it does not matter in the end. And you can peruse the github actions to see the current state of whether it runs on different operating systems and avoid CRAN altogether.