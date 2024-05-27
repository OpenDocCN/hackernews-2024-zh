<!--yml
category: 未分类
date: 2024-05-27 13:14:59
-->

# Distributed Authorization

> 来源：[https://www.osohq.com/post/distributed-authorization](https://www.osohq.com/post/distributed-authorization)

> “The ability to do authorization queries over data in our existing database reduces architectural complexity and is a big technical win for us. We’re super excited to take advantage of it.” – Anthony Cristiano, Staff Software Engineer, Headway

Authorization as a Service. The reality falls short of the promise. That’s because to use an authorization service, you have to send it a lot of application data. Any solution that signs you up for a data synchronization process isn’t providing a service. It’s creating a burden.

When we started work on Oso, we knew this. It’s one of the key problems we set out to solve. Today we’re taking a major step towards realizing that vision with the release of ***Distributed Authorization***.

Distributed Authorization lets you apply your application data to authorization decisions without synchronizing it to Oso Cloud. Instead, you distribute authorization decisions between the Oso Cloud service and the Oso Cloud client in your application.

This may sound counterintuitive – shouldn’t an authorization service answer authorization questions on its own? Surprisingly, no.

In this post, we’ll show you how we reached that conclusion and why we’re so excited about Distributed Authorization.

## DIY Authorization: Good until it Isn’t

Authorization starts with a question. Consider [gitcloud](https://github.com/osohq/gitcloud), our sample version control app. You may be asked this question: May `alice` `read` the `amazing-app` repository? In order to answer this question you need two things: logic and data. The logic might be:

A `user` may `read` a `repository` in their `organization` if they have the `maintainer` or `admin` role on that `repository`

The data might be:

*   `alice` is a member of the `acme` organization.
*   the `acme` organization contains the `amazing-app` repository
*   `alice` has the `maintainer` role on the `amazing-app` repository

By combining the logic and the data, we get our answer: `alice` can `read` the `amazing-app` repository.

In the early stages of an application, this logic usually lives in the application code, and the data in the application database. For the case of our simple question above, that might look like this:

```
// Get this user's roles on the repo  const userRoles = db.getUserRoles({ userId: user.id, repoId: issue.repoId}); 
// Check whether the user has a role that confers read access  if(!userRoles.some((r) => ["maintainer", "admin"].includes(r)) {
  throw  new PermissionDeniedRoleRequired("maintainer"); }
```

```
# SQL for getting the repository roles for a user. select "role"  from repository_role rr join repository r on rr.repo_id = r.id
join organization_member om on r.org_id = om.org_id
where om.user_id = ?
    and r.id = ?
```

If your authorization logic isn’t much more complicated than that, this works just fine. But over time, authorization questions and logic both tend to become more complex.

*   It becomes cumbersome to manage roles at the repository level, so you allow users to inherit roles on repositories from their role in the organization.
*   You start getting a lot of requests to add new roles to gitcloud, so you give users the ability to create custom roles with configurable permission
*   You add an Issue feature to gitcloud, so repo users have a way to report bugs.

Great news – your Issue feature is taking off! In fact, it’s so popular that your users request a list of Issues that they have permission to triage, because it’s getting hard to keep track of them. You decide that a user can `triage` an issue if:

*   They created the `issue` *or*
*   They have the `triage` role on the `repository` that contains the `issue` *or*
*   They have the `triage` role on the `organization` that contains the `repository` *or*
*   They have a `custom role` that grants them the necessary permissions

This looks something like this:

Sample code left as an exercise for the reader.

Good luck sifting through that code 6 months from now when you want to figure out the rules for granting triage access to a user.

## Policy As Code: Decouple Authorization Logic From Application Logic

Your application has an authorization policy, whether you build it explicitly or not. When you mix application logic and authorization logic in your application code, you make that policy implicit. The problem with things that are implicit is that they’re easy to misunderstand, or to miss entirely. Suppose you decide to add teams to gitcloud, so different teams in an organization can have different permissions. Will you be able to find all the functions and queries you need to modify in order to make teams work properly? Do you want to have to do that?

This is why we introduced [Polar](https://www.osohq.com/docs/reference/polar/foundations), our declarative DSL for authorization. Polar allows you to separate your authorization logic from your application logic and express it in a language that is purpose built for authorization. A simple Polar policy looks something like this:

```
allow(user: User, "read", org: Organization) if   has_role(user, "member", org); 	
has_role(User{"alice"}, "member", Organization{"acme"});
```

We’ve specified the entities we’re interested in: `User` and `Organization`. We’ve defined the conditions that allow a given `User` to `read` an `Organization`, in terms of the `member` role. Finally, we’ve asserted that `alice` has the `member` role on the `acme` organization.

Like any new language, it takes a little getting used to if you haven’t worked with Polar before, but we can see from this example that it’s both concise and expressive. These are important characteristics if our aim is to make your authorization logic explicit. If you had to wade through a bunch of confusing syntax to divine that logic, we would have done a lot of work for nothing.

Explicit policy is a big improvement for complex authorization logic. Now we needed to let you take advantage of that policy in your code. This is why we built the Oso library.

## Oso Library: Authorization for Monoliths

The Oso library represented our initial implementation of a client to Polar. There was a lot to like about it. It was open source. It operated directly over your data. It integrated tightly with your application. With your policy defined in Polar and the open source library embedded in your application, you could find out whether `alice` can `read` a `repository` like this:

```
@app.route("/repo/<name>") def repo_show(name):
    repo = Repository.get_by_name(name)

  try:
 oso.authorize(User.get_current_user(), "read", repo)

  return f"<h1>A Repo</h1><p>Welcome to repo {repo.name}</p>", 200     except NotFoundError:
  return f"<h1>Whoops!</h1><p>Repo named {name} was not found</p>", 404
```

The beauty of this was that as your authorization logic gets more complicated, the application code stayed just this simple. You expressed the authorization logic in a .polar file, where you could version and inspect it alongside your application code. When you needed to ask if a given user was authorized to perform a given action, you always asked the same question, in exactly those terms:

```
oso.authorize(<User>, <action>, <resource>)
```

As powerful as this was, over time we learned that the library approach has some significant downsides.

The library got access to application data via a [foreign function interface](https://doc.rust-lang.org/nomicon/ffi.html) (FFI). This was clever (and fun) engineering, but the mental model tripped up a lot of developers, and made things hard to debug.

There was no standard data model. This was part of the point – we built the library to operate over your data. As convenient as it was to have direct access to that data, in practice we observed performance issues and inconsistent behavior caused by details of the underlying data model that we could neither see nor change ourselves. This lack of standardization also made it hard for us to build new capabilities on top of the library.

But the main problem was this: the library was a great solution if you have a single service. A lot of people, however, have more than one service! For these people, the library approach only solved a piece of the puzzle.

## Oso Cloud: Authorization for Microservices

When your application is a monolith, it’s straightforward to implement your authorization logic with something like the Oso library. You write your logic in a `.polar` file that lives in the same repository as all the rest of your code. Then you load it when you initialize the client. Finally, you issue authorization requests against the loaded policy. You don’t have to worry about data, because it’s all in a single database that all your code can access.

But when you break your application up into multiple services, you introduce new questions. Which service should own the authorization model? There’s no good answer, because the model needs data to be effective, and that data is now split across services. Can each service have its own model? No, because authorization questions often span services. Let’s go back to deciding whether `alice` can triage an `Issue`.

The Issue service knows whether `alice` created the issue. The Repository service knows whether `alice` has the `triage` role on the repository, and the User service knows whether `alice` has the `triage` role on the organization. So now you’re making multiple API calls to figure out whether `alice` can `triage` the issue.

The design that ends up making the most sense is to have a dedicated service for authorization, but now you have to build your own APIs, create your own data model, figure out how to design change control. It’s a lot of work.

So our next step was to take that work on. We designed Oso Cloud to solve authorization in multi-service environments. This let us address two persistent issues in the library model:

*   We built a hosted solution that could be shared by all services in an application.
*   We defined a consistent data model for authorization data.

The Oso Cloud data model is called [facts](https://www.osohq.com/docs/concepts/oso-cloud-data-model). Facts are a lightweight, flexible format that can represent any authorization data in your application.

Oso Cloud brings the benefits of policy as code to multi-service applications. You can make your authorization logic explicit, share it amongst all of your services, and represent your authorization data in a standard format that’s optimized for authorization operations. But now we’ve recreated exactly the problem we set out to solve: you have to sync your application data to our service.

## Authorization as a Service: Excessive Centralization

Let’s take a look back at why we started down this road. Our application got brittle because the application logic and the authorization logic were intertwined. We wanted to make authorization logic explicit, and in order to do that in a way that worked for microservices, we created a hosted service that provides the benefits of Polar behind an API.

To answer authorization questions, we need more than logic. We also need data. And in that respect, Oso Cloud suffers from the same limitation that all centralized authorization services do. It needs that data to exist *in the service*.

But that’s our problem, not yours. You already have that data where you need it: in your application. By requiring you to convert it to facts and synchronize it to Oso Cloud, we’re making our lives easier at the cost of making yours harder. Now you have to deal with potentially massive initial data loads, distributed transactions, sync and replication processes, detecting drift. None of this is easy.

We know it’s not easy, because you’ve told us it isn’t. The good news is that this was never the end state for us.

## Distributed Authorization: Microservices done right

What we really want to do is let you centralize your authorization logic without centralizing all of your authorization data. To do this, we have to let go of a fundamental assumption of Authorization as a Service: that the service has to answer the question independently. It’s this requirement that makes it necessary for the authorization service to have *all of your application data*.

What if, instead, you could centralize your logic and your common authorization data (things like roles) in the authorization service, and then distribute the evaluation of authorization questions between the server and the client? That’s Distributed Authorization. Instead of only answering `yes` or `no`, Oso Cloud can now respond to an authorization question with `yes, if`.

That `yes, if` is followed by a list of conditions that are evaluated at the client using your local data. You give the Oso Cloud client a configuration file that tells it how to test those conditions by converting your application data to facts. At authorization time, the Oso Cloud service evaluates as much of the request as it knows, and then hands off the remainder to the Oso Cloud client in your application.

## A Practical Application: Filtering Lists

One area where this shines is filtering lists of authorized data. We’ve already talked about how painful it can be to get your application data into a centralized authorization service. It can also be painful to get it back out. If you want to generate a list of Issues that `alice` can `triage` and all of your data is centralized in your authorization service, then the operation looks like this:

*   Ask the service for a list of Issue IDs that `alice` can `triage`
*   Get the full list of IDs back from the service
*   Use the list of IDs to query your local database for Issue data

This is how the `oso.list` command works, and it works great in a lot of cases.

For users who belong to multiple organizations, or who belong to organizations with very active repositories, that list of Issues can get very long! It's not hard to envision situations where a user has access to hundreds or thousands of issues. That’s a lot of data going across the internet before you’re able to use it in your application.

Oso Cloud now supports [List Filtering using Distributed Data](https://www.osohq.com/docs/guides/integrate/filter-lists). When you use local data to generate lists of authorized resources, rather than returning the full list of resources, Oso Cloud instead returns a (much shorter) query filter to the client. The client then uses this filter to construct the full list from your locally stored data. You get your response back from Oso faster, and because you generate the list in your local database, you can sort and paginate the results using the tools you're already using to handle large sets of application data.

## Bringing our Vision into Focus

> “Our team was looking for a new authorization system. We assessed a number of technologies (plus building our own) and Oso's distributed architecture, in-depth auth knowledge, and 99.99% uptime were key reasons why we chose Oso.” – Guhan Venguswamy, Head of Platform Engineering, Jasper.ai

With Distributed Authorization, you get the the benefits of explicit authorization logic without the penalty of centralizing all your authorization data. It’s Authorization as a Service that works with your application, rather than forcing you to rework your application to work with it.

This is a major milestone for us, but we know the job’s not done. We’re continuing to make it easier for you to centralize the data that you do want to keep in Oso Cloud. We’re enhancing Polar to make it even more flexible and robust. And we haven’t forgotten about monoliths – although the Oso library is now deprecated, we’re working on bringing these capabilities to you too!

In the coming weeks, we’ll be sharing more on how Distributed Authorization works under the hood, and the different ways customers like Headway and Jasper.ai are already using it.

If you’d like to learn more about how to use it in your app, [check our our docs for Distributed Authorization and List Filtering](https://www.osohq.com/docs/guides/integrate/filter-lists?utm_content=distributedauthz_launchpost#list-filtering-with-decentralized-data).

Do you have questions or want to share your thoughts? [Reach out to us on Slack](https://join-slack.osohq.com/) or [Schedule a 1x1 with an Engineer](https://calendly.com/osohq/oso-cloud-1-on-1?utm_content=distributed_authz_launchpost)! We’d love to talk with you.

‍