<!--yml
category: 未分类
date: 2024-05-27 15:03:01
-->

# Using ChatGPT for Podcast Suggestions | ryanbuening.com

> 来源：[https://ryanbuening.com/posts/chatgpt-for-podcast-suggestions/](https://ryanbuening.com/posts/chatgpt-for-podcast-suggestions/)

📅 March 17, 2024

# Using ChatGPT for Podcast Suggestions

An experiment I tried recently with ChatGPT was to see if it could generate some Podcast suggestions for me based on my current podcast subscriptions. I was able to supply ChatGPT with an [OPML file](https://en.wikipedia.org/wiki/OPML) that is commonly used to transfer podcast subscriptions between podcast apps.

Below is the conversation I had:

🧑‍💻

> Are you aware of the OPML.xml format?

🤖

> Yes, I'm aware of the OPML (Outline Processor Markup Language) format. OPML is an XML-based file format used for creating outlines of information. It's commonly used for exchanging lists of web feeds between web feed aggregators or between feed aggregators and podcasting clients. OPML files typically contain a hierarchical structure of outlines, which can include links, metadata, and other information. It's a simple and widely adopted format for organizing and sharing structured data.

🧑‍💻

> I have an OPML podcast file with my current podcasts, can you analyze the text and recommend some other podcasts I might be interested based on my current subscriptions?

🤖

> Sure, please provide me with the OPML file, and I'll be happy to help analyze it and suggest some other podcasts based on your current subscriptions.

🧑‍💻

```
<?xml version='1.0' encoding='UTF-8' standalone='yes' ?>
<opml version="1.0">
  <head>
    <title>Pocket Casts Feeds</title>
  </head>
  <body>
    <outline text="feeds">
      <outline type="rss" text="Developer Tea" xmlUrl="https://feeds.simplecast.com/dLRotFGk" />
      <outline type="rss" text="Hanselminutes with Scott Hanselman" xmlUrl="https://feeds.simplecast.com/gvtxUiIf" />
      <outline type="rss" text="Coding After Work Podcast" xmlUrl="https://codingafterwork.libsyn.com/rss" />
      <outline type="rss" text="SmartLess" xmlUrl="https://rss.art19.com/smartless" />
      <outline type="rss" text=".NET Rocks!" xmlUrl="https://www.spreaker.com/show/5634793/episodes/feed" />
      <outline type="rss" text="Level-up Engineering" xmlUrl="https://feeds.captivate.fm/level-up-engineering/" />
      <outline type="rss" text="Depresh Mode with John Moe" xmlUrl="https://feeds.simplecast.com/Hq3igj9D" />
      <outline type="rss" text="All It Takes Is A Goal" xmlUrl="https://feeds.buzzsprout.com/1556747.rss" />
      <outline type="rss" text="The Tim Ferriss Show" xmlUrl="https://rss.art19.com/tim-ferriss-show" />
      <outline type="rss" text="The Greg McKeown Podcast" xmlUrl="https://feeds.megaphone.fm/WOM3905173735" />
      <outline type="rss" text="Hidden Brain" xmlUrl="https://feeds.simplecast.com/kwWc0lhf" />
      <outline type="rss" text="The InfoQ Podcast" xmlUrl="https://feeds.soundcloud.com/users/soundcloud:users:215740450/sounds.rss" />
      <outline type="rss" text="The Rabbit Hole: The Definitive Developer's Podcast" xmlUrl="https://therabbithole.libsyn.com/rss" />
      <outline type="rss" text="Podcast – Rich Roll" xmlUrl="http://www.richroll.com/category/podcast/feed/" />
      <outline type="rss" text="Merge Conflict" xmlUrl="https://feeds.fireside.fm/mergeconflict/rss" />
      <outline type="rss" text="Tabs and Spaces Podcast" xmlUrl="https://tabsandspaces.libsyn.com/rss" />
      <outline type="rss" text="Scott H Young Podcast" xmlUrl="https://feeds.soundcloud.com/users/soundcloud:users:328172648/sounds.rss" />
      <outline type="rss" text="Money Guy Show" xmlUrl="http://moneyguyshow.libsyn.com/rss" />
      <outline type="rss" text="Awkward Silences" xmlUrl="https://feeds.transistor.fm/awkward-silences" />
      <outline type="rss" text="Finding Mastery" xmlUrl="https://rss.art19.com/finding-mastery" />
      <outline type="rss" text="How to Money" xmlUrl="https://www.omnycontent.com/d/playlist/e73c998e-6e60-432f-8610-ae210140c5b1/63841c13-0d18-43da-80af-ae320005b8a3/ac04f6c2-caa5-49c7-9e23-ae320005b8ad/podcast.rss" />
      <outline type="rss" text="The Changelog: Software Development, Open Source" xmlUrl="https://changelog.com/podcast/feed" />
      <outline type="rss" text="Choiceology with Katy Milkman" xmlUrl="https://feeds.pacific-content.com/choiceology" />
      <outline type="rss" text="Coding Blocks" xmlUrl="https://www.codingblocks.net/podcast-feed.xml" />
      <outline type="rss" text="Ten Percent Happier with Dan Harris" xmlUrl="https://rss.art19.com/ten-percent-happier" />
      <outline type="rss" text="Huberman Lab" xmlUrl="https://feeds.megaphone.fm/hubermanlab" />
      <outline type="rss" text="The Modern .NET Show" xmlUrl="https://thedotnetcorepodcast.libsyn.com/rss" />
      <outline type="rss" text="Armchair Expert with Dax Shepard" xmlUrl="https://feeds.megaphone.fm/armchair-expert-wide" />
      <outline type="rss" text="Akimbo: A Podcast from Seth Godin" xmlUrl="https://feeds.acast.com/public/shows/07058b20-e537-4225-9c56-3adf0e2a669a" />
      <outline type="rss" text="Feel Better, Live More with Dr Rangan Chatterjee" xmlUrl="https://feeds.acast.com/public/shows/beee88f6-2309-4034-aed1-949dc57f9779" />
      <outline type="rss" text="The Jordan Harbinger Show" xmlUrl="https://up.audio/rss/1344999619/www.podcastone.com/podcast?categoryID2=1237" />
      <outline type="rss" text="Design Details" xmlUrl="https://feeds.simplecast.com/eew_vyNL" />
      <outline type="rss" text="Web Rush" xmlUrl="https://feeds.simplecast.com/tOjNXec5" />
      <outline type="rss" text="Programming Throwdown" xmlUrl="https://feeds.transistor.fm/programming-throwdown" />
      <outline type="rss" text="The Happiness Lab with Dr. Laurie Santos" xmlUrl="https://www.omnycontent.com/d/playlist/e73c998e-6e60-432f-8610-ae210140c5b1/96c5c41e-0bc8-4661-b184-ae32006cd726/d623ef0b-3fee-4c26-b815-ae32006cd739/podcast.rss" />
      <outline type="rss" text="Nudge" xmlUrl="https://feeds.megaphone.fm/HS2038060283" />
      <outline type="rss" text="ReThinking" xmlUrl="https://feeds.feedburner.com/TED_TakenForGranted" />
      <outline type="rss" text="Maintainable" xmlUrl="https://feeds.simplecast.com/7y1CbAbN" />
      <outline type="rss" text="no dogma podcast" xmlUrl="https://feeds.simplecast.com/EoO4h3Xv" />
      <outline type="rss" text="The Backend Engineering Show with Hussein Nasser" xmlUrl="https://anchor.fm/s/1eb6d14/podcast/rss" />
      <outline type="rss" text="ShopTalk" xmlUrl="https://shoptalkshow.com/feed/podcast/" />
      <outline type="rss" text="The Art of Manliness" xmlUrl="https://feeds.simplecast.com/azy33G8_" />
      <outline type="rss" text="Gold Minds with Kevin Hart" xmlUrl="https://feeds.simplecast.com/ijknOKeU" />
      <outline type="rss" text="The Knowledge Project with Shane Parrish" xmlUrl="https://theknowledgeproject.libsyn.com/rss" />
      <outline type="rss" text="Front End Happy Hour" xmlUrl="https://feeds.soundcloud.com/users/soundcloud:users:206137365/sounds.rss" />
      <outline type="rss" text="Dev Questions with Tim Corey" xmlUrl="https://feeds.buzzsprout.com/1183862.rss" />
      <outline type="rss" text="REWORK" xmlUrl="https://feeds.buzzsprout.com/2260539.rss" />
      <outline type="rss" text="The Unhandled Exception Podcast" xmlUrl="https://feeds.buzzsprout.com/978640.rss" />
      <outline type="rss" text="Tetragrammaton with Rick Rubin" xmlUrl="https://feeds.megaphone.fm/tetragrammaton" />
      <outline type="rss" text="Beyond the To-Do List" xmlUrl="https://feeds.megaphone.fm/TNM6091206471" />
      <outline type="rss" text="The Stack Overflow Podcast" xmlUrl="https://feeds.simplecast.com/XA_851k3" />
      <outline type="rss" text="Conan O’Brien Needs A Friend" xmlUrl="https://feeds.simplecast.com/dHoohVNH" />
      <outline type="rss" text="dotnet rambles" xmlUrl="https://anchor.fm/s/705ad7e4/podcast/rss" />
      <outline type="rss" text="How I Built This with Guy Raz" xmlUrl="https://rss.art19.com/how-i-built-this" />
      <outline type="rss" text="The Productive C# Podcast" xmlUrl="https://anchor.fm/s/991d954/podcast/rss" />
      <outline type="rss" text="The Always Sunny Podcast" xmlUrl="https://feeds.megaphone.fm/TPC3838283892" />
      <outline type="rss" text="A Slight Change of Plans" xmlUrl="https://www.omnycontent.com/d/playlist/e73c998e-6e60-432f-8610-ae210140c5b1/0f1283e2-80fe-460b-b157-ae3900375d38/d548f2c1-d522-4aa8-9ff3-ae3900375d41/podcast.rss" />
      <outline type="rss" text="Soft Skills Engineering" xmlUrl="https://softskills.audio/feed.xml" />
      <outline type="rss" text="Dev Interrupted" xmlUrl="https://feeds.buzzsprout.com/1422892.rss" />
      <outline type="rss" text="The Carey Nieuwhof Leadership Podcast" xmlUrl="https://careynieuwhof.libsyn.com/rss" />
      <outline type="rss" text="The Daily Stoic" xmlUrl="https://rss.art19.com/the-daily-stoic" />
      <outline type="rss" text="Azure DevOps Podcast" xmlUrl="https://azuredevopspodcast.clear-measure.com/rss" />
      <outline type="rss" text="IT Career Energizer" xmlUrl="https://itcareerenergizer.libsyn.com/rss" />
      <outline type="rss" text="The Bike Shed" xmlUrl="https://feeds.fireside.fm/bikeshed/rss" />
      <outline type="rss" text="Impact Theory with Tom Bilyeu" xmlUrl="https://feeds.megaphone.fm/ITLLC7422673968" />
      <outline type="rss" text="CodeNewbie" xmlUrl="http://feeds.codenewbie.org/cnpodcast.xml" />
      <outline type="rss" text="On Purpose with Jay Shetty" xmlUrl="https://www.omnycontent.com/d/playlist/e73c998e-6e60-432f-8610-ae210140c5b1/32f1779e-bc01-4d36-89e6-afcb01070c82/e0c8382f-48d4-42bb-89d5-afcb01075cb4/podcast.rss" />
      <outline type="rss" text="Adventures in .NET" xmlUrl="https://feeds.redcircle.com/257bddf3-88dd-40c3-b457-ec050c614ba3" />
      <outline type="rss" text="CoRecursive: Coding Stories" xmlUrl="https://corecursive.libsyn.com/feed" />
      <outline type="rss" text="Deep Questions with Cal Newport" xmlUrl="https://feeds.buzzsprout.com/1121972.rss" />
      <outline type="rss" text="Software Engineering Radio - the podcast for professional software developers" xmlUrl="https://seradio.libsyn.com/rss" />
      <outline type="rss" text="Meta-Cast" xmlUrl="https://feeds.captivate.fm/meta-cast/" />
    </outline>
  </body>
</opml>
```

🤖

> Great! You have quite a diverse selection of podcasts in your subscriptions. Based on your current subscriptions, here are some additional podcasts you might be interested in:

```
Reply All: A podcast about the internet that is actually an unfailingly original exploration of modern life and how to survive it.

The Joe Rogan Experience: A long-form, in-depth conversation with guests from various backgrounds, including comedians, actors, musicians, authors, scientists, and more.

TED Radio Hour: A journey through fascinating ideas, astonishing inventions, and new ways to think and create.

Freakonomics Radio: Explores the hidden side of everything, featuring interviews with Nobel laureates, provocateurs, and more.

Science Vs: Pits facts against popular myths and misconceptions, exploring the science behind contentious topics.

The Moth: True stories told live, featuring a variety of voices and experiences from people around the world.

Radiolab: Investigates a strange world, telling stories of science, philosophy, and the human experience in innovative ways.

Revisionist History: Reinterprets events, ideas, and people from the past, challenging conventional wisdom and shedding new light on familiar subjects.

Invisibilia: Explores the invisible forces that shape human behavior, blending storytelling with scientific research to uncover fascinating insights.

The Daily: A news podcast by The New York Times, covering the biggest stories of our time in depth and detail.
```

> These podcasts cover a wide range of topics, from science and psychology to storytelling and current events. Hopefully, you'll find some new favorites among them!

🧑‍💻

> Can you suggest some more podcasts I might be interested in based on my OPML file?

🤖

> Certainly! Based on your current subscriptions, here are some more podcasts you might find interesting:

```
Syntax - Tasty Web Development Treats: A podcast all about web development, covering topics like JavaScript frameworks, CSS, and more.

The Indie Hackers Podcast: Interviews with successful indie founders, exploring how they built and grew their businesses.

The GaryVee Audio Experience: Insights and advice on entrepreneurship, marketing, and personal development from Gary Vaynerchuk.

The Creative Penn Podcast: Interviews with authors, entrepreneurs, and publishing professionals, offering insights into writing, self-publishing, and book marketing.

The Indie Hackers Podcast: Interviews with successful indie founders, exploring how they built and grew their businesses.

The Accidental Creative: Strategies for unlocking creativity and productivity, helping you thrive in your work and life.

The Growth Show by HubSpot: Interviews with business leaders and entrepreneurs, sharing insights on growth strategies and tactics.

The Side Hustle School: Inspiring stories of people who have turned their side projects into successful businesses, offering practical advice for aspiring entrepreneurs.

The Tony Robbins Podcast: Interviews with thought leaders and experts in personal development, offering strategies for achieving success and fulfillment.

The Tim Ferriss Show: Interviews with top performers from various fields, exploring their habits, routines, and strategies for success.
```

> These podcasts align with your interest in technology, personal development, entrepreneurship, and business. Hopefully, you'll find some valuable insights and inspiration from them!

#### Conclusion

Overall, I was pleased with the quality of recommendations. ChatGPT did suggest The Tim Ferriss Show which already existed in my OPML file, but outside of that, it was able to give some decent suggestions. 🎙️🎧

# [random](/tags/random) | [technology](/tags/technology)