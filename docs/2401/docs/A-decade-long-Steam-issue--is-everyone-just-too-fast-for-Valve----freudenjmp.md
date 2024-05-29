<!--yml
category: 未分类
date: 2024-05-27 14:45:35
-->

# A decade long Steam issue, is everyone just too fast for Valve? | freudenjmp

> 来源：[https://blog.freudenjmp.com/posts/no-user-logon/](https://blog.freudenjmp.com/posts/no-user-logon/)

Valve has shown to occasionally not act on community feedback and bug reports. A story about a decade old bug.

#### tl;dr[](#tldr)

*To fix the infamous `No user logon` issue in Counter-Strike that existed for over a decade, wait 10 seconds in main menu after starting CS2, so Steam can properly validate your Steam ID.*

*Here are a few popular mitigations that do absolutely jack shit to fix the root cause. If you’ve found this article via Google: STOP, DO NOT DO THESE:*

*   *Reinstall the game*
*   *Validate the game files*
*   *Restart Steam*
*   *Restart your computer*
*   *Disable WiFi*

*To learn what to do instead, read this article.*

## *Introduction[](#introduction)*

***(You can skip to [No user logon](#no-user-logon) if you don’t care about the history of Counter-Strike or you can skip to [Solution](#solution) if you don’t care about the technical details/root cause.)***

**[Counter-Strike](https://store.steampowered.com/app/730/CounterStrike_2/) is a famous game developed by [Valve](https://www.valvesoftware.com/en/). Recently, Counter-Strike 2 (CS2) has been published and has *replaced* (!) its predecessor Counter-Strike: Global Offensive (CS:GO). Some people tend to not consider CS2 a game but one big bug. While this may be a bit harsh, it’s not far from the truth:**

**Be prepared for new bugs appearing, as Valve didn’t manage to include CS2 in [Valves HackerOne bug bounty program](https://hackerone.com/valve). `cs2.exe` is not in scope and the only (outdated) mention of CS2 is inside their description. This means that Valve is not paying for bug reports related to CS2.**

> **Effective 6/14/2023 10 AM PDT, CS:GO is out-of-scope for new reports. Reports for CS2 Limited Test are currently out-of-scope.**

**However, this of course didn’t stop us from reporting bugs to Valve via e-mail. I hope you are not surprised to learn that these bugs have neither been fixed nor did they bother to reply. We have found at least one critical zero-day that compromises the competitive integrity of the game which is yet under the [industry-standard 90-day responsible disclosure time frame](https://versprite.com/blog/what-is-responsible-disclosure/).**

**Notable is that CS2 *replaced* CS:GO on September 27, 2023, which is a novum. This means CS:GO can’t be played anymore by the average user. Such a hard cutoff date is a new level of “*forcing things on the community they maybe don’t like*”. Gamers and platforms such as my employer [Esportal](https://esportal.com) were suddenly forced to use CS2.**

**It’s not as if I never worked around bugs or needed to fix them on my own in CS:S or CS:GO. It is fine to a degree and expected as no software is bug free. But the amount of bugs in CS2 sets a new standard in a negative way. Especially bugs that are only relevant to community servers which Valve cares even less about.**

**Since middle of 2023, when CS2 release was apparent and we started to prepare support for it on Esportal, my working days look like this:**

```
**`journey
  section My working day before it even starts at 9 AM
    Enthusiastic morning: 7
    Read Slack: 5
    Yet another CS2 bug: 3
    Bug is 10 years old: 1`** 
```

## **No user logon[](#no-user-logon)**

***Bugs that are reported by the community for years and years are still not fixed and are now in CS2\. One of these bugs is the infamous `No user logon` disconnection happening randomly while you are happily playing:***

****The infamous “No user logon” disconnection mid-game****

***For years ([2008](https://www.techsupportforum.com/threads/counter-strike-source-no-steam-logon-error.214386/), [2011](https://www.elitepvpers.com/forum/counter-strike/1264745-no-steam-logon.html), [2013](https://www.reddit.com/r/GlobalOffensive/comments/1fct7v/no_steam_logon_problem_need_help/), [2014](https://www.reddit.com/r/GlobalOffensive/comments/21auhx/kicked_for_no_steam_logon/), [2017](https://steamcommunity.com/discussions/forum/1/143387886726485073/), [2017](https://www.reddit.com/r/GlobalOffensive/comments/79i69y/no_user_logon/), [2018](https://steamcommunity.com/app/730/discussions/0/1727575977593194982/), [2021](https://steamcommunity.com/app/730/discussions/0/4809273833195803820/), [2023](https://www.reddit.com/r/FACEITcom/comments/nz3lmq/no_user_logon_problem_how_to_fix/), [2023](https://www.reddit.com/r/csgo/comments/16k7ioj/no_user_logon/)) people around the world have been reporting this issue in multiple forums, including the official support forum from Valve. Note that from a technical perspective `No user logon` and the sometimes mentioned `No steam logon` are likely just different names for the same root cause. I can’t know for sure, but it would make sense.***

***There are countless *proposed fixes* to the issue:***

***Spoiler: None of these *proposed fixes* actually fix the issue. They are just randomly stopping the root cause from kicking in by pure coincidence.***

```
***`pie title Fixing "No user logon"
  "Think they fixed it" : 100
  "Actually fixed it" : 0`*** 
```

### ***Esportal specific[](#esportal-specific)***

***We were facing this issue on Esportal as well throughout the years. While the problem was not significant usually it was a constant companion that cost us several days to debug and mitigate it if one counts all the efforts made in the past. Yet we never really fixed the problem and only were able to mitigate so it does not happen often.***

*   ***CS:GO: *(all times CET)*

    *   First occurrence: 2019-11-15 19:15:32 (start of data information)
    *   Last occurrence: 2023-09-26 21:38:01 (the day before the CS:GO got replaced by CS2)*** 

***In CS2 we were quite happy as apparently the issues was gone. We didn’t see any reports from our users and we didn’t see any occurrences in our logs. We were happy until the first week of January 2024 when we observed an increase of user reports related to this issue compared to all reports:***

```
***`xychart-beta
  title "Relative amount of 'No user logon' tickets"
  x-axis [30.12., 31.12., 01.01., 02.01., 03.01., 04.01., 05.01., 06.01., 07.01., 08.01., 09.01.]
  y-axis "Share of daily tickets (in %)" 0 --> 23
  bar [0.5, 0, 0, 0, 6, 10, 18, 18, 23, 10, 9]`*** 
```

***This is significant, I started to investigate. Coincidentally, these are the time of the day when we received the reports:***

*   ***CS2: *(all times CET)*

    *   First occurrence: 2023-12-30 19:45:15 (and the only one that day)
    *   2023-12-31: no reports
    *   2024-01-01: no reports
    *   2024-01-02: no reports
    *   2024-01-03: 15:19:30 - 15:38:26
    *   2024-01-04: 14:27:12 - 15:57:24
    *   2024-01-05: 14:00:57 - 17:13:36
    *   2024-01-06: 13:01:55 - 17:35:12
    *   2024-01-07: 12:56:30 - 17:44:09
    *   2024-01-08: 14:15:06 - 16:41:57*** 

***My colleague connected the dots and wrote in Slack:***

> ***~13-17 PM CET is 04-08 AM in Washington (where Valve is operating from), maybe they are running some kind of routine during their night
> — Jane Doe, Jan 8th, 2024, colorized***

***Previously the reported issues were evenly distributed throughout the day. Now they are concentrated in a specific time frame.***

***We could confirm that players world wide are facing the issue outside of Esportal and thought it indeed is caused by some maintenance in the middle of the night in Washington.***

***Though I was thinking “I looked into this bug a hundred times already without a meaningful result” we were now incentivized to fix it once and for all given the significance of the problem and the volume of tickets.***

## ***The symptoms[](#the-symptoms)***

***The observed `No user logon` errors happened 2-3 minutes after the player connected to the game. It was interesting to know, as this time span was pretty constant.***

***Luckily, a colleague mentioned, without knowing what important information they were sitting on at that time:***

> ***I don’t see my skins in CS2 until some minutes into the game.***

***Indeed, [players outside of Esportal reported missing skins](https://www.reddit.com/r/GlobalOffensive/comments/190s7am/gaben_stole_our_usps/) as well.***

****My colleague incepting an idea into my head without them knowing.****

***Boom! Was this the missing piece of the puzzle? The one information that was required to connect the dots which wasn’t apparent for years? I felt like I was in the movie [Inception](https://www.imdb.com/title/tt1375666/) and my colleague being [Leonardo DiCaprio](https://www.imdb.com/name/nm0000138/) incepting an idea into my head.***

***It is well-known that Valve is sensitive when it comes to skins in their games. They make substantial amount of money with skins. So what if individual skins are not shown until maybe, just maybe, the player owning the skin has been properly authorized by Steam? This would make sure people can’t spoof their identity on the gameserver and use skins they don’t own.***

## ***Validation of the hypothesis[](#validation-of-the-hypothesis)***

***I picked a random match where a report came in. Matches are played 5vs5, so 10 players connect to a gameserver in total. Looking at the gameserver logs and applying proper filtering I was able to immediately see something that supports the hypothesis: `STEAM USERID validated` for a given user who didn’t have issues with `No user logon` appeared roughly 1min20s after the user connected. Example with user `Alice`:***

 ***`|  
```
1
2
3

```

 |  
```
16:39:55: "Alice<1><>" connected
16:41:14: "Alice<1><>" STEAM USERID validated
17:17:32: "Alice<1><CT>" disconnected (reason "NETWORK_DISCONNECT_DISCONNECT_BY_USER")

```

 |`*** 

***In older logs before 3rd of January the Steam validation always finished within 2-3 seconds after connection instead.***

***I tested myself and indeed: missing skins were only a thing during night time in Washington. I was able to reproduce the issue by connecting to a gameserver at 5 AM Washington time. I was not able to reproduce the issue during the Washington daytime.***

> ***Hypothesis that `No user logon` and no skins are connected is now confirmed.***

## ***NETWORK_DISCONNECT_STEAM_LOGON[](#network_disconnect_steam_logon)***

***Now it is apparent this issue has something to do with Steam validation. But what exactly is the bug? Coming back to the logs of the match I picked I now looked at user `Bob` who reported `No user logon` issues:***

 ***`|  
```
1
2
3
4
5
6
7
8
9
10
11
12
13

```

 |  
```
16:40:03: "Bob<6><>" connected
16:40:08: "Bob<6><Unassigned>" disconnected (reason "NETWORK_DISCONNECT_LOOPSHUTDOWN")
16:40:13: "Bob<6><>" connected
16:43:02: STEAMAUTH: Client Bob received failure code 8
16:43:02: "Bob<6><TERRORIST>" disconnected (reason "NETWORK_DISCONNECT_STEAM_LOGON")
16:43:53: "Bob<6><>" connected
16:43:58: "Bob<6><TERRORIST>" disconnected (reason "NETWORK_DISCONNECT_LOOPSHUTDOWN")
16:44:04: "Bob<6><>" connected
16:46:59: STEAMAUTH: Client Bob received failure code 8
16:46:59: "Bob<6><TERRORIST>" disconnected (reason "NETWORK_DISCONNECT_STEAM_LOGON")
16:47:38: "Bob<6><>" connected
16:49:09: "Bob<6><>" STEAM USERID validated
17:17:32: "Bob<6><CT>" disconnected (reason "NETWORK_DISCONNECT_EXITING")

```

 |`*** 

***It is apparent it took 9 minutes for this user to get into the match until they received `STEAM USERID validated`.***

***After the match ended, the disconnect reason is `NETWORK_DISCONNECT_EXITING`. It’s an internal identifiers of the [Source 2 engine](https://en.wikipedia.org/wiki/Source_2) which CS2 is using. The prefix `NETWORK_DISCONNECT_` is used to separate these identifiers from others in the engine.***

***`NETWORK_DISCONNECT_EXITING` is a valid disconnect reason tells us the user closed their CS2 game while being connected to the gameserver. Another natural reason would be `NETWORK_DISCONNECT_DISCONNECT_BY_USER` which means the user disconnected from the gameserver by themselves without closing their CS2 game.***

***Looking at the events before the Steam validation was successful, four disconnections are apparent with two different reasons:***

*   ***`NETWORK_DISCONNECT_LOOPSHUTDOWN`***
*   ***`NETWORK_DISCONNECT_STEAM_LOGON`***

***I can reasonably assume the message `No user logon` is the human translation of the identifier `NETWORK_DISCONNECT_STEAM_LOGON`, so I started looking into this first. Interesting is that `NETWORK_DISCONNECT_STEAM_LOGON` is an immediate consequence of `STEAMAUTH: Client Bob received failure code 8`. What does it mean though? To find out, I searched for the string `STEAMAUTH:` in the binary file that is providing the core of the Source 2 engine, `libengine2.so`:***

 ***`|  
```
1
2

```

 |  
```
$ grep -a "STEAMAUTH:" engine2.so
-insecure-insecure_forced_by_launchersystem/networkSTEAMAUTH: Client %s received failure code %d

```

 |`*** 

***The last part of the match is interesting: `STEAMAUTH: Client %s received failure code %d`. As I confirmed this string is in `libengine2.so`, I opened the file in a [reverse engineering](https://en.wikipedia.org/wiki/Reverse_engineering) tool, searched for the same string again and found the function which is referencing the string. This helps me to understand the context of the string and what it is used for.***

***A little trick that comes handy when looking at Counter-Strike is the [CS:GO source code leak](https://www.ign.com/articles/valve-counter-strike-source-code-leak-no-danger). The same string appears in the file [sv_steamauth.cpp on line 792](https://github.com/perilouswithadollarsign/cstrike15_src/blob/f82112a2388b841d72cb62ca48ab1846dfcc11c8/engine/sv_steamauth.cpp#L792). Utilizing a [decompiler](https://en.wikipedia.org/wiki/Decompiler) I was able to get pseudo C code that reflects what the function is doing in CS2\. Combining the information from the source code leak and the reverse engineering tool I was able to understand the context of the string and what it is used for. The functions look nearly identical in both versions of Counter-Strike, but not 100%. Here is what the properly readable code for CS2 looks like:***

 ***`|  
```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39

```

 |  
```
void CSteam3Server::OnValidateAuthTicketResponseHelper(CSteam3Server* pThis, CBaseClient *cl, EAuthSessionResponse eAuthSessionResponse)
{
  Warning("STEAMAUTH: Client %s received failure code %d\n", cl->GetClientName(), eAuthSessionResponse);
  g_Log.Printf("STEAMAUTH: Client %s received failure code %d\n", cl->GetClientName(), eAuthSessionResponse);
  switch (eAuthSessionResponse)
  {
    case 1: // k_EAuthSessionResponseUserNotConnectedToSteam
      if (!client[2506])
        cl->Disconnect(NETWORK_DISCONNECT_STEAM_LOGON);
      break;
    case 2: // k_EAuthSessionResponseNoLicenseOrExpired
      cl->Disconnect(NETWORK_DISCONNECT_STEAM_OWNERSHIP);
      break;
    case 3: // k_EAuthSessionResponseVACBanned
    case 9: // k_EAuthSessionResponsePublisherIssuedBan
      if (!BLanOnly())
        cl->Disconnect(NETWORK_DISCONNECT_STEAM_VACBANSTATE);
      break;
    case 4: // k_EAuthSessionResponseLoggedInElseWhere
      if (!client[2506] && !BLanOnly())
        cl->Disconnect(NETWORK_DISCONNECT_STEAM_LOGGED_IN_ELSEWHERE);
      break;
    case 5: // k_EAuthSessionResponseVACCheckTimedOut
      cl->Disconnect(NETWORK_DISCONNECT_STEAM_VAC_CHECK_TIMEDOUT);
      break;
    case 6: // k_EAuthSessionResponseAuthTicketCanceled
      if (!BLanOnly())
        sub_2F3900(client);
      break;
    case 7: // k_EAuthSessionResponseAuthTicketInvalidAlreadyUsed
    case 8: // k_EAuthSessionResponseAuthTicketInvalid
      if (!BLanOnly())
        cl->Disconnect(NETWORK_DISCONNECT_STEAM_LOGON);
      break;
    default:
      cl->Disconnect(NETWORK_DISCONNECT_STEAM_DROPPED);
      break;
  }
}

```

 |`*** 

***Leaving other cases aside as I am only interested in failure code `8` at the moment, this function disconnects the user from the gameserver with the beloved `NETWORK_DISCONNECT_STEAM_LOGON` reason. It happens when something called `CSteamServer3` tells us that the `eAuthSessionResponse` is one of:***

*   ***`k_EAuthSessionResponseUserNotConnectedToSteam` (maps to `1`)***
*   ***`k_EAuthSessionResponseAuthTicketInvalidAlreadyUsed` (maps to `7`)***
*   ***`k_EAuthSessionResponseAuthTicketInvalid` (maps to `8`)***

## ***Steam3 validation:[](#steam3-validation)***

***So… what is `CSteamServer3`? I don’t know for sure, but I can deduce from what I learned looking at the leaked CS:GO source code. Saving you from following me through the jungle of C++ classes and functions, here is what I found out as diagram followed by a textual explanation:***

```
***`sequenceDiagram
  box User computer
    participant CS2.exe
  end
  box Internet
    participant Gameserver
    participant Steam3 server
  end
  CS2.exe->>Gameserver: Connect with untrusted Steam ID
  Gameserver->>Steam3 server: Is that Steam ID valid?
  Note over CS2.exe: Can continue playing<br/> while the validation<br/>with Steam3 is pending
  alt
    Steam3 server->>Gameserver: yes
    Gameserver->>CS2.exe: Assign skins, trust user
  else
    Steam3 server->>Gameserver: no
    Gameserver->>CS2.exe: Disconnect with "No user logon"
  end`*** 
```

***Steam3 server is likely responsible for authenticating users. It is not the same as the Steam client that you are using to play games. It is also not the same as the Steam servers that are responsible for matchmaking and other things. It is a separate thing. Think: it makes sure to proof that you are who you are claiming to be and that you own the game you are trying to play.***

***When connecting to a gameserver, your game (`CS2.exe`) tells the gameserver what your Steam ID is. As you could be a bad person, you could modify the game and tell the gameserver that your Steam ID is `STEAM_0:0:0` instead. That would be the Steam ID of the first Steam account ever created. The gameserver has no way to know at this point whether your information is correct or not. This is where the Steam3 server comes into play. The gameserver asks the Steam3 server if the Steam ID is valid and whether you own the game.***

***Note that while the gameserver is waiting for the response, you can continue playing normally on the gameserver, but without skins.***

***If the Steam3 server says `yes`, the gameserver starts trusting that information. It can now do interesting stuff with it, like starting to assign your personal skins to you. This would be the moment where you and others can see your skins.***

***And now we know what is likely happening in Washington at night between 4 and 8 AM: the Steam3 server is doing maintenance or is very slow to respond for other reasons. As we learned, that “slowness” is currently roughly 1min20s.***

***If the Steam3 server says `no` instead, the gameserver knows that you (or your game) is lying and disconnects you with `NETWORK_DISCONNECT_STEAM_LOGON`. Looking at the logs from the match, the disconnect happens 2min50s seconds after the user connected to the gameserver. This is why users report the `No user logon` message appears after 2-3 minutes. This is a reasonable timeframe because in networks it’s always a good idea to have enough leeway for possible issues. It’s better to wait a bit longer than to disconnect too early.***

## ***Making it trustable[](#making-it-trustable)***

***The above scenario is not fully complete, though. It imposes one problem: it only guarantees that the Steam ID told to the gameserver is valid and owns the game. It does *not* proof that the instance of `CS2.exe` that connected to the gameserver is actually the one belonging to the signed in Steam account running on the same machine via `Steam.exe`. So what is needed is a way to proof that `CS2.exe` is actually trustable.***

***This can be achieved by ensuring that `CS2.exe` can initiate a connection to Steam3 via `Steam.exe` running on the same machine. As `Steam.exe` as the Steam client knows the account which is currently signed in, it compares the Steam ID sent by `CS2.exe` to it. If they match, `Steam.exe` tells the Steam3 server that the Steam ID is valid, for `CS2.exe`. If it can’t match them, it doesn’t tell the Steam3 server about it.***

```
***`sequenceDiagram
  box User computer
    participant Steam.exe
    participant CS2.exe
  end
  box Internet
    participant Gameserver
    participant Steam3 server
  end
  CS2.exe-->>+Steam.exe: Connect with untrusted Steam ID
  Note over Steam.exe: Confirms that Steam ID<br/>sent by CS2.exe is the<br/>same as the currently<br/>signed in Steam account
  Steam.exe-->>-Steam3 server: Temporarily store info that Steam ID is valid for game CS2
  CS2.exe->>Gameserver: Connect with untrusted Steam ID
  Gameserver->>Steam3 server: Is that Steam ID valid?
  Note over CS2.exe: Can continue playing<br/> while the validation<br/>with Steam3 is pending
  alt
    Steam3 server->>Gameserver: yes
    Gameserver->>CS2.exe: Assign skins, trust user
  else
    Steam3 server->>Gameserver: no
    Gameserver->>CS2.exe: Disconnect with "No user logon"
  end`*** 
```

> ***Learning: In reality though, the answer `no` is too simple. Deducing from above, it can be reasonably assumed that the answer `k_EAuthSessionResponseAuthTicketInvalid` (which maps to `8`) is the one we see in the logs for a failed Steam3 validation: `STEAMAUTH: Client Bob received failure code 8`.***

***Reasons why Steam3 server could answer `no` are for example:***

1.  ***Steam3 is broken and answers wrong things: unlikely, even Valve can be trusted to get this right***
2.  ***Steam3 server is under maintenance: unlikely, as other users can connect***
3.  ***The Steam ID is not valid: unlikely, usually our players are trustworthy***
4.  ***The Steam ID is valid, but the instance of `CS2.exe` is not trustable: likely***
5.  ***Steam3 server doesn’t know about the Steam ID for this `CS2.exe` instance: likely***

***Aha! The last two points are likely candidates for the issue. But why do some users have problems with untrusted `CS2.exe` instances while other users are fine? And why was it possible that user `Bob` was able to validate their Steam ID after 9 minutes?***

## ***NETWORK_DISCONNECT_LOOPSHUTDOWN[](#network_disconnect_loopshutdown)***

***The other disconnect reason `NETWORK_DISCONNECT_LOOPSHUTDOWN` is still unclear. It’s appearing first and then the user connects again, eventually getting disconnected with `NETWORK_DISCONNECT_STEAM_LOGON`.***

 ***`|  
```
1
2
3
4
5

```

 |  
```
16:40:03: "Bob<6><>" connected
16:40:08: "Bob<6><Unassigned>" disconnected (reason "NETWORK_DISCONNECT_LOOPSHUTDOWN")
16:40:13: "Bob<6><>" connected
16:43:02: STEAMAUTH: Client Bob received failure code 8
16:43:02: "Bob<6><TERRORIST>" disconnected (reason "NETWORK_DISCONNECT_STEAM_LOGON")

```

 |`*** 

***It seems like the gameserver disconnects the user with `NETWORK_DISCONNECT_LOOPSHUTDOWN`. Then the game connects itself again. This is a feature of the game and it happens automatically after 5 seconds to retry the connection, hence the multiple connection attempts visible in the gameserver logs.***

***Again: why are some users getting disconnected with `NETWORK_DISCONNECT_LOOPSHUTDOWN` while others aren’t?***

## ***Loops in the Source 2 engine[](#loops-in-the-source-2-engine)***

***A loop in terms of software engineering is something that is executed until a specific goal is reached.***

***The Source 2 engine has one active loop at a time. When `CS2.exe` starts, it is started with an active loop. A loop processes things in the background, waits until they are finished and once they are finished, invokes the next loop. A loop can run for a very long time, for example the `game` loop runs as long as you are playing. A loop can be shutdown in case something goes wrong.***

***In this example, a the `levelload_loop` is executed until the level, materials and physics engine are loaded. Once they are loaded, the `levelload_loop` is considered finished and it executes the `game_loop`:***

 ***`|  
```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21

```

 |  
```
void levelload_loop()
{
  bool loop_done = false;

  start_load_of_level_in_background();
  start_load_of_materials_in_background();
  start_load_of_physics_engine_in_background();

  while (!loop_done)
  {
    process_user_input_like_mouse_and_keyboard();
    draw_user_interface_to_screen();

    if (level_loaded && materials_loaded && physics_engine_loaded)
    {
      loop_done = true;
    }
  }

  game_loop();
}

```

 |`*** 

***A valid shutdown reason for this loop would be missing material required for the level.***

## ***The disconnection[](#the-disconnection)***

***So if a loop can be shutdown, and the disconnect reason is `NETWORK_DISCONNECT_LOOPSHUTDOWN`…***

***Yes, I hear you thinking. This means that the first disconnection is initiated by `CS2.exe` because some loop is shutdown. The user is not disconnected by the gameserver but by themselves.***

***No wonder the root cause hasn’t been found in so many years: it’s not a bug in the gameserver, it’s a bug in the game.***

****Me at that very moment: the famous “WASSS” by twitch streamer [ohnePixel](https://www.twitch.tv/ohnepixel)
The 175+ shows his and my heartbeat rate because I searched in the wrong place for years****

***So the search of the root cause has to continue in `CS2.exe` and not on the gameserver side.***

> ***Learning: the first disconnection before a `No user logon` comes from `CS2.exe` and not from the gameserver.***

## ***CS2 startup procedure[](#cs2-startup-procedure)***

***When `CS2.exe` is started it executes various loops from the Source 2 engine. The final loop is the `game` loop which is responsible for the actual menu interaction, user interaction and gameplay. It is *the* loop that is running as long as the game is fully loaded and/or you are playing.***

***As `game` is the final loop it can’t be the one which is shut down and so it can’t be the one initiating the disconnection with `NETWORK_DISCONNECT_LOOPSHUTDOWN` as otherwise the game would end. To find out which loops are executed before the `game` loop, I decided to look at the output of the game console of `CS2.exe`, which can be enabled in the game settings.***

***Among a lot of other things, the following can be observed:***

 ***`|  
```
1
2
3
4

```

 |  
```
[SteamNetSockets] SDR RelayNetworkStatus:  avail=Attempting  config=OK  anyrelay=Attempting   (Performing ping measurement)
[SteamNetSockets] AuthStatus (steamid:<redacted>):  OK  (OK)
[Client] CL:  CLoopModeLevelLoad::MaybeSwitchToGameLoop switching to "game" loopmode with addons ()
[EngineServiceManager] SwitchToLoop game requested:  id [1] addons []

```

 |`*** 

***It appears there is a loop `levelload` (look at `CLoopModeLevelLoad`) which is executed before the `game` loop. In the end, that loop wants to transition to the `game` loop. And *directly* before that transition it says `AuthStatus (steamid:<redacted>): OK (OK)`. So the last thing that happens before the `game` loop is started is that the Steam ID validation is initiated.***

***It all makes sense: `levelload` is called on startup of `CS2.exe`. While it’s not loading an actual map you are playing on, the initial screen you see in CS2 is likely considered a level too and initializing things like the intro video and the main menu. And while that initial screen is loaded in `levelload` it tries to initiate the validation with Steam3 via `Steam.exe`.***

> ***Learning: `levelload` is an important initialization loop and is, among other things, responsible to initiate the validation with Steam3 via `Steam.exe` from `CS2.exe` as the **last thing** this loop does.***

***The updated diagram with these learnings looks a bit different now. Note that `CS2.exe` is only considered the “initiator” of the `levelload` and all the game logic that was previously assigned to `CS2.exe` is taken over by the `game` loop:***

```
***`sequenceDiagram
  box User computer
    participant Steam.exe
    participant CS2.exe
    participant "levelload" loop
    participant "game" loop
  end
  box Internet
    participant Gameserver
    participant Steam3 server
  end
  CS2.exe->>"levelload" loop: Starts
  "levelload" loop-->>+Steam.exe: Connect with untrusted Steam ID
  "levelload" loop->>"game" loop: Starts
  Note over Steam.exe: Confirms that Steam ID<br/>sent by CS2.exe is the<br/>same as the currently<br/>signed in Steam account
  Steam.exe-->>-Steam3 server: Temporarily store info that Steam ID is valid for game CS2
  "game" loop->>Gameserver: Connect with untrusted Steam ID
  Gameserver->>Steam3 server: Is that Steam ID valid?
  Note over "game" loop: Can continue playing<br/> while the validation<br/>with Steam3 is pending
  alt
    Steam3 server->>Gameserver: yes
    Gameserver->>"game" loop: Assign skins, trust user
  else
    Steam3 server->>Gameserver: no
    Gameserver->>"game" loop: Disconnect with "No user logon"
  end`*** 
```

## ***The bug[](#the-bug)***

***Deducing from above: CS2 is only fully initialized **after** the `levelload` loop successfully completes. The game can’t be considered to be in a usable state when it fails to complete.***

> ***Learning: When the initialization of `levelload` is incomplete (speak: CS2 has not been fully loaded/initialized), the Steam3 validation is never initiated because it is the last thing that loop wants to do. This `CS2.exe` is now broken and can’t be fixed until a simple restart of `CS2.exe`.***

***Putting everything together, the bug is now apparent:***

> ***`NETWORK_DISCONNECT_LOOPSHUTDOWN` is caused by premature shutdown of `levelload` without initiating the Steam3 validation. Because `levelload` detects the premature shutdown, it disconnects the user from the gameserver every time a reconnection is tested.***

***The diagram looks like this when the bug is triggered:***

```
***`sequenceDiagram
  box User computer
    participant CS2.exe
    participant "levelload" loop
    participant "game" loop
  end
  box Internet
    participant Gameserver
    participant Steam3 server
  end
  CS2.exe->>"levelload" loop: Starts
  "levelload" loop->>"game" loop: Starts
  "game" loop->>Gameserver: Connect with untrusted Steam ID
  Gameserver->>Steam3 server: Is that Steam ID valid?
  Note over "game" loop: Can continue playing for 2min50s max
  Steam3 server->>Gameserver: no
  Gameserver->>"game" loop: Disconnect with "No user logon"`*** 
```

****Note: while all of the examples, names and explanations here based on CS2, the bug is equivalent in CS:GO and even Counter-Strike: Source, though the technical names and means are different because those games are based on an older version of the Source engine.****

## ***Bug invocation[](#bug-invocation)***

***But why is `levelload` prematurely shutdown and why is it only happening to some users?***

***Remember the multitude of people who thought they have fixed the issue? They didn’t, they were just lucky because the bug invocation is a [race condition](https://en.wikipedia.org/wiki/Race_condition) when CS2 is started in a specific way! Race condition means that the behavior of an application is dependent on timing, and that’s what happens here.***

***The `levelload` loop is shut down before it can complete sometimes. And it’s only happening to some users because it depends on the following factors:***

1.  *****The way Counter-Strike is started**: the blame factor is 90%***
2.  ***Speed of the computer: blame factor is 3%***
3.  ***Speed of the user: blame factor is 3%***
4.  ***State of the moon: blame factor is 3%***
5.  ***Something is actually wrong with the user configuration: blame factor is 1%. Sadly I couldn’t investigate further here.***

***And what would be the way to start CS2 so the bug triggers with 99% certainty?***

> ***Triggering the bug with 99% certainty: connect to a gameserver without CS2 being open before.***

***And that’s why everyone is just too fast (to connect) for Valve:***

```
***`sequenceDiagram
  box User computer
    participant CS2.exe
    participant "levelload" loop
    participant "game" loop
  end
  box Internet
    participant Gameserver
    participant Steam3 server
  end
  CS2.exe->>"levelload" loop: Starts
  rect rgb(255, 0, 0)
    Note over "levelload" loop: THE BUG:<br/><br/>Loop shuts down prematurely<br/>because Steam.exe tells CS2.exe<br/>to connect to a gameserver<br/>too fast because you asked it to.
  end
  "levelload" loop->>"game" loop: Starts
  "game" loop->>Gameserver: Connect with untrusted Steam ID
  Gameserver->>Steam3 server: Is that Steam ID valid?
  Note over "game" loop: Can continue playing for 2min50s max
  Steam3 server->>Gameserver: no
  Gameserver->>"game" loop: Disconnect with "No user logon"`*** 
```

***There are multiple ways of doing that:***

*   ***Connecting from a server browser outside the game while it’s not running yet or has *just* been started before***
*   ***Connecting to a friend via Steam friends list outside the game while it’s not running yet or has *just* been started before***
*   ***Manually connecting to a server using Steams browser protocol (`steam://connect/127.0.0.1:27015`) outside the game while it’s not running yet or has *just* been started before***

***You can increase chances by starting CS2 and, before it has fully initialized, doing one of the above.***

## ***Solution[](#solution)***

*****Do not**:***

*   ***Reinstall the game***
*   ***Validate the game files***
*   ***Restart Steam***
*   ***Restart your computer***
*   ***Disable WiFi***
*   ***Connect from outside the game to a gameserver before it has been started***

*****Instead:*****

> ***Start CS2 well before you connect to a gameserver. It’s good enough to wait until you see the game console or wait 5-10 seconds after you saw the intro video.***

***If you want to be absolutely sure you are not affected by the bug, start CS2, open the game console and type `status`. Check the output for this line. If you see it, you are good to go:***

 ***`|  
```
1

```

 |  
```
[EngineServiceManager] @ Current  :  game

```

 |`*** 

## ***Conclusion[](#conclusion)***

***To close the question you are still asking yourself: it was possible for the user `Bob` to validate 9 minutes after the first try for just one reason: they restarted CS2 and weren’t unlucky that time.***

***Another conclusion is: once a Steam ID is validated, it will never fail later for that game instance except if you close your Steam client while `CS2.exe` is running.***

## ***Closing words[](#closing-words)***

***While I am sure the proposed solution works for 99% of all users, I am well aware that there is a very small minority of users who are affected by this error for other reasons not covered in this blog post.***

***Guess what the Esportal matchmaking platform did until we fixed the behavior on 10th of January, 2024: it started `CS2.exe` and as soon as the process was available (but not fully initialized) it executed the `steam://connect/<IP>:<Port>` command with the appropriate matchmaking gameserver. User tickets related to this issue stopping coming in the moment we applied the fix and deployed it to every Esportal player.***

***That said, I am ending the blog post with a graph that I really like and I do hope to never ever see a Slack message with that error again:***

```
***`xychart-beta
  title "Relative amount of 'No user logon' tickets"
  x-axis [03.01., 04.01., 05.01., 06.01., 07.01., 08.01., 09.01., 10.01., 11.01., 12.01.]
  y-axis "Share of daily tickets (in %)" 0 --> 23
  bar [6, 10, 18, 18, 23, 10, 9, 0, 0, 0]`*** 
```

***Social discussions:***

***Thanks to my colleagues for proof-reading and providing feedback!***

****Disclaimer: opinions my own and not the ones of my employer.****