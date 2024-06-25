<!--yml

category: 未分类

date: 2024-05-27 14:45:35

-->

# 十年长的Steam问题，是Valve只是对于社区反馈和错误报告太过迟缓？| freudenjmp

> 来源：[https://blog.freudenjmp.com/posts/no-user-logon/](https://blog.freudenjmp.com/posts/no-user-logon/)

Valve偶尔表现出对社区反馈和错误报告不予理睬的情况。一个关于十年前错误的故事。

#### tl;dr[](#tldr)

*为了修复《反恐精英》中存在十多年的臭名昭著的`无用户登录`问题，在启动CS2后的主菜单中等待10秒，以便Steam可以正确验证您的Steam ID。*

*以下是一些流行的缓解措施，绝对不能解决根本问题。如果你是通过谷歌找到这篇文章的：停止，不要做这些事情：*

+   *重新安装游戏*

+   *验证游戏文件*

+   *重新启动Steam*

+   *重新启动你的电脑*

+   *禁用WiFi*

*想要了解应该怎么做，请阅读这篇文章。*

## *简介[](#introduction)*

***(如果你不关心《反恐精英》的历史，可以跳到[无用户登录](#no-user-logon)，如果你不关心技术细节/根本原因，可以跳到[解决方案](#solution)。)***

**[反恐精英](https://store.steampowered.com/app/730/CounterStrike_2/)是由[Valve](https://www.valvesoftware.com/en/)开发的一款著名游戏。最近，反恐精英2（CS2）已经发布，并*取代*了它的前身反恐精英:全球攻势（CS:GO）。有些人倾向于不将CS2视为游戏，而将其视为一个大问题。虽然这可能有点苛刻，但也不无道理：**

**准备好出现新的错误，因为Valve没有将CS2包含在[Valve的HackerOne漏洞悬赏计划](https://hackerone.com/valve)中。`cs2.exe`不在范围内，关于CS2的唯一（已过时的）提及在于它们的描述中。这意味着Valve不支付与CS2相关的漏洞报告。**

> **自2023年6月14日上午10点（太平洋夏令时）起，CS:GO不再接受新报告。有关CS2有限测试的报告目前不在范围内。**

**然而，这当然没有阻止我们通过电子邮件向Valve报告错误。希望你不会惊讶地发现，这些错误既没有被修复，也没有得到他们的回复。我们至少发现了一个危及比赛完整性的关键零日漏洞，这仍然处于[行业标准的90天负责任披露时间框架](https://versprite.com/blog/what-is-responsible-disclosure/)之内。**

**值得注意的是，CS2于2023年9月27日取代了CS:GO，这是一个新现象。这意味着普通用户无法再玩CS:GO了。这样的强硬截止日期是一个新的“*强迫社区接受他们可能不喜欢的东西*”的程度。像我的雇主[Esportal](https://esportal.com)这样的游戏玩家和平台突然被迫使用CS2。**

**并不是说我从未解决过 CS:S 或 CS:GO 中的错误或需要自己修复它们。在某种程度上这是可以接受的，并且是预期的，因为没有软件是无错误的。但 CS2 中的错误数量在消极方面树立了新标准。尤其是那些只与社区服务器有关的 bug，Valve 对此更是漠不关心。**

**自 2023 年中旬以来，当 CS2 发布已成定局，我们开始为其在 Esportal 上提供支持时，我的工作日就是这样过的：**

```
**`journey
  section My working day before it even starts at 9 AM
    Enthusiastic morning: 7
    Read Slack: 5
    Yet another CS2 bug: 3
    Bug is 10 years old: 1`** 
```

## **无用户登录[](#no-user-logon)**

***社区多年来报告的漏洞仍然未被修复，现在出现在 CS2 上。其中一个问题是臭名昭著的“无用户登录”在你愉快玩耍时随机发生的断开连接：***

****臭名昭著的“无用户登录”在游戏中断开连接****

***多年来（[2008](https://www.techsupportforum.com/threads/counter-strike-source-no-steam-logon-error.214386/)，[2011](https://www.elitepvpers.com/forum/counter-strike/1264745-no-steam-logon.html)，[2013](https://www.reddit.com/r/GlobalOffensive/comments/1fct7v/no_steam_logon_problem_need_help/)，[2014](https://www.reddit.com/r/GlobalOffensive/comments/21auhx/kicked_for_no_steam_logon/)，[2017](https://steamcommunity.com/discussions/forum/1/143387886726485073/)，[2017](https://www.reddit.com/r/GlobalOffensive/comments/79i69y/no_user_logon/)，[2018](https://steamcommunity.com/app/730/discussions/0/1727575977593194982/)，[2021](https://steamcommunity.com/app/730/discussions/0/4809273833195803820/)，[2023](https://www.reddit.com/r/FACEITcom/comments/nz3lmq/no_user_logon_problem_how_to_fix/)，[2023](https://www.reddit.com/r/csgo/comments/16k7ioj/no_user_logon/)），世界各地的人们在多个论坛上报告了这个问题，包括 Valve 的官方支持论坛。请注意，从技术角度看，“无用户登录”和有时提到的“无 Steam 登录”很可能只是相同根本原因的不同名称。我不能确定，但这是有道理的。***

***有无数*提议的修复方法*：***

***剧透：这些*提议的修复方法*实际上都不能解决问题。它们只是纯粹偶然地阻止了根本原因的发生。***

```
***`pie title Fixing "No user logon"
  "Think they fixed it" : 100
  "Actually fixed it" : 0`*** 
```

### ***Esportal 特定[](#esportal-specific)***

***多年来，我们在 Esportal 上也面临着这个问题。虽然问题通常不太明显，但它是一个持续存在的伴侣，如果计算过去所做的所有努力，我们花费了几天的时间来调试和缓解它。然而，我们从未真正解决这个问题，只能缓解它，使其不经常发生。***

+   ***CS:GO：*(所有时间均为 CET)*

    +   第一次出现：2019-11-15 19:15:32（数据信息开始）***

    +   最后一次出现：2023-09-26 21:38:01（CS:GO 被 CS2 取代的前一天）***

***在 CS2 中，我们非常高兴，因为问题似乎已经消失了。我们没有看到任何来自用户的报告，也没有在日志中看到任何事件。我们一直很开心，直到2024年1月的第一周，我们观察到与所有报告相比与此问题相关的用户报告增加：***

```
***`xychart-beta
  title "Relative amount of 'No user logon' tickets"
  x-axis [30.12., 31.12., 01.01., 02.01., 03.01., 04.01., 05.01., 06.01., 07.01., 08.01., 09.01.]
  y-axis "Share of daily tickets (in %)" 0 --> 23
  bar [0.5, 0, 0, 0, 6, 10, 18, 18, 23, 10, 9]`*** 
```

***这很重要，我开始调查。巧合的是，这些正是我们收到报告的时间：***

+   ***CS2：*(所有时间均为 CET)*

    +   第一次出现：2023-12-30 19:45:15（当天唯一一次出现）

    +   2023-12-31: 没有报告

    +   2024-01-01: 没有报告

    +   2024-01-02: 没有报告

    +   2024-01-03: 15:19:30 - 15:38:26

    +   2024-01-04: 14:27:12 - 15:57:24

    +   2024-01-05: 14:00:57 - 17:13:36

    +   2024-01-06: 13:01:55 - 17:35:12

    +   2024-01-07: 12:56:30 - 17:44:09

    +   2024-01-08: 14:15:06 - 16:41:57***

***我的同事连接起点在 Slack 中写道：***

> ***中欧时间下午1点至5点是华盛顿时间上午4点至8点，也许他们正在进行某种常规工作***
> 
> — Jane Doe，2024年1月8日，上色***

***以前报告的问题在一天中均匀分布。现在它们集中在一个特定的时间范围内。***

***我们可以证实，世界各地的玩家都面临着 Esportal 之外的问题，我们认为这确实是由华盛顿夜间某种维护引起

***虽然我在想“我已经检查了这个 bug 一百次，但没有得到有意义的结果”，但由于问题的重要性和工单的数量，我们现在有了动力彻底解决它。***

## ***[症状](#the-symptoms)***

***观察到的`无用户登录`错误发生在玩家连接到游戏后的2-3分钟。有趣的是，这个时间跨度相当恒定。***

***幸运的是，一位同事在当时并不知道他们掌握了多么重要的信息：***

> ***我在 CS2 中直到游戏进行了一段时间才看到我的皮肤。***

***的确，[Esportal之外的玩家报告说丢失了皮肤](https://www.reddit.com/r/GlobalOffensive/comments/190s7am/gaben_stole_our_usps/)。***

****我的同事在没有知情的情况下将一个想法植入我的脑海中。****

***砰！这是缺失的拼图吗？连接这些年来看不见的拼图所需的信息？我感觉自己就像是电影[盗梦空间](https://www.imdb.com/title/tt1375666/)中的人物，而我的同事则是[莱昂纳多·迪卡普里奥](https://www.imdb.com/name/nm0000138/)，将一个想法植入我的脑海中。***

***众所周知，当涉及到他们游戏中的皮肤时，Valve 是很敏感的。他们通过皮肤赚取了大量的钱。所以如果个别皮肤在也许，只有当拥有皮肤的玩家经过了 Steam 的适当授权后，才会显示出来呢？这将确保人们不能在游戏服务器上伪造身份并使用他们不拥有的皮肤。***

## ***验证假设[](#validation-of-the-hypothesis)***

***我选择了一场报告中的随机比赛。比赛是 5v5 进行的，所以总共有 10 名玩家连接到游戏服务器。查看游戏服务器日志并应用适当的过滤后，我立即看到了支持假设的一些东西：对于一个没有`No user logon`问题的给定用户，`STEAM USERID validated`大约在用户连接后 1 分钟 20 秒后出现。以用户`Alice`为例：***

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

***在 1 月 3 日之前的旧日志中，Steam 验证总是在连接后的 2-3 秒内完成。***

***我自己进行了测试，确实：只有在华盛顿的夜间才会出现缺少的皮肤。我能够通过在华盛顿时间早上 5 点连接到游戏服务器来重现这个问题。在白天我无法重现这个问题。***

> ***`No user logon`和没有皮肤之间的联系的假设现在得到了确认。***

## ***[NETWORK_DISCONNECT_STEAM_LOGON](#network_disconnect_steam_logon)***

***现在显而易见的是，这个问题与 Steam 验证有关。但是到底是什么 Bug？回到我选择的比赛的日志，我现在看着报告了`No user logon`问题的用户 `Bob`：***

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

***显然，这个用户进入比赛直到收到`STEAM USERID validated`时花了 9 分钟。***

***比赛结束后，断开连接原因是`NETWORK_DISCONNECT_EXITING`。这是[Source 2 引擎](https://en.wikipedia.org/wiki/Source_2)内部标识符，CS2 正在使用它。前缀`NETWORK_DISCONNECT_`用于将这些标识符与引擎中的其他标识符分开。***

***`NETWORK_DISCONNECT_EXITING`是一个有效的断开连接原因，告诉我们用户在连接到游戏服务器时关闭了他们的 CS2 游戏。另一个自然的原因可能是`NETWORK_DISCONNECT_DISCONNECT_BY_USER`，这意味着用户在没有关闭 CS2 游戏的情况下自己断开了与游戏服务器的连接。***

***在 Steam 验证成功之前的事件中，有四次断开连接明显可见，有两种不同的原因：***

+   ***`NETWORK_DISCONNECT_LOOPSHUTDOWN`***

+   ***`NETWORK_DISCONNECT_STEAM_LOGON`***

***我可以合理地假设`No user logon`消息是标识符`NETWORK_DISCONNECT_STEAM_LOGON`的人类翻译，所以我首先开始研究这个。有趣的是，`NETWORK_DISCONNECT_STEAM_LOGON`是`STEAMAUTH: Client Bob received failure code 8`的直接结果。但是这意味着什么呢？为了找出答案，我在提供 Source 2 引擎核心的二进制文件`libengine2.so`中搜索字符串`STEAMAUTH：`：***

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

***比赛的最后部分很有趣：`STEAMAUTH: 客户端%s收到失败代码%d`。由于我确认了这个字符串在`libengine2.so`中，我打开了一个[逆向工程](https://zh.wikipedia.org/wiki/逆向工程)工具，再次搜索了相同的字符串，并找到了引用该字符串的函数。这帮助我理解了字符串的上下文以及它的用途。***

***在查看《反恐精英》时的一个小技巧是[CS:GO源代码泄漏](https://www.ign.com/articles/valve-counter-strike-source-code-leak-no-danger)。相同的字符串出现在[sv_steamauth.cpp文件的792行](https://github.com/perilouswithadollarsign/cstrike15_src/blob/f82112a2388b841d72cb62ca48ab1846dfcc11c8/engine/sv_steamauth.cpp#L792)中。利用[反编译器](https://zh.wikipedia.org/wiki/反编译器)我能够获得反映 CS2 中函数正在执行的伪 C 代码。结合源代码泄漏和逆向工程工具的信息，我能够理解字符串的上下文和用途。在两个版本的《反恐精英》中，函数看起来几乎相同，但并非百分之百相同。下面是 CS2 的可读代码：***

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

***暂时不考虑其他情况，因为我只对失败代码`8`感兴趣，这个函数将用户与游戏服务器断开连接，并显示`NETWORK_DISCONNECT_STEAM_LOGON`原因。当`CSteamServer3`告诉我们`eAuthSessionResponse`是以下之一时会发生这种情况：***

+   ***`k_EAuthSessionResponseUserNotConnectedToSteam` (映射为 `1`)***

+   ***`k_EAuthSessionResponseAuthTicketInvalidAlreadyUsed` (映射为 `7`)***

+   ***`k_EAuthSessionResponseAuthTicketInvalid` (映射为 `8`)***

## ***Steam3验证:[](#steam3-validation)***

***那么... `CSteamServer3`是什么？我不太确定，但我可以从查看泄露的《反恐精英：全球攻势》源代码中学到的东西推断出来。为了让你不必跟随我穿越 C++ 类和函数的丛林，这里是我发现的图表和文字说明：***

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

***Steam3服务器很可能负责验证用户。它与您用于玩游戏的 Steam 客户端不同。它也与负责匹配和其他事务的 Steam 服务器不同。它是一个独立的东西。想象一下：它确保您所声称的身份是您的，并且您拥有您想要玩的游戏。***

***当连接到游戏服务器时，你的游戏（`CS2.exe`）会告诉游戏服务器你的 Steam ID 是什么。由于你可能是个坏人，你可以修改游戏并告诉游戏服务器你的 Steam ID 是 `STEAM_0:0:0`。那将是第一个创建的 Steam 帐户的 Steam ID。在这一点上，游戏服务器无法知道你的信息是否正确。这就是 Steam3 服务器发挥作用的地方。游戏服务器询问 Steam3 服务器 Steam ID 是否有效以及你是否拥有该游戏。***

***请注意，在游戏服务器等待响应时，你可以继续在游戏服务器上正常游戏，但没有皮肤。***

***如果 Steam3 服务器回答“是”，游戏服务器开始信任该信息。它现在可以对其进行有趣的操作，比如开始为你分配你的个人皮肤。这将是你和其他人可以看到你的皮肤的时刻。***

***现在我们知道华盛顿在凌晨 4 点到 8 点之间可能发生的情况：Steam3 服务器正在进行维护或因其他原因响应非常慢。正如我们所了解的，这种“慢速”目前大约是 1 分 20 秒。***

***如果 Steam3 服务器回答“不”，游戏服务器就知道你（或你的游戏）在撒谎，并用 `NETWORK_DISCONNECT_STEAM_LOGON` 将你断开连接。从比赛日志中可以看到，用户连接到游戏服务器后的 2 分 50 秒后断开连接。这就是为什么用户报告称“2-3 分钟后出现无用户登录”消息的原因。这是一个合理的时间范围，因为在网络中，为可能的问题留有足够的余地总是个好主意。等待时间长一些总比过早断开连接要好。***

## ***使其可信[](#making-it-trustable)***

***上述情景还没有完全结束。它带来了一个问题：它只保证了告诉游戏服务器的 Steam ID 是有效的并且拥有该游戏。它*没有*证明连接到游戏服务器的 `CS2.exe` 实例实际上是属于通过 `Steam.exe` 在同一台机器上运行的登录的 Steam 帐户的 `CS2.exe`。因此，需要一种方法来证明 `CS2.exe` 实际上是可信的。***

***这可以通过确保 `CS2.exe` 可以通过在同一台机器上运行的 `Steam.exe` 向 Steam3 发起连接来实现。由于 Steam 客户端 `Steam.exe` 知道当前登录的帐户，因此它将由 `CS2.exe` 发送的 Steam ID 与之进行比较。如果它们匹配，`Steam.exe` 就告诉 Steam3 服务器该 Steam ID 对于 `CS2.exe` 是有效的。如果它们无法匹配，它就不会告诉 Steam3 服务器。***

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

> ***学习：尽管如此，现实中答案“不”太简单。从上面推断，可以合理地假设，失败的 Steam3 验证的日志中我们看到的答案 `k_EAuthSessionResponseAuthTicketInvalid`（映射为 `8`）是一个我们看到的失败的 Steam3 验证的日志：`STEAMAUTH: 客户端 Bob 收到了失败代码 8`。***

***Steam3 服务器可能回答“不”的原因包括：***

1.  ***Steam3损坏并回答错误的事情：不太可能，即使 Valve 也可以信任他们做对***

1.  ***Steam3 服务器正在维护中：不太可能，因为其他用户可以连接***

1.  ***Steam ID 无效：不太可能，通常我们的玩家是值得信任的***

1.  ***Steam ID是有效的，但`CS2.exe`的实例不可信：很可能***

1.  ***Steam3 服务器不知道这个`CS2.exe`实例的 Steam ID：很可能***

***啊哈！最后两点是问题的可能候选者。但是为什么一些用户在不可信的`CS2.exe`实例上遇到问题，而其他用户则没有问题呢？为什么用户`Bob`能在9分钟后验证他们的Steam ID呢？***

## ***NETWORK_DISCONNECT_LOOPSHUTDOWN[](#network_disconnect_loopshutdown)***

***另一个断开连接的原因`NETWORK_DISCONNECT_LOOPSHUTDOWN`仍然不清楚。它首先出现，然后用户再次连接，最终以`NETWORK_DISCONNECT_STEAM_LOGON`断开连接。***

***`|***

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

***似乎游戏服务器断开了用户连接，原因是`NETWORK_DISCONNECT_LOOPSHUTDOWN`。然后游戏自行重新连接。这是游戏的一个功能，它在5秒后自动重试连接，因此在游戏服务器日志中可以看到多次连接尝试。***

***再说一遍：为什么一些用户会因为`NETWORK_DISCONNECT_LOOPSHUTDOWN`而断开连接，而其他用户却没有？***

## ***Source 2 引擎中的循环[](#loops-in-the-source-2-engine)***

***在软件工程中，循环是指执行直到达到特定目标的东西。***

***Source 2 引擎一次只能执行一个活动循环。当`CS2.exe`启动时，它将以一个活动循环启动。循环在后台处理事情，等待它们完成，一旦它们完成，就会调用下一个循环。循环可以运行很长时间，例如`game`循环会在你玩游戏时持续运行。如果出现问题，循环可以关闭。***

***在这个例子中，`levelload_loop`会执行，直到关卡、材质和物理引擎加载完成。一旦它们加载完成，`levelload_loop`被视为完成，并执行`game_loop`：***

***`|***

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

***这个循环的有效关闭原因可能是关卡所需的材料丢失。***

## ***断开连接[](#the-disconnection)***

***所以如果一个循环可以被关闭，并且断开连接的原因是`NETWORK_DISCONNECT_LOOPSHUTDOWN`…***

***是的，我知道你在想什么。这意味着第一次断开连接是由`CS2.exe`发起的，因为某个循环被关闭。用户并非由游戏服务器而是由自己断开连接。***

***难怪这么多年来都没有找到根本原因：这不是游戏服务器的 bug，而是游戏的 bug。***

****我当时的心情：知名 twitch 主播 [ohnePixel](https://www.twitch.tv/ohnepixel) 的著名“WASSS”***

175+显示了他和我的心跳率，因为多年来我在错误的地方搜索****

***因此，根本原因的搜索必须在 `CS2.exe` 中继续进行，而不是在游戏服务器端。***

> ***教训：`No user logon` 前的第一次断开连接来自 `CS2.exe`，而不是游戏服务器端。***

## ***CS2 启动流程[](#cs2-startup-procedure)***

***当 `CS2.exe` 启动时，它执行 Source 2 引擎的各种循环。最后一个循环是 `game` 循环，负责实际菜单交互、用户交互和游戏过程。只要游戏完全加载和/或您正在玩游戏，它就会持续运行。***

***由于 `game` 是最终循环，它不能是被关闭的循环，因此也不能是启动与 `NETWORK_DISCONNECT_LOOPSHUTDOWN` 断开连接的循环，否则游戏将结束。为了找出在 `game` 循环之前执行的循环，我决定查看 `CS2.exe` 的游戏控制台输出，该控制台可以在游戏设置中启用。***

***除了许多其他事情外，还可以观察到以下情况：***

***`|***

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

***似乎存在一个循环 `levelload`（查看 `CLoopModeLevelLoad`），该循环在 `game` 循环之前执行。最后，该循环希望转到 `game` 循环。而在这个转换之前，它直接显示 `AuthStatus (steamid:<redacted>): OK (OK)`。因此，在启动 `game` 循环之前发生的最后一件事是启动了 Steam ID 验证。***

***这一切都说得通：`levelload` 在启动 `CS2.exe` 时被调用。虽然它没有加载您正在玩的实际地图，但是您在 CS2 中看到的初始屏幕很可能也被视为一个级别，并且正在初始化诸如引导视频和主菜单之类的内容。而在加载此初始屏幕时，`levelload` 尝试通过 `Steam.exe` 启动 Steam3 验证。***

> ***教训：`levelload` 是一个重要的初始化循环，负责通过 `CS2.exe` 向 Steam3 发起验证，作为该循环执行的**最后一件事情**。***

***根据这些经验教训更新的图表现在看起来有些不同。请注意 `CS2.exe` 只被视为 `levelload` 的“发起者”，而以前分配给 `CS2.exe` 的所有游戏逻辑都由 `game` 循环接管：***

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

## ***bug[](#the-bug)***

***由上述推断可知：只有在 `levelload` 循环成功完成后，CS2 才被完全初始化。当它未能完成时，游戏不能被认为处于可用状态。***

> ***教训：当 `levelload` 的初始化不完整时（即：CS2 尚未完全加载/初始化），Steam3 验证永远不会启动，因为这是该循环想要执行的最后一件事情。现在，`CS2.exe` 已经损坏，并且在重新启动 `CS2.exe` 之前无法修复。***

***将所有内容整合在一起，现在 bug 明显了：***

> ***`NETWORK_DISCONNECT_LOOPSHUTDOWN` 是由于 `levelload` 过早关闭而没有启动 Steam3 验证所致。因为 `levelload` 检测到了过早的关闭，所以每次重新连接测试时都会断开用户与游戏服务器的连接。***

***当触发 bug 时，图表如下：***

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

****注意：虽然所有的例子、名称和解释都是基于 CS2 的，但该 bug 在 CS:GO 和甚至 Counter-Strike: Source 中都是等价的，尽管技术名称和手段不同，因为这些游戏是基于较旧版本的 Source 引擎。****

## ***bug 调用[](#bug-invocation)***

***但为什么 `levelload` 会过早关闭，为什么只发生在某些用户身上？***

***记住那些以为已解决问题的人们？他们并没有，他们只是幸运，因为该 bug 被调用时是一个[竞争条件](https://en.wikipedia.org/wiki/Race_condition)，当 CS2 以特定方式启动时会发生这种情况！竞争条件意味着应用程序的行为取决于时间，这就是这里发生的情况。***

***`levelload` 循环有时会在完成之前被关闭。而且它只发生在某些用户身上，因为它取决于以下因素：***

1.  *****Counter-Strike 的启动方式**：责任因素为 90%***

1.  ***计算机速度：责任因素为 3%***

1.  ***用户速度：责任因素为 3%***

1.  ***月亮状态：责任因素为 3%***

1.  ***用户配置实际上有问题：责任因素为 1%。遗憾的是，我在这里无法进一步调查。***

***那么如何以 99% 的确率触发 CS2 的 bug 呢？***

> ***以 99% 的确率触发该 bug：在 CS2 打开之前连接到游戏服务器。***

***这就是为什么每个人都对 Valve 来说都太快（连接）了：***

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

***有多种方法可以做到这一点：***

+   ***在游戏未运行或刚刚启动时通过服务器浏览器连接***

+   ***在游戏未运行或刚刚启动时通过 Steam 好友列表连接到朋友***

+   ***在游戏未运行或刚刚启动时使用 Steam 浏览器协议 (`steam://connect/127.0.0.1:27015`) 手动连接到服务器***

***您可以通过启动 CS2 并在其完全初始化之前执行上述操作之一来增加机会。***

## ***解决方案[](#solution)***

*****请勿**：***

+   ***重新安装游戏***

+   ***验证游戏文件***

+   ***重新启动 Steam***

+   ***重新启动您的计算机***

+   ***禁用 WiFi***

+   ***在游戏启动之前从游戏服务器外部连接到游戏服务器***

*****相反：*****

> ***在连接到游戏服务器之前，请先启动 CS2。最好等到看到游戏控制台或在看到介绍视频后等待 5-10 秒。***

***如果你想绝对确定你没有受到这个 bug 的影响，启动 CS2，打开游戏控制台并输入 `status`。检查输出中的这一行。如果你看到了它，你就没问题：***

***`|

```
1

```

|

```
[EngineServiceManager] @ Current  :  game

```

|`***

## ***结论[](#结论)***

***为了结束你仍在自问的问题：用户 `Bob` 能在第一次尝试后 9 分钟内验证成功的唯一原因是：他们重新启动了 CS2，那次他们运气不错。***

***另一个结论是：一旦 Steam ID 被验证，对于该游戏实例，除非在运行 `CS2.exe` 时关闭 Steam 客户端，否则它将永远不会失败。***

## ***结束语[](#结束语)***

***虽然我相信所提出的解决方案对于 99% 的所有用户都有效，但我也清楚地意识到，有一个非常小的用户群体由于本博客文章中未涉及的其他原因而受到此错误的影响。***

***你猜到了 Esportal 匹配平台在我们于 2024 年 1 月 10 日修复该行为之前是怎么做的吗：它启动了 `CS2.exe`，并且一旦进程可用（但尚未完全初始化），就使用适当的匹配游戏服务器执行了 `steam://connect/<IP>:<Port>` 命令。我们应用了修复措施并将其部署到每个 Esportal 玩家后，与此问题相关的用户工单就停止了。***

***话虽如此，我要以一个我真的很喜欢的图结束这篇博客，并且我真心希望再也不要看到一个 Slack 消息带着那个错误：***

```
***`xychart-beta
  title "Relative amount of 'No user logon' tickets"
  x-axis [03.01., 04.01., 05.01., 06.01., 07.01., 08.01., 09.01., 10.01., 11.01., 12.01.]
  y-axis "Share of daily tickets (in %)" 0 --> 23
  bar [6, 10, 18, 18, 23, 10, 9, 0, 0, 0]`*** 
```

***社交讨论:***

***感谢我的同事们对校对和提供反馈！***

****免责声明：观点为个人观点，与我的雇主无关。****
