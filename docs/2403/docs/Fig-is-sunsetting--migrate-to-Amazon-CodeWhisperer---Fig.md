<!--yml
category: 未分类
date: 2024-05-27 14:51:41
-->

# Fig is sunsetting, migrate to Amazon CodeWhisperer | Fig

> 来源：[https://fig.io/blog/post/fig-is-sunsetting](https://fig.io/blog/post/fig-is-sunsetting)

Happy 2024! Last year was a big year for our team: we joined Amazon in August, and by November, we had already shipped [Amazon CodeWhisperer for command line](https://docs.aws.amazon.com/codewhisperer/latest/userguide/command-line-getting-started-installing.html).

We’re keenly aware of the impact generative AI is having on the way developers build. We’re hard at work innovating and we have a lot of exciting new things coming to CodeWhisperer for command line later this year. With that said, we have decided to fully focus our efforts on these new features, and given that, we will be sunsetting Fig on September 1, 2024.

We will continue supporting Fig until September 1\. Until then, we encourage users to migrate over to [CodeWhisperer for command line](https://docs.aws.amazon.com/codewhisperer/latest/userguide/command-line-getting-started-installing.html). To make this transition as easy as possible, users can upgrade to CodeWhisperer for command line directly from the Fig dashboard (more below).

[CodeWhisperer for command line](https://docs.aws.amazon.com/codewhisperer/latest/userguide/command-line-getting-started-installing.html) offers Fig’s core features including 1) IDE-style completions for 500+ CLIs and 2) natural language-to-bash translation. It’s free on the Individual tier and is designed to be faster and more reliable than Fig. We have several exciting new CodeWhisperer for command line features coming in 2024 including support for Linux, AI chat, inline AI completions, autocomplete in SSH, and private autocomplete.

CodeWhisperer for command line does not include features similar to Fig Scripts, Dotfiles, Plugins, or Servers and we do not plan to support these features any time soon.

The Fig Dashboard UI will handle the migration for you — it takes just a few minutes. The migration process exports your data to your local device, installs CodeWhisperer for command line, transfers your settings, then uninstalls Fig. Once the CodeWhisperer app is installed, you will be prompted to grant macOS accessibility permissions and then login or sign up.

**Note**: You will need to be on v2.18 to see the new Fig Dashboard UI.

Fig will delete all user data on September 1\. Users can export their data to their local device any time before then by running `fig export`. The Fig to CodeWhisperer migration flow will automatically do this for users.

Yes! Before Fig is sunset you can use Fig and CodeWhisperer at the same time. When both tools are installed, Fig’s Autocomplete feature will turn off in favor of CodeWhisperer for command line. This means you can keep using Fig for Scripts, Dotfiles, Plugins, and Servers until September 1st but start using CodeWhisperer for command line for its more reliable CLI completions and AI straight away.

If you have any questions about the migration, please just create a discussion in our [public community](https://github.com/withfig/fig/issues/new?assignees=&labels=&projects=&template=1_general_question.md).

To all Fig’s users, customers, and contributors, we are incredibly grateful for your feedback, contributions, and support over the years. We are thrilled to have made such an impact and we are beyond excited to continue working with you all while we continue to ship!

Brendan, Matt, and the Amazon CodeWhisperer for command line team.