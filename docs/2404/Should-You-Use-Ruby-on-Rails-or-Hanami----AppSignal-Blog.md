<!--yml
category: 未分类
date: 2024-05-27 13:30:01
-->

# Should You Use Ruby on Rails or Hanami? | AppSignal Blog

> 来源：[https://blog.appsignal.com/2024/04/24/should-you-use-ruby-on-rails-or-hanami.html](https://blog.appsignal.com/2024/04/24/should-you-use-ruby-on-rails-or-hanami.html)

*This post was updated on 23 May 2024 to clarify the differences in models and data persistence between Rails and Hanami.*

Ruby on Rails is the most popular web framework in the Ruby ecosystem and has a large user base, ranging from freelancers to large established companies. With an active user community and wide-ranging documentation, it can be used to build everything from simple applications to complex web platforms.

That said, a new contestant is taking on Rails’ dominance for the full-stack Ruby framework title: Hanami. It is a fast, modular Ruby framework with improved performance and maintainability compared to Rails.

In this article, we'll explore the strengths and weaknesses of each framework in terms of performance, features, testing, and more. So whether you are looking to build a customer-facing web app, an internal tool, or a massively scalable API, you should leave being better informed about what to use for your next project.

Let's get started with a brief introduction to each framework.

## Introducing Ruby on Rails

[Ruby on Rails](https://guides.rubyonrails.org/getting_started.html) is the most well-known Ruby web application development framework with a mission to enhance developer productivity (by making a bunch of assumptions about how apps should be built, often referred to as the "Rails way").

Some of these assumptions include making sure developers do not spend time configuring stuff, or what is termed as "convention over configuration", and an emphasis on using DRY ("don't repeat yourself") principles. DRY principles encourage a developer to avoid repeating code over and over again, but instead use single and focused representations of app functionality to ensure maintainability and organization.

## Introducing Hanami

While Rails is very well-known in the Ruby community, [Hanami](https://hanamirb.org/) is less so. It's a fairly new modern Ruby framework trying to take on Rails' dominance of the full-stack web framework space.

Hanami is built from the ground up to have a small memory footprint and a focus on modularity, which, in turn, makes for a very fast and nimble framework.

Obviously, these brief introductions won't really give you all the information you need to decide on the framework that suits you best. For that, we'll need to dive deeper into each, starting with how they are structured.

## Structure and Architecture of Rails and Hanami

Rails and Hanami are somewhat similar in that both are Ruby frameworks. However, how each is built and their application architecture is where you'll find most of the differences.

For starters, with Rails, you have fewer files or *abstractions* (the building blocks you use to build your app), which tend to grow in size as you develop your app. On the other hand, Hanami takes abstractions to a whole new level, with lots of files that tend to be smaller in size.

The diagrams below illustrate the point more clearly. Let's start with Rails.

Now compare the Rails abstraction diagram with the Hanami one below.

As you can see, each framework generally follows the *model-view-controller* (MVC) structure. However, Hanami takes abstraction to the next level.

Here's a simplified outline of how each framework is organized:

*   **Routes** - Each framework has a routes definition with your app's endpoints.
*   **Controllers vs. actions** - In Rails, you get controllers with one or a couple of actions in them. Controllers receive and respond to route requests by directing them to the relevant actions. In Hanami, there are no controllers. Instead, you go straight to self-contained actions (each action has its own self-contained class).
*   **Models and persistence** - The Rails model layer takes care of data validation and database communication, including any queries your app may have. Persistence is handled by the Ruby Object Mapper (ROM). In Hanami, the model layer is more abstract, separated into entities and repositories. Entities handle domain logic and are database-agnostic, whereas *repositories* (repos) are used to communicate with the database. Hanami 2.0 onwards ships without a pre-configured persistence layer, allowing you to choose an Object Relational Mapper (ORM) that suits your needs or program ORM free.
*   **View rendering** - As with the persistence layer, Ruby on Rails views tend to contain everything you need for rendering data to the outside world in one place. This includes all the HTML structure, view helpers, and view logic. When it comes to view rendering in Hanami, things are more abstract. For starters, you have *views* that utilize any view helpers the app has and also render the *template*. The *template* handles the actual HTML structure, while the *parts* take care of any presentation logic.

So, what does all this mean when it comes to building an app? With fewer abstractions, Rails is a good choice for getting your app off the ground and is more beginner-friendly (as we shall see later on in this article). With Rails, you can build very robust monolith apps, but your code will get more and more complex as you scale. On the other hand, Hanami's more complex structure can be daunting to learn, but it will allow you to build massively scalable applications and could make for much better code organization.

Next, let's take a look at each framework's ecosystem.

## Ecosystem and Community

As we mentioned earlier, Rails has a more established framework than Hanami. Rails has been around for longer and, as such, has a bigger and more mature community.

The differences in ecosystem and community are summarized below:

*   **Documentation** - It doesn't matter what you're trying to build, if you use Rails, you have access to very well-done documentation. You are guided through all parts of an app build, from authentication, to data persistence, view presentation, and everything in between. Additionally, a wide range of third-party tutorials cover almost everything imaginable in terms of building an application.

That said, things are not so established on the Hanami side of things. Being a relatively newer framework, Hanami has a comparatively limited documentation base. The Hanami team has done an impressive job with the official guides, but compared to the Rails documentation base, it doesn't come close.

*   **Community** - Here, as with the documentation, Rails is the clear winner. Rails has been around much longer than Hanami, so it's been adopted the most. It doesn't matter whether you are a beginner or advanced developer, you'll find a Rails community on relevant subreddits, Slack groups, Discords, and so forth. On the other hand, Hanami is still growing, meaning it has a much smaller community.
*   **Gems and libraries** - Since both Hanami and Rails are Ruby frameworks, it could be argued that gems that work in Rails will work in Hanami. Although this is technically true, you'll often find that Rails has a more established base of gems and libraries for all sorts of specialized functions.

That said, because of Hanami's focus on abstractions and specialization, it has some very advanced gems that could take your Rails apps to the next level. For example, the default [dry-rb gems](https://dry-rb.org/) in Hanami could bring better code organization and abstraction when used in Rails apps.

Moving on, let's compare each framework's learning curve and adoption.

## Ease of Use, Adoption, Governance, and Learning Curve

For the very reasons outlined in the previous section, Ruby on Rails easily beats Hanami in terms of ease of use, industry adoption, and learning curve, specifically:

*   **Learning** - Imagine you're a complete beginner who's looking to learn a new programming language. Your very first thought will be to check out online learning resources. If a framework has widely available learning materials covering the app-building process from start to finish, you'll likely choose that language over others. And because Rails has a more established documentation base, it trumps Hanami as the beginner's choice of framework. In addition, Rails is much more friendly for beginners to pick up since it makes so many assumptions under the hood (compared to Hanami, which offers a more abstract way of building apps). Hanami's abstractions can be daunting even for established Rails developers.
*   **Industry adoption** - Without including the adoption of other popular and more established frameworks like Python, React, C#, and others, if we consider the adoption of Ruby frameworks, Rails easily eclipses Hanami. The [Rails homepage](https://rubyonrails.org/) lists some big-name organizations using the framework. On the other hand, as the new kid on the block, Hanami is not so widely adopted. We'll have to wait and see whether that will change in the future.
*   **Job market prospects** - In terms of job prospects, you'll find more job openings for Rails developers than Hanami developers.
*   **Governance** - Another important aspect that should not be overlooked is governance. A changing landscape guides how open-source frameworks evolve and advance. However, more than that, the core team members of these frameworks often wield immense powers and can dictate what goes into a framework, how it develops, etc.

A good example is [January's announcement](https://twitter.com/dhh/status/1743664413964374505) by David Heinemeier Hansson (DHH), the creator of Rails. He said that, in the future, he would push for Rails to offer first-class support for building full-stack progressive web applications and native notifications. These features would make Rails very attractive to mobile developers.

In reaction, many developers, some outside the Ruby ecosystem and even others who had dropped Rails over the years, overwhelmingly responded in the positive, saying they would gladly pick up the framework or learn it if DHH kept his word. This example just goes to show you that governance is a very important consideration when deciding what framework to pick.

Let's now switch gears and look into something more technical, application performance.

## Performance: Ruby on Rails vs. Hanami

As a developer, you will be concerned about your app's reliability, responsiveness, and how it will utilize server resources once deployed in production. These are fundamental concerns that can greatly affect your choice of framework.

You could use many methods to run performance tests on your app, one of the most popular being [Apache JMeter](https://jmeter.apache.org/). Let's use the [benchmark numbers](https://web-frameworks-benchmark.netlify.app/result?asc=0&f=hanami,rails&order_by=level64) to compare Hanami and Rails.

Here's a quick benchmark test showing the number of requests each framework can handle per second:

Hanami beats Rails hands down, with the ability to handle 3 times more requests than Rails can handle.

This next screenshot shows the average latency values for each framework:

Again, Hanami beats Rails with an average latency time that's about 3 times shorter.

If you're looking for a Ruby framework that is blazingly fast (let's say, you need to work on a fast and hugely scalable API), then you'll be hard-pressed to find anything better than Hanami in the Ruby landscape.

## Testing

When it comes to testing code, both frameworks are very much comparable since you can test either using the versatile [RSpec](https://rspec.info/) library.

RSpec is included via the `hanami-rspec` gem for Hanami apps, whereas you need to install the `rspec-rails` gem to use it in your Rails app. The example below shows a very basic test spec that is included with your new Hanami app:

Running it with `$ bundle exec rspec spec/requests/root_spec.rb` should result in a passing test (assuming you've not edited the default view template):

On the Rails side, you need to handle some things by yourself. Firstly, you need to add the RSpec gem to the development/test block in the `Gemfile` as shown below, then run `bundle`:

The next step is to run the RSpec install with the command `bundle exec rails generate rspec:install` to get it ready for your Rails project.

Finally, before writing and running any tests, you'll likely need to install another gem to define test data for your Rails app: [FactoryBot](https://github.com/thoughtbot/factory_bot_rails).

And assuming you've set up your test suite properly and defined a few tests, you can run a test just as you would in Hanami, with `bundle exec rspec spec/models/user_spec.rb`. You can get your test results just as you would with a Hanami app:

Finally, let's see how the frameworks compare when it comes to deployment.

## Deployment

Nowadays, there are lots of options for deploying Ruby apps to production.

To begin with, you could go with a Platform-as-a-Service (PaaS) provider like [Heroku](https://www.heroku.com/ruby), or [Fly](https://fly.io/) for a more seamless experience. You can also do a bit of DevOps: set up a Docker installation on a VPS and deploy your app there.

Whichever option you choose, you can expect deployment to be relatively similar in both cases. The only caveat is the lack of extensive documentation and tutorials for deploying Hanami apps.

As we pointed out before, being a newer framework, Hanami still doesn't have extensive documentation covering the deployment process and the challenges that might crop up. If you don't mind hacking around this, then you should be okay.

## Wrapping Up

In this article, we've looked at two Ruby frameworks — Ruby on Rails and Hanami — comparing them in terms of features, architecture, performance, and more. Ultimately, we have to answer the question: "Which framework should I use, and why?". We can summarize as follows:

*   If you're a beginner Ruby developer, start with Rails: it's easier to pick up and learn. If you're a more advanced Ruby developer, develop a Hanami skillset, as it will help you develop even more robust Ruby applications.
*   If you need to develop an app that should be fast, say an API serving several clients, Hanami's small memory footprint, blazing fast responses, and low latency will serve you well. However, if you just need to develop a monolithic full-stack app to validate a SaaS idea, Rails will definitely get you there faster.

Ultimately, what you end up choosing is really up to you. It isn't a bad idea to acquire skills in both frameworks. That way, you can decide on the best framework to go for, depending on the needs of your project.

Happy coding!

**P.S. If you'd like to read Ruby Magic posts as soon as they get off the press, [subscribe to our Ruby Magic newsletter and never miss a single post](https://blog.appsignal.com/ruby-magic)!**

**P.P.S. Did you know AppSignal has an integration for [Rails](https://www.appsignal.com/ruby/rails-monitoring) and [Hanami](https://www.appsignal.com/ruby/hanami-monitoring)?**