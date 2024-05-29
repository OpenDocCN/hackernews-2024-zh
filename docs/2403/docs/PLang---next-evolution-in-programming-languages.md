<!--yml
category: 未分类
date: 2024-05-29 12:37:03
-->

# PLang - next evolution in programming languages

> 来源：[https://plang.is](https://plang.is)

**Heads up: Building code costs money**
Each code line incurs usually between $0.03 - $0.15 fee via LLM. The payoff? Exceptional efficiency gains.

**Early Adopters Note: PLang is at version 0.1.**

Be prepared for breaking changes and the presence of bugs. Get involved in our

[community on Discord](https://discord.gg/A8kYUymsDD)

or contribute by reporting

[issues on GitHub](https://github.com/PLangHQ/plang/issues)

. If you're keen on aiding PLang's growth, check out how to contribute on our

[GitHub page](https://github.com/PLangHQ)

.

### Hello PLang world

It's easy to create your first application.

1.  Open your favorite text editor.
2.  Type in.

`Start - write out "Hello PLang world"`

Now build & run using (you must

[install PLang first](https://github.com/PLangHQ/plang/blob/main/Documentation/Install.md)

)

plang exec

*On first build you will be ask to pre-purchase credits*

#### Rapid development

You have never seen software being built this fast.

#### Simple development

Just write what you want to happen. `select newest users, 50 per page` and PLang will give you 50 latest users from your database

#### No nonsense development

Focus on **what** the app should do, not *how*.

#### No config hell

There are no configurations to set. Just run. If the programming language needs something from you, it will ask you.

## Variables

Of course there are %variables% `- set %name% as "Dwight Shrewd"`

## Functions

They are called Goals, you can call any Goal and send parameters `- call !FooBar %name%`

## Conditions and lists

You write your if statement, like you want it. Go through lists like never before.

`- if %products.Count% > 0 then - go through %products% call !ProcessProduct`

C# developer? Yes, that is the List.Count property

## Retry, Caching and Error Handling simpler than ever

This is one of those WTF! Check this out!

`- select name from user where id=%id% cache result for 10 minutes retry 10 times over 5 min period on error call !NotifyAdmin`

It caches the results if it returns data, if there is an error it will try again and if that fails the retries it calls a function to notify admin

### Database support

Create tables, views, triggers in Plang. With SQLite, SqlServer, MySql, Postresql or any IDbConnection implementation

*Create Setup.goal file in your directory, this is the content* `Setup - create table comments, columns text(not null), sentiment(not null), created(datetime, default now)`

### LLM built-in

Simply ask the LLM to evaluate request and you can work with the response written to a variable you can work with `Start - [llm] system: is the user content positive or negative user: %userInput% scheme: {sentiment:positive|negative} write to %sentiment% - insert into comments, text=%userInput%, %sentiment%` Now run

plang exec userInput="This is awesome"

### Webserver

Start your webserver in one line `Start - start webserver` Now run

plang exec

### REST support

Want to talk with a REST api, easy, it does everything for you. *Create RestTest.goal in your folder* `RestTest - get http://https://cat-fact.herokuapp.com/facts write to %json% - write out %json.text%`

Now run

plang exec RestTest

### Messages built-in

You can send message between users in a simple way. All encrypted (AES256) `Start - send message to %publicKey% content: Hey, how are you doing today, have your tried PLang?`

### Built-in scraper

Scrape your favorite website, store it for later. `Start - go to https://example.org/ - click, "More information..." link - extract .help-article into %helpContent% - write out %helpContent%`

### Built-in authentication

The language has built in authentication, so no need to handle passwords and emails anymore when communicating with other Plang services. `- get http://api.plang.is/api/balance write to %balance% - write out 'My balance is: %balance%'` The end service can validate that the request is coming from you. No need for username and passwords. User management and registration becomes simpler & more private

### UI - early development

Describe the layout you want `- center content, horizontal and vertical - create login form, #username, #password, TOS (https://plang.is/tos.html) submit to !Login` This will give platform nuetrality, desktop, mobile, watch, tv, or any other

## Events with super powers

*You can add your own events into any Goal or Step* `- before each goal, call !LogGoal - before step nr 2 in !Process.Image, call !Preprocess.Image - on error on !Fetch.Data, call !Report.SysAdmin`

## View Source x 100

*The code is verifiable*

You can see all the source code that are built using PLang.

You can use it to learn and to modify the goals.

All build files are series of json files with instruction on how the PLang runtime should act

## Test your code

Just write what you want to test

## Some of the other features

### Run Python Script

Execute Python scripts effortlessly. `- run processImage.py %imagePath%`

### IDE Support

Download VS Code extension to help with development

[Download now](https://marketplace.visualstudio.com/items?itemName=PlangHQ.plang-extension)

### Caching Made Easy

Efficient data caching mechanism. Need Redis or other, not a problem.

### OS Independence

I need [help with this one](https://github.com/PLangHQ), but underlying code is c#, fully os indepenant.

### Build Events

This powerfull for analyzing, debugging, etc.

### Dependency Injection

Want to user your own logger? `@logger=MyLogger.Logger` or llm, caching, settings, encryption, etc.

### Schedule Tasks

So simple, it's crazy `- every other monday, at 11:23, call !ProcessUsers` or sleep `- wait for 2 sec`

### Compression

Need to work with zip files `- unzip %filepath% to %newPath%` `- compress %filepath% to %compressedPath%`

### Run Command Line Apps

Run any terminal/command line app `- convert video.mp4 to audio.mp3 using ffmpeg`

### Privacy Control

Goal only have access to the path they run from. If they need access outside, it will ask for permission

### App Extensibility

Inject your own logger, settings, llm service, database, caching, encryption modules. They are super simple to write.

### Solid foundation

Plang language is written in C#, it builds and runs on the C# runtime. Secure, fast and solid foundation

## How do I buy?

PLang is an open source, so the software itself is free.
Translating your request to code uses an LLM service, this service is not free of charge.

When you run 'plang build' for the first time, you are asked to buy a voucher for the amount of your choosing.

You can also use OpenAI API service without buying voucher from PLang.
Then OpenAI prices apply and it does not go through PLang service

Buying voucher for the service supports the development of PLang, so please support the project :)

## Price table

### Building code

Input $0.02 / 1K tokens

Output $0.06 / 1K tokens

This is when you run 'plang build'.

### Llm request at runtime
gpt-4-turbo (default)

Input $0.02 / 1K tokens

Output $0.06 / 1K tokens

This is when you run 'plang run' and use the [llm] module

### Llm request at runtime
gpt-3.5

Input $0.0030 / 1K tokens

Output $0.004 / 1K tokens

This is when you run 'plang run' and use the [llm] module with gpt-4-turbo parameter