<!--yml
category: 未分类
date: 2024-05-27 14:37:05
-->

# Reclaim your focus with ~12 lines of bash — The Autodidacts

> 来源：[https://www.autodidacts.io/coldturkey-selfcontrol-freedom-leechblock-alternative-in-bash/](https://www.autodidacts.io/coldturkey-selfcontrol-freedom-leechblock-alternative-in-bash/)

Computers are a tool that you can use or be used by. [I have written about my tool-taming tactics before](https://autodidacts.io/intentional-computing/?ref=autodidacts.io), including blocking distracting websites in my hosts file. My “checking” function is the natural evolution of that tactic.

It implements the core functionality of popular anti-procrastination apps like [ColdTurkey](https://getcoldturkey.com/?ref=autodidacts.io), [LeechBlock NG](https://www.proginosko.com/leechblock/?ref=autodidacts.io), [Freedom](https://freedom.to/?ref=autodidacts.io), and [Self Control](https://selfcontrolapp.com/?ref=autodidacts.io) **in just ~12 lines of bash**.

Here’s how to get set up with “checking”:

1.  Back-up your existing `/etc/hosts` file.
2.  Rename it to `/etc/hosts-checking`.
3.  Create a copy of it at `/etc/hosts-doing`, and add all the distracting websites to it. My personal kryptonite: Email, HackerNews, Lobste.rs, Website Stats, and Spotify for Artists for my band.
4.  Add the “checking” function below to your `~/.bashrc`. (Sorry, fish/zsh/nushell/powershell users. I’m sure you’re smart enough to convert this stupid-simple function to your shell scripting language of choice!)
5.  Run `source ~/.bashrc`.
6.  Optional: to avoid entering your password every time, edit your sudoers file (with `sudo visudo /etc/sudoers`) and add two lines to the end: `user ALL=NOPASSWD: /usr/bin/ln -sf /etc/hosts-checking /etc/hosts` and `user ALL=NOPASSWD: /usr/bin/ln -sf /etc/hosts-doing /etc/hosts` (replacing "user" with your username).

Here’s how to use it:

```
checking 15m 
```

You can pass the duration in any unit the sleep function accepts (`checking 12s`, `checking 1hr`, etc)

Here’s the thing itself:

```
function checking () {
    #set -x
    #set -o errexit
    export DURATION="${1:-1m}"
    echo "Starting a Checking session lasting ${DURATION}"
    sudo ln -sf /etc/hosts-checking /etc/hosts && resolvectl flush-caches
    echo "Distracting websites and comms unblocked!"
    sleep "$DURATION"
    notify-send "Checking session complete! Close your tabs."
    sleep 1m
    sudo ln -sf /etc/hosts-doing /etc/hosts && resolvectl flush-caches
    echo "Done"
    trap "sudo ln -sf /etc/hosts-doing /etc/hosts" INT # Handle Ctrl+C
    #systemctl suspend
} 
```

It’s so simple, it’s essentially self-documenting.

Notice the 1-minute grace period, with a reminder that it’s time to wrap things up.

The optional suspend at the end is just to drive the point home, and give a moment to clear the brain. (If I actually pay attention and stop when the notification pops up, I can hit `Ctrl+C`.)

Here are some ideas of what you can block in your `hosts-doing`:

1.  News sites
2.  Analytics and dashboards
3.  Feed readers
4.  Social media
5.  Email and Chat

```
127.0.0.1       news.ycombinator.com lobste.rs  
127.0.0.1       plausible.io googleanalytics.com goatcounter.com umami.is usefathom.com artists.spotify.com  
127.0.0.1       commafeed.com feedly.com inoreader.com
127.0.0.1       reddit.com facebook.com x.com instagram.com bsky.social threads.net craigslist.org
127.0.0.1       gmail.com slack.com discord.com 
```

This is just an example. I only bother blocking services I (over) use, and I put one site per line for readability.

Obviously, don’t block anything you need for your real work.

Also, to borrow Tiny Tiny RSS’s charming disclaimer^([1](https://tt-rss.org/?ref=autodidacts.io)): “There’s no warranty. If it breaks you get to keep both parts.”