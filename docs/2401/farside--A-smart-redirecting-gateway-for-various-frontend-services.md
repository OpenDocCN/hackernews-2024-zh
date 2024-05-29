<!--yml
category: 未分类
date: 2024-05-27 15:23:39
-->

# farside: A smart redirecting gateway for various frontend services

> 来源：[https://sr.ht/~benbusby/farside/](https://sr.ht/~benbusby/farside/)

Contents

1.  [About](#about)
2.  [Demo](#demo)
3.  [How It Works](#how-it-works)
4.  [Cloudflare](#regarding-cloudflare)
5.  [Development](#development)
    1.  [Compiling](#compiling)
    2.  [Environment Variables](#environment-variables)

### [#](#about)About

A redirecting service for FOSS alternative frontends.

[Farside](https://farside.link) provides links that automatically redirect to working instances of privacy-oriented alternative frontends, such as Nitter, Libreddit, etc. This allows for users to have more reliable access to the available public instances for a particular service, while also helping to distribute traffic more evenly across all instances and avoid performance bottlenecks and rate-limiting.

Farside also integrates smoothly with basic redirector extensions in most browsers. For a simple example setup, [refer to the wiki](https://github.com/benbusby/farside/wiki/Browser-Extension).

### [#](#demo)Demo

Farside's links work with the following structure: `farside.link/<service>/<path>`

For example:

^(Note: This table doesn't include all available services. For a complete list of supported frontends, see: [https://farside.link](https://farside.link))

Farside also accepts URLs to "parent" services, and will redirect to an appropriate front end service, for example:

### [#](#how-it-works)How It Works

The app runs with an internally scheduled cron task that queries all instances for services defined in [services.json](https://git.sr.ht/~benbusby/farside/tree/HEAD/services.json) every 5 minutes. For each instance, as long as the instance takes <5 seconds to respond and returns a successful response code, the instance is added to a list of available instances for that particular service. If not, it is discarded until the next update period.

Farside's routing is very minimal, with only the following routes:

*   `/`
    *   The app home page, displaying all live instances for every service
*   `/:service/*glob`
    *   The main endpoint for redirecting a user to a working instance of a particular service with the specified path
    *   Ex: `/libreddit/r/popular` would navigate to `<libreddit instance URL>/r/popular`
        *   If the service provided is actually a URL to a "parent" service (i.e. "youtube.com" instead of "piped" or "invidious"), Farside will determine the correct frontend to use for the specified URL.
    *   Note that a path is not required. `/libreddit` for example will still redirect the user to a working libreddit instance
*   `/_/:service/*glob`
    *   Achieves the same redirect as the main `/:service/*glob` endpoint, but preserves a short landing page in the browser's history to allow quickly jumping between instances by navigating back.
    *   Ex: `/_/nitter` -> nitter instance A -> (navigate back one page) -> nitter instance B -> ...
    *   *Note: Uses Javascript to preserve the page in history*

When a service is requested with the `/:service/...` endpoint, Farside requests the list of working instances from the db and returns a random one from the list and adds that instance as a new entry in the db to remove from subsequent requests for that service. For example:

A user navigates to `/nitter` and is redirected to `nitter.net`. The next user to request `/nitter` will be guaranteed to not be directed to `nitter.net`, and will instead be redirected to a separate (random) working instance. That instance will now take the place of `nitter.net` as the "reserved" instance, and `nitter.net` will be returned to the list of available Nitter instances.

This "reserving" of previously chosen instances is performed in an attempt to ensure better distribution of traffic to available instances for each service.

Farside also has built-in IP ratelimiting for all requests, enforcing only one request per second per IP.

### [#](#regarding-cloudflare)Regarding Cloudflare

Instances for each supported service that are deployed behind Cloudflare are not included when using [farside.link](https://farside.link). If you would like to also access instances that use Cloudflare (in addition to instances that do not), you can either use [cf.farside.link](https://cf.farside.link) instead, or deploy your own instance of Farside and set `FARSIDE_SERVICES_JSON=services-full.json` when running.

If you do decide to use [cf.farside.link](https://cf.farside.link) or use the full instance list provided by `services-full.json`, please be aware that Cloudflare takes steps to block site visitors using Tor (and some VPNs), and that their mission to centralize the entire web behind their service ultimately goes against what Farside is trying to solve. Use at your own discretion.

### [#](#development)Development

To run Farside without compiling, you can perform the following steps:

*   Install dependencies: `mix deps.get`
*   Initialize db contents: `FARSIDE_CRON=0 mix run -e Farside.Instances.sync`
*   Run Farside: `mix run --no-halt`

#### [#](#compiling)Compiling

You can create a standalone Farside app using the steps below. In the example, the Farside executable is copied to `/usr/local/bin`, but can be moved to any preferred destination. Note that the executable still depends on the C runtime of the machine it is built on, so if you want a more portable binary, you should build Farside on a system with older library versions.

```
MIX_ENV=cli && mix deps.get && mix release
cp _build/cli/rel/bakeware/farside /usr/local/bin
sudo chmod +x /usr/local/bin/farside
farside 
```

#### [#](#environment-variables)Environment Variables

| Name | Purpose |
| FARSIDE_TEST | If enabled, bypasses the instance availability check and adds all instances to the pool. |
| FARSIDE_PORT | The port to run Farside on (default: `4001`) |
| FARSIDE_DATA_DIR | The path to the directory to use for storing instance data (default: `/tmp`) |
| FARSIDE_SERVICES_JSON | The JSON file to use for selecting instances (default: `services.json`) |
| FARSIDE_CRON | Set to 0 to deactivate the scheduled instance availability check (default on). |