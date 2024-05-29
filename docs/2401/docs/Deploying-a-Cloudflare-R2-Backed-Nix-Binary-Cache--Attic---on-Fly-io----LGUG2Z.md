<!--yml
category: 未分类
date: 2024-05-27 14:51:46
-->

# Deploying a Cloudflare R2-Backed Nix Binary Cache (Attic!) on Fly.io :: LGUG2Z

> 来源：[https://lgug2z.com/articles/deploying-a-cloudflare-r2-backed-nix-binary-cache-attic-on-fly-io/](https://lgug2z.com/articles/deploying-a-cloudflare-r2-backed-nix-binary-cache-attic-on-fly-io/)

I have tried running the [Attic Nix Binary Cache](https://github.com/zhaofengli/attic) on my Hetzner dedicated server in Germany a few times in the past, but the peering issues and the latency to Xfinity in Seattle have always made me throw my hands up in frustration.

This morning I noticed [a comment](https://github.com/zhaofengli/attic/issues/1#issuecomment-1368527062) by Zhaofeng on the repo issue tracker.

> As a NixOS aficionado myself, I begrudgingly admit that I’ve been running my instance on fly.io 😛

I’m not sure if this is comment is still current, but hey, if Zhaofeng is/was running his binary cache on [fly.io](https://fly.io/), there’s no reason why we can’t too, right?

## Storage[⌗](#storage)

While Attic does support local storage, I figured I’d use Cloudflare’s R2 as a storage backend, both as an excuse to try out the [free tier](https://developers.cloudflare.com/r2/pricing/) (10GB), and because it’ll be easy enough to lift-and-shift if I ever decide to move the server from fly.io.

It’s easy enough to create a new R2 bucket and grab a read-write API token scoped to the bucket on the [Cloudflare Dashboard](https://dash.cloudflare.com).

## Server Configuration[⌗](#server-configuration)

Once we have our credentials, we can put together a configuration file for the Attic server based on the [example config](https://github.com/zhaofengli/attic/blob/e6bedf1869f382cfc51b69848d6e09d51585ead6/server/src/config-template.toml).

```
listen = "[::]:8080" token-hs256-secret-base64 = "<generate this with openssl rand 64 | base64 -w0>"   [database] url = "sqlite:///data/attic.db?mode=rwc"   [storage] bucket = "<your bucket name>" type = "s3" region = "auto" endpoint = "https://<your cloudflare account id>.r2.cloudflarestorage.com"   [storage.credentials] access_key_id = "<your access key id>" secret_access_key = "<your secret access key>"   [chunking] nar-size-threshold = 65536 min-size = 16384 avg-size = 65536 max-size = 262144   [compression] type = "zstd"   [garbage-collection] interval = "12 hours" 
```

One thing to note is that we are storing our SQLite database file at `/data/attic.db` - this is going to be a fly volume that we’ll create soon.

You should probably `git-crypt` this file (or encrypt it with `sops` or `age`) in your configuration repo since it has sensitive credentials.

**Not sure about how to handle secrets in NixOS configuration repos? I have a big old article all about [handling secrets in NixOS](https://lgug2z.com/articles/handling-secrets-in-nixos-an-overview/)!**

## Dockerfile[⌗](#dockerfile)

Luckily, there is an automatically published Docker image available for us to extend. There isn’t much to do here except bring in our configration file, reference it when we start `atticd` and ensure it’s running in `monolithic` mode.

```
FROM ghcr.io/zhaofengli/attic:latest COPY ./server.toml /attic/server.toml EXPOSE 8080 CMD ["-f", "/attic/server.toml", "--mode", "monolithic"] 
```

## Fly Config[⌗](#fly-config)

The last piece of the deployment puzzle is to put together a `fly.toml` file for our `atticd` instance.

Let’s start by ensuring we have a volume created in our desired region.

```
fly volume create atticdata -r sea -n 1 
```

The volume can be called whatever we want, we just need to make sure to update the `source` in the `[mounts]` section in our `fly.toml` file.

```
app = "<pick your own fly app name>" primary_region = "sea"   [mounts]  source = "atticdata" destination = "/data"   [http_service]  internal_port = 8080 force_https = true auto_stop_machines = true auto_start_machines = true min_machines_running = 0 processes = ["app"]   [services.concurrency]  type = "connections" hard_limit = 1000 soft_limit = 1000 
```

That’s pretty much it, time to `fly deploy`! (I do this with `--ha=false` since I don’t want to create more than one machine)

## Generating an Attic token[⌗](#generating-an-attic-token)

We’ll need to execute a command on the newly deployed fly.io machine to generate a “godmode” token for ourselves. Let’s start by getting the machine ID.

```
fly machine list --json |  jq --raw-output '.[].id' 
```

Then we can call the `atticadm` command on the fly.io machine via `fly machine exec` to generate our token.

```
fly machine exec <your machine id> 'atticadm make-token --sub "<your preferred username>" --validity "10y" --pull "*" --push "*" --create-cache "*" --configure-cache "*" --configure-cache-retention "*" --destroy-cache "*" --delete "*" -f /attic/server.toml' 
```

We need to keep this token somewhere safe and preferably encrypted. I keep mine encrypted with `sops-nix` in my NixOS configuration monorepo and have it decrypted and mounted to `/run/secrets/attic/token` on machines that have been given access to decrypt it.

## Logging into Attic[⌗](#logging-into-attic)

This part is pretty simple! We can call the server whatever we like in our configuration; here “fly” will do. The final two arguments are the URL generated for our fly.io app and our access token.

```
# if pkgs.attic is installed in environment.systemPackages attic login fly https://<your fly app name>.fly.dev $(cat /run/secrets/attic/token)   # if pkgs.attic is not installed, we can always run it directly from the flake nix run github:zhaofengli/attic#default login fly https://<your fly app name>.fly.dev $(cat /run/secrets/attic/token) 
```

## Pushing to our Attic cache[⌗](#pushing-to-our-attic-cache)

Now that we have the `atticd` cache server running on fly.io and our computer authenticated with a token that allows us to push, let’s set up a cache and push our entire(!) system configuration.

Note: In my case 256MB of RAM was not enough to keep up with everything I wanted to push! I suggest scaling the RAM to at least 512MB, if not 1GB with `fly scale memory 512`. The snippet below assumes that you have bumped the RAM to 512MB which *should* be able to handle three concurrent jobs (`-j 3`) without the machine giving “out of memory” errors and restarting the `atticd` process.

```
# if pkgs.attic is installed in environment.systemPackages attic cache create system -j 3 attic push system /run/current-system -j 3   # if pkgs.attic is not installed, we can always run it directly from the flake nix run github:zhaofengli/attic#default cache create system -j 3 nix run github:zhaofengli/attic#default push system /run/current-system -j 3 
```

## Configuring NixOS to use our Attic binary cache[⌗](#configuring-nixos-to-use-our-attic-binary-cache)

There are two things that we need to do before we can configure NixOS to use our new binary cache.

First, we need to get the public key of our `system` cache.

Next, we need to create a `netrc` file which contains our `attic` token. The format looks like this:

```
machine <your fly app name>.fly.dev
password <your attic token> 
```

Since this file once again contains sensitive information, if you want to store this in your configuration repo, I recommend encrypting it. In the example below, I have added the `netrc` file contents via `sops-nix`, which mounts the decrypted contents to `/run/secrets/attic/netrc` for machines that have been given decryption access.

```
{  nix = { settings = { substituters = [ "https://nix-community.cachix.org?priority=41" # this is a useful public cache! "https://numtide.cachix.org?priority=42" # this is also a useful public cache! "https://<your fly app name>.fly.dev/system?priority=43" ]; trusted-public-keys = [ "nix-community.cachix.org-1:mB9FSh9qf2dCimDSUo8Zy7bkq5CX+/rkCWyvRCYg3Fs=" "numtide.cachix.org-1:2ps1kLBUWjxIneOy1Ik6cQjb41X0iXVXeHigGmycPPE=" "<your cache public key>" ];   netrc-file = config.sops.secrets."attic/netrc".path; }; }; } 
```

Now, when our NixOS machines are rebuilt:

*   First the official NixOS cache (with a priority of 40) will be checked
*   Next, the `nix-community` public cache (with a priority of 41) will be checked
*   Next, the `numtide` public cache (with a priority of 42) will be checked
*   Finally, our private cache (with a priority of 43) will be checked
*   If there are no cache hits at all, the package will be built from source

What we have at this point is pretty damn cool.

**In the next article we’ll make this even cooler still**, by setting up GitHub Actions jobs [to build each of our NixOS system configurations whenever we push a new commit, and push the outputs of each those builds to our private `system` cache](https://hachyderm.io/@LGUG2Z/111767749455052075)!

If you have any questions or comments you can reach out to me on [Twitter](https://twitter.com/LGUG2Z) and [Mastodon](https://hachyderm.io/@LGUG2Z).

If you’re interested in what I read to come up with solutions like this one, you can subscribe to my [Software Development RSS feed](https://notado.app/feeds/jado/software-development).

If you’d like to watch me writing code while explaining what I’m doing, you can also [subscribe to my YouTube channel](https://www.youtube.com/channel/UCeai3-do-9O4MNy9_xjO6mg?sub_confirmation=1).

If you found this content valuable, or if you are a happy user of [`komorebi`](https://github.com/LGUG2Z/komorebi) or my [NixOS starter templates](https://github.com/LGUG2Z), please consider sponsoring me on [GitHub](https://github.com/sponsors/LGUG2Z) or tipping me on [Ko-fi](https://ko-fi.com/lgug2z).