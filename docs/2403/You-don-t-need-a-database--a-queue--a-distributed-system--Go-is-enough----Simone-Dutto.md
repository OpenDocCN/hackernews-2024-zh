<!--yml
category: 未分类
date: 2024-05-27 14:50:27
-->

# You don’t need a database, a queue, a distributed system: Go is enough. | Simone Dutto

> 来源：[https://simonedutto.github.io/2024-03-09/binary-duel](https://simonedutto.github.io/2024-03-09/binary-duel)

Written by Simone Dutto
on  March 09, 2024

# You don't need a database, a queue, a distributed system: Go is enough.

### 

This article is a reminder to me and you all that you probably don’t need to make things harder for yourself.

Reading all these articles about software architectures and scalability has influenced me to think that before shipping an idea we need to have it all figured out.

## The Scalability Tale

We need to choose a *database*: so, let’s start with that. We need to choose a db with a generous free tier, set it up, secure it, check the client API, and debug obscure network thingy (VPC, security group, TLS, etc.).

After that, we need to choose a scalable application deployment: so we need to check the pricing on that, set the scalability metric, check if it works, see the supported runtime, etc.
At a certain point, we asked ourselves self “Do I need to learn K8?”.

Finally, we are gonna write our first character of code, right? No, we need to set up the CI/CD pipeline. So let’s start the fight to make it work with a secret key and permission model (I Love IAM).

By the end of this process the magic, the sparkle is lost and the will to ship the thing is gone away.

Been there, done that.

So with this new project: [Binary Duel](https://binary-duel.com/) ← shameless plug, I decided to go with a completely different approach.

## My Current Project

Binary Duel is a simple quiz game to compete with friends, it even has matchmaking, and it is all handled by a single half-CPU Golang Server.
And I love it.

Incredible features:

*   The quiz state machine is full in memory
*   The queue system for the matchmaking is in memory
*   The questions are stored in a sqlite database
*   There is no auth, no login, no SSO, no nothing
*   It is deployed on an App Engine from the CLI of my personal PC

## FAQ

### Does it scale?

No.

### What if someone decides to DoS your application?

Well, the attack will work. See the [article](https://medium.com/@ifrimvale/binary-duel-front-end-in-2-weeks-with-svelte-and-dasyui-48230a633eb4) of the frontend guy who “friendly fired” my backend.

## Numbers for engineers

The backend is a Golang [Fiber](https://docs.gofiber.io/) backend.

The game state machine is map with a mutex to ensure memory safety.

```
var Games = make(map[string]*Game)
var mux *sync.RWMutex 
```

> I have selected the RW Mutex to improve performance since a lot of requests are RO operations.

The queue for the matchmaking is handled similarly.

The functioning is better explained in this [article](https://medium.com/@ifrimvale/binary-duel-front-end-in-2-weeks-with-svelte-and-dasyui-48230a633eb4), but the main idea is that:

*   there are some actions to progress the state machine (`/submit-answer`, `/join-game`)
*   there is periodic request to check the state of the game machine (`/game-status`)

I have load tested my application with a prod-like load using [Artillery](https://www.artillery.io/).

The backend can handle approximately 100 concurrent games with a median response time of 30ms in a free tier (thanks Google) App Engine. Since each game is expected to last 1-3 minutes. It means that scalability would become an issue if and when Binary Duel is used by more than a few thousands of people an hour.
In this fortunate case, I will be very happy to host the infrastructure on a K8 cluster with autoscaling, self-healing, a distributed database, a Redis server and so on.

## Final Thoughts

Let the scalability be a consequence of the success, not something that holds you back.