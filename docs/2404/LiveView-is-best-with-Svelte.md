<!--yml
category: 未分类
date: 2024-05-27 12:53:04
-->

# LiveView is best with Svelte

> 来源：[https://blog.sequin.io/liveview-is-best-with-svelte/](https://blog.sequin.io/liveview-is-best-with-svelte/)

We’re [Sequin](https://sequin.io/docs?ref=blog.sequin.io). We turn 3rd-party API data (e.g. Salesforce, AWS) into Postgres tables and event streams. As an infrastructure company, we questioned if we really need a SPA. So, we started with LiveView, which helped us move fast but left us wanting more. This post is about that journey.

Phoenix's [LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html?ref=blog.sequin.io) has polarized our team. Compared to SPA, there are components and features that we’re able to build 2-3x faster. Conversely, there are components and features that are frustrating to build or feel very counterintuitive.

Said another way, LiveView makes a lot of things easy. But it also makes some easy things hard.

This created tension. Do we keep forging down this path? Or do we give in and convert our app to a SPA?

Fortunately, we found a companion library called [LiveSvelte](https://github.com/woutdp/live_svelte?ref=blog.sequin.io). LiveView enables a development experience with Svelte that’s unlike any other fullstack paradigm I’ve used.

The team agrees: this is a killer way to build.

To appreciate the LiveView+Svelte paradigm, I’ll start by explaining how LiveView works and what makes it different. Then, I’ll detail the friction we encountered with a pure LiveView approach. At that point, you’ll be able to appreciate what LiveSvelte offers.

## What is LiveView

LiveView offers a very unique way to build web applications.

In a traditional server-rendered web application, the server is stateless. The client requests a page and the server renders it. All client actions route back to the server, which re-renders the next page.

In a SPA, the client is in charge of building pages. It leverages a backend API to read and write data. Client apps are stateful (e.g. `useState` in React).

In LiveView, the server is in charge of rendering the page. But it’s stateful. Actions in the frontend are handled by the backend, but the server *incrementally* updates the DOM, much like in a SPA.

At a high-level, the reason a SPA is complex is because distributed systems are complex. Supporting a client JS app is supporting a microservice (and one that runs in a hostile, untrusted environment, no less!)

In *theory* your frontend app uses a backend REST API that *could* be used to support lots of different services and clients. In reality, the needs of your frontend app are unique. So your backend routes and controllers explode with functions that serve the needs of a single client.

If nothing else, this complexity just means shaving a lot of yaks. Each request requires a fair bit of plumbing on both the frontend and the backend. Callstacks can easily exceed half a dozen layers:

*   `onMount`
*   `await api.fetchUsers`
*   `parseResponse`
*   `Router.handle(/api/users)`
*   `AuthPlug.verify_cookie`
*   `UsersController.index`
*   `Users.list_for_org`
*   `ApiHelpers.prepare_response`

The promise of LiveView is that you get to create rich client-side experiences without the frontend microservice. You're back to the much simpler world where you can query the database in the function adjacent to the function that renders your table rows. If a new row comes in, you just need to push it to your table, and LiveView will update the client for you.

But in addition, you also get to enjoy building an app using the stateful paradigm of frontend frameworks. It's much easier and faster to build rich interaction patterns this way vs prior backend paradigms where you'd need to "rebuild the world" on each request.

## Where LiveView makes easy things hard

There's a lot of good stuff in LiveView. But there are also real thorns.

There are two primary areas that we struggled with LiveView:

### Client-side state is inevitable

There is a (literal) speed of light limitation with this approach: your server can only be *so close* to your users.

Invariably, you’re going to need to do some stuff client-side. Animations, tooltips, showing/hiding DOM elements, disabling form fields, etc.

For example, there’s a form in our app with two interdependent dropdowns. Selecting an option in the first dropdown allows our server to generate the list for the second dropdown. To get the best UX, you want to disable the second dropdown immediately after the first dropdown changes. Then, when it’s repopulated by the server, you can re-enable it:

*Simulating 1000ms of roundtrip latency between the client and the server.*

To pull this off, as far as we could tell, you need to use two independent concepts in LiveView:

*   Use the [JS module](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.JS.html?ref=blog.sequin.io) to disable the second dropdown when the first dropdown changes.
*   Use a [hook](https://hexdocs.pm/phoenix_live_view/js-interop.html?ref=blog.sequin.io#client-hooks-via-phx-hook) to register an event listener on the second dropdown. Then, send an action to re-enable the second dropdown from the backend.

And for slightly more complex interaction patterns, you’ll need to incorporate a *third* concept, LiveView state. For example, maybe you only want to re-enable the second dropdown in certain conditions.

The way these three concepts fit together is not obvious (we’re still not sure this is the right pattern!)

So, while the server is in charge of a lot of DOM changes, it can’t command all of them. You use JS and hooks to sprinkle in JavaScript where needed. These tools feel side-chained to core LiveView, and therefore their patterns of use are not obvious. And the more JS and hooks you use, the more of your DOM state now exists *outside* of LiveView.

This is a stark contrast to a paradigm like React. In React, it’s state and actions all the way down. With that core concept, you can do most anything. And there is no blurry line between DOM state and component state.

React can take that approach because there’s no latency between client-side actions and client-side state. This means you can let React’s state paradigm handle every action and transition. Because all of LiveView’s state is server-side, it has to contend with the latency between client-side actions and server-side state. This means that while LiveView state *looks like* other frontend frameworks, the model is actually quite different.

Take input fields, for example. In React, a character can’t be inserted into an input field without routing through state. This unlocks a powerful programming model, where your component re-renders – and therefore responds – to every keystroke. It gives the state and action paradigm a lot of reach, where you can use one core concept (`useState`) to solve a huge space of problems.

In LiveView, it’s more accurate to say that the input field is changed by the user, *and then* a short while later LiveView finds out about it and reacts to it. With no latency, *it looks a lot like React*. But with increased latency, it’s quite a different paradigm.

In frontend frameworks like React, you need to contend with server-side latency all the time. But *when* a high-latency action is going to take place is clear (i.e. you’re fetching from a server). In LiveView, the boundary is murkier.

### Three components

LiveView has three different types of components: LiveViews, LiveComponents, and Components.

LiveViews and LiveComponents are like stateful components in React, whereas Components are like functional components.

Importantly, a LiveView will always be the uppermost parent component. You render LiveComponents and Components as children underneath a LiveView.

In React, it's easy to switch between stateful and functional components–just add or remove `useState` hooks. The API for both are the same (they both accept props in the same way). And outside state, they have an identical feature set. For example, they can both register and respond to DOM events in the same way.

The ease of switching between component types is important. As an app matures, you’re constantly factoring out components. You’re figuring out which bits should be reused, what should be generalized, where state should live, etc.

In LiveView, all three components are very different. As a result, refactoring a LiveView into a LiveComponent is surprisingly cumbersome.

In particular:

*   The syntax for rendering and passing props to LiveViews and LiveComponents is different.
*   The lifecycle of LiveViews and LiveComponents are different.
*   The [communication options](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html?ref=blog.sequin.io#module-unifying-liveview-and-livecomponent-communication) between LiveViews and LiveComponents are different. For example, you `send` to LiveViews but `send_update` to LiveComponents.
*   LiveComponents are not processes, and so can't interact with the rest of the system like LiveViews can.

That last point is what makes LiveComponents so different and so frustrating. The limitations *make sense*: A LiveView is a process. That's one of the best parts about a LiveView, they're "just processes" and so they can fit into your Elixir/OTP system like every other process. For example, you can use pub/sub in a LiveView to subscribe to system-wide changes.

A LiveComponent is *not* its own process, they are modules invoked by a LiveView. The parent LiveView process holds the state for all subcomponents. So, a LiveView has a `pid`, state, and an inbox; a LiveComponent does not. This means the LiveView also has to handle all message routing for its child LiveComponents.

This is in keeping with Elixir/OTP design principles: processes are the building blocks. To give LiveComponents the same powers of independent state management and action handling, they would each need to be their own process.

Still, for the life of me, I *really* struggled with LiveComponents. So often, I wanted to send my LiveComponent an event/action but didn't have a good way to do it. You end up using `send_update`, which is an awkward API. We couldn't decide: do we send *actions* via `send_update`, or do we use it to patch state? If we use it to patch state, how do we tell in our `update` clause whether we're mounting or updating?

## The elusive “LiveView way”

LiveView often made us feel like we were “missing something.” The “LiveView way” feels elusive.

Perhaps LiveView is in an uncanny valley. It shares a lot in common with contemporary frontend frameworks. So, our “React brains” and intuitions would kick in, driving us to use old patterns–but those would often lead to a dead end. More alienness would have forced us to recognize the differences and to approach problems differently.

You can do a lot with just LiveView state and actions. But there are limits, and when you hit them you need to switch paradigms.

It has components to help you organize and reuse code. But due to differences between JavaScript and Elixir, LiveView can’t really offer the same isomorphic component trees without a ton of abstraction, and so has LiveViews and LiveComponents.

**This is what makes LiveSvelte so promising**. As you’ll see, it shifts more responsibility to the frontend. It embraces the fact that the frontend will have its own state. And it lets you take advantage of all the maturity of contemporary JavaScript component frameworks.

## LiveView + Svelte

LiveSvelte lets you render Svelte components from LiveView. It's an awesome paradigm.

There’s a couple different ways to render Svelte from your LiveViews, but the most basic way looks like this:

```
# LiveView component
defmodule Web.SyncLive.Form do
  def render(assigns) do
    assigns = 
      assigns
      |> Map.put(:encoded_collections, Enum.map(assigns.collections, &encode_collection/1))
      |> Map.put(:encoded_errors, encode_errors(assigns.changeset))

    ~H"""
      <.svelte
        name="MyForm"
        props={
          %{
            collections: @encoded_collections,
            credential_options: @credential_options,
            errors: @encoded_errors,
          }
        }
        socket={@socket}
      />
    """
  end
end 
```

This is an Elixir module, the LiveView. Inside the render, we first take our Elixir data structures and encode them for the frontend. We like the pattern of explicitly encoding Elixir structs and such as plain maps before passing to Svelte, like this:

```
 defp encode_collection(%Collection{} = collection) do
    %{
      "id" => collection.id,
      "slug" => collection.slug,
      "name" => collection.name
    }
  end 
```

We’re able to set props on the Svelte component. Those are passed down as you’d expect to the component:

```
// Svelte component
<script>
  export let resource;
  export let credential_options = [];
  export let errors = {};
  export let live;
</script> 
```

One of the props that LiveSvelte sets for us is the `live` prop. To communicate from the Svelte component back up to the LiveView, we can call `live.pushEvent`. For example, check how easy it is to send the server changes to the form:

```
<script>
  // ...
  $: {
    live.pushEvent("form_updated", { form }, () => {});
  }
</script> 
```

This is a reactive block in Svelte. It will be executed whenever the variable `form` is changed. (Kind of like a `useEffect`, where `form` is the dependency.)

The LiveView can handle and respond to the `pushEvent` using typical Elixir message handling semantics:

```
# In the LiveView
# ...
  @impl LiveView
  def handle_event("form_updated", %{"form" => form}, socket) do
    params = decode_params(socket, form)
    {:noreply, merge_changeset(socket, params)}
  end

  defp merge_changeset(socket, params) do
    changeset = Collection.create_changeset(socket.assigns.resource, params)

    assign(socket, :changeset, changeset)
  end 
```

We first decode the params from the frontend, reversing any encoding/mapping we did on the way out. Then, `merge_changeset/2` updates our changeset. If there are any validation errors in the changeset, those will make their way back to the frontend via the `errors` prop.

So, you have data flow from Elixir down to the component via props. The LiveView process can update props at any time to cause the Svelte component to re-render. Any other communication can happen via the websocket.

The boundary between the two is very clear–just as clear as in any SPA.

What's most game-changing, though, is that you have a *backend, stateful process* that is collaborating with a *frontend, stateful process*.

And it's *so* fun and productive.

The three powerhouse properties:

1.  The backend controls the props on the frontend component.
2.  The frontend *and* the backend are stateful.
3.  You have a private, bi-directional communication channel between the two *where either side can initiate a message to the other*.

#1 is made possible thanks to LiveView’s rendering paradigm: re-renders on the server are automatically pushed and applied to the client. This lets the server update props on the component just like a JS parent component can!

#2 is possible because a LiveView is a process. Processes are how Elixir encapsulates and reduces state.

#3 is made possible by the persistent websocket that LiveView gives you, wired to the frontend.

Consider the differences between this paradigm and a SPA:

First, all browser routing happens via the backend. This is a great simplifier. (In a regular SPA you have to maintain *two* sets of routes, one for the browser and one for your API.)

Second, the backend is stateful. It knows what route you’re on. Which resource you’re working with. Each action it handles can be far more incremental, as it’s applying a state change to itself vs rebuilding state from scratch.

Third, communication between the frontend and backend is private and coupled, as it should be. You’re not “polluting” your server’s public routes with a bunch of RPC calls that support a single component. When you see a `pushEvent` in the client, you know exactly where the handler for that is – in the collaborating Elixir module.

Fourth, functionality is split across just two files. Sure, the backend module will call out to your backend functions (e.g. fetch data from database) and the frontend will import components and styles. But roundtrips between the two aren’t routing through a stack of API modules, routers, and controllers.

Fifth, communication between frontend and backend is far less ceremonious. The backend can simply update props to inform frontend changes. And the frontend can `pushEvent` without needing handlers for expired tokens, timeouts, or outages. It’s binary: either the websocket is open which means the server is open for business, or it’s not in which case LiveView helpfully shows the user a global “disconnected” banner.

In the simplest terms, the frontend microservice is eliminated.

What you end up with feels like such a great split of responsibilities with very little boilerplate. All your business logic is on the backend – how you load data, *which* data to load, how to sort and filter the data, your validators, etc. Your frontend code is stupid simple. In Svelte, it’s all (1) `if/end` blocks to conditionally render stuff (2) animations and (3) a few dead simple `pushEvent` functions back to the server.

That last part has been blowing my mind. The typical SPA frontend is full of so much logic, usually `map`, `reduce`, and `filter` in order to process server data, prepare data for display, or prepare data for the server. In a LiveSvelte app, all this can just happen server-side. The LiveView can prepare data exactly as the Svelte component needs it. This keeps complexity in your server language, in your server's data structures, and in your server's test suite.

The backend LiveView and the frontend Svelte component aren't so much coupled as they are two halves: the LiveView only renders that Svelte component, and that Svelte component is only ever rendered by that LiveView.

In contrast to a “regular” LiveView, this paradigm:

*   Embraces state and state transitions in the frontend.
*   Creates a clear boundary layer between the frontend and backend.
*   Leverages Svelte’s component paradigm, which like other contemporary JS frameworks is very mature and familiar.
*   In general, lets great frontend frameworks do what they do best! A pure LiveView approach doesn’t let you tap into this huge ecosystem. (For example, Svelte comes with great animation primitives.)

By moving more into the frontend, we no longer felt like we were straddling an awkward middleground.

We chose LiveSvelte because React didn't have a similarly complete LiveView library. The joy of working with Svelte has been a very happy bonus. Because LiveView does the heavy lifting with state management, our state management in Svelte is very simple. For basic state and reactivity, Svelte is the lightest and fastest frontend framework I've worked with. We also prefer its templating features to React's, namely getting to use `if/else` instead of ternary operators and its conditional property setting.

Further, Svelte 5 is around the corner, and we're bullish on its [runes](https://svelte.dev/blog/runes?ref=blog.sequin.io). We think it makes Svelte even easier to pick up and reason about, meaning everyone on the team is empowered to traverse the stack.

I’m now convinced LiveView shines brightest as a backend-for-frontend. By rendering frontend components, incrementally updating them, maintaining a stateful backend process, and providing a websocket API, it creates a tremendously productive platform for frontend applications.

If you’re using LiveView and resonated with any of the friction I highlighted, you need to give this a try. If you’ve never used LiveView, you’ll find that this paradigm *lowers* the learning curve. This is because you’re able to use a lot of the JavaScript framework primitives you’re used to.

* * *

**Update 4/3/2024**: Join the discussion on [Hacker News](https://news.ycombinator.com/item?id=39916144&ref=blog.sequin.io).