<!--yml
category: 未分类
date: 2024-05-27 15:00:32
-->

# A Few Words on Testing - by Thorsten Ball - Register Spill

> 来源：[https://registerspill.thorstenball.com/p/a-few-words-on-testing](https://registerspill.thorstenball.com/p/a-few-words-on-testing)

First, my credentials. More than half of all the code I wrote in my life is test code. My name is attached to [hundreds](https://interpreterbook.com/) of [pages](https://compilerbook.com/) of TDD. In my first internship as a software developer I wrote tests and did TDD while pair programming in my first week. I’ve written unit tests, integration tests, tests for exploration, tests to stop problems from reappearing, tests to leave a message, tests using testing frameworks and BDD and no framework at all, tests in Ruby, JavaScript, C, Go, Rust, Scheme, Bash. After two drinks, I’m willing to say that I know more about testing than many others. Right now — no drinks — I’m willing to say that I love testing and that writing tests has brought me a lot of joy.

Yet I can no longer say that I’m free of doubt. To stick with the theme: I’m much more sober about testing today than I was ten years ago. Recently, in the past few months, the doubts have grown.

Too many flaky tests. Too much time spent getting the tests to pass after making a tiny change that I knew was correct but the tests didn’t. Too many integration tests that made people wait 20, 30, 40 minutes until they could merge their change, only to reveal — months later — that they never tested anything. Too many times have I fixed a bug and *knew* it was fixed because I tested it manually, thoroughly, and was 100% sure that I know how the code works and that this can’t happen again, but then spent hours — 10 times longer than it took me to fix the bug — to write a test only to prove what I knew all along, that the bug is fixed.

It’s not that I was ever a capital-b Believer in tests. I never believed in testing coverage as a metric, never really cared whether someone wrote their tests first or last (although I think too few people have seriously tried TDD), came to think that most discussions around functional vs. intergration vs. whathaveyou tests are a big misunderstanding and that people who say you should never hit the database in tests should get real and probably haven’t written enough tests yet.

Still, I’ve always thought of tests as good and untested code as bad. Whenever I merged something that didn’t have a test I felt guilty, even when deep down I knew that the test might not be worth it. Tests, I thought, are a sign of quality and the better tested something is, the higher the quality of the product.

Enter [Ghostty](https://mitchellh.com/ghostty) and [Zed](https://zed.dev/).

Both are among the highest-quality software I have ever used and hacked on.

Both have less tests than I expected.

Both do have tests, of course. Ghostty has extensive tests for its core: the terminal state, the font rendering, the parser of escape and control sequences, and so on. Zed also a lot of tests for its foundational data structures — the rope, the SumTree, the editor, and so on — and tests for big features, and [very smart, very cool property tests for async code](https://www.youtube.com/watch?v=ms8zKpS_dZE).

But neither codebase has tests, for example, that take a long-ass time to run. No tests that click through the UI and screenshot and compare and hit the network. Zed’s complete testsuite takes 136 seconds to run 1052 tests in CI. Ghostty’s takes 38 seconds, including compilation. Many tests, but less than I thought.

In both codebases I’ve merged PRs without any tests and frequently see others do the same. And the world didn’t end and no one shed any tears and the products are still some of best I’ve ever used and the codebases contain some of the most elegant code I’ve ever read.

So now I’m writing this and it feels like a confession to say that I’m beginning to think that maybe there’s no correlation between software quality and tests. Maybe the tests are only a symptom. A symptom of something else that causes the quality.

Maybe the wisest thing about testing that’s ever been said and maybe the only thing that you need to know about testing — forget about the testing pyramids, and the mocks vs. stubs debates, and the dependency injectors, and the coverage numbers — is what Kent Beck said [16 years ago in an answer on Stack Exchange](https://stackoverflow.com/a/153565):

> I get paid for code that works, not for tests, so my philosophy is to test as little as possible to reach a given level of confidence […]. If I don’t typically make a kind of mistake (like setting the wrong variables in a constructor), I don’t test for it.

Maybe that’s all you need. That and people who give a damn.