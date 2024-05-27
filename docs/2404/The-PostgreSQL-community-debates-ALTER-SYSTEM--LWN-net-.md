<!--yml
category: 未分类
date: 2024-05-27 13:23:33
-->

# The PostgreSQL community debates ALTER SYSTEM [LWN.net]

> 来源：[https://lwn.net/Articles/968300/](https://lwn.net/Articles/968300/)

| **Please consider subscribing to LWN**Subscriptions are the lifeblood of LWN.net. If you appreciate this content and would like to see more of it, your subscription will help to ensure that LWN continues to thrive. Please visit [this page](/subscribe/) to join up and keep LWN on the net. |

By **Jonathan Corbet**
April 8, 2024

Sometimes the smallest patches create the biggest discussions. A case in point would be the process by which the PostgreSQL community — not a group normally prone to extended, strongly worded megathreads — resolved the question of whether to merge a brief patch adding a new configuration parameter. Sometimes, a proposal that looks like a security patch is not, in fact, intended to be a security patch, but getting that point across can be difficult.

The PostgreSQL server is a complex beast that can be extensively configured and tuned for the environment in which it runs. There are, naturally, [many configuration parameters](https://www.postgresql.org/docs/current/runtime-config.html) available for administrators to work with. There are also two ways to set those parameters. In most deployments, perhaps, administrators will edit the `postgresql.conf` file to configure the system as needed. It is, however, also possible to adjust parameters within a running database with the [`ALTER SYSTEM` command](https://www.postgresql.org/docs/current/sql-altersystem.html). Any changes made that way will be saved to a separate `postgresql.auto.conf` file, which is also read by the server at startup; these changes are thus persistent.

#### Disabling `ALTER SYSTEM`

In early September 2023, Gabriele Bartolini [raised the idea](/ml/pgsql-hackers/CA+VUV5rEKt2+CdC_KUaPoihMu+i5ChT4WVNTr4CD5-xXZUfuQw@mail.gmail.com/) of allowing administrators to disable `ALTER SYSTEM`, either via a command-line parameter or a configuration option in its own right (which, using PostgreSQL jargon, he calls a "GUC", standing for "grand unified configuration" parameter).

> The main reason is that this would help improve the "security by default" posture of Postgres in a Kubernetes/Cloud Native environment - and, in general, in any environment on VMs/bare metal behind a configuration management system in which changes should only be made in a declarative way and versioned like Ansible Tower, to cite one.

PostgreSQL core contributor Tom Lane quickly [expressed his disagreement](/ml/pgsql-hackers/1756006.1694118434@sss.pgh.pa.us/) with this proposal: "<q>I don't think we need random kluges added to the permissions system. I especially don't believe in kluges to the effect of 'superuser doesn't have all permissions anymore'.</q>" He suggested using [event triggers](https://www.postgresql.org/docs/current/event-triggers.html) to implement such a restriction locally if it is really wanted.

Alvaro Herrera [pointed out](/ml/pgsql-hackers/202309091514.fdjfxlr5kjrw@alvherre.pgsql/) that `ALTER SYSTEM`, as a system-wide command, does not invoke event triggers. He also [more thoroughly explained the use case](/ml/pgsql-hackers/202309081455.mjyy7fyeb7xb@alvherre.pgsql/) that was driving this request:

> I've read that containers people consider services in a different light than how we've historically seen them; they say "cattle, not pets". This affects the way you think about these services. postgresql.conf (all the PG configuration, really) is just a derived file from an overall system description that lives outside the database server. You no longer feed your PG servers one by one, but rather they behave as a herd, and the herder is some container supervisor (whatever it's called).
> 
> Ensuring that the configuration state cannot change from within is important to maintain the integrity of the service. If the user wants to change things, the tools to do that are operated from outside.

Lane, though, [described](/ml/pgsql-hackers/1882832.1694187082@sss.pgh.pa.us/) the feature as "<q>false security</q>", and the discussion wound down for a while. But the fundamental disconnect had already become apparent: the opposition to restricting `ALTER SYSTEM` was based on the idea that it was intended as a security feature. As such, it would be a failure, since there are many other ways for a PostgreSQL user with the ability to run `ALTER SYSTEM` to take control of the server. But, as Bartolini [said](/ml/pgsql-hackers/CA+VUV5pK4N8FaGa0y47ZFVk9qWOwWMYOAUtX0J-o4A7hHfJRYA@mail.gmail.com/), this restriction is meant as a *usability* feature, closing off a configuration mechanism that is not meant to be used in systems where declarative configuration systems are in charge.

#### The new year

Robert Haas [restarted the conversation](/ml/pgsql-hackers/CA+TgmoaJxzx+EqmfVQxtJMKor7WSminuHjfCaO1N2z=cNoHG6Q@mail.gmail.com/) in January, acknowledging that the proposal is not meant as a security feature, but worrying that it would be seen that way anyway:

> I have to admit that I'm a little afraid that people will mistake this for an actual security feature and file bug reports or CVEs about the superuser being able to circumvent these restrictions. If we add this, we had better make sure that the documentation is extremely clear about what we are guaranteeing, or more to the point about what we are not guaranteeing.

Even then, he worried, about "<q>security researchers threatening to publish our evilness in the Register</q>". Lane then [declared](/ml/pgsql-hackers/2256675.1706642404@sss.pgh.pa.us/) that the project "<q>should reject not only this proposal, but any future ones that intend to prevent superusers from doing things that superusers normally could do</q>". Haas [responded](/ml/pgsql-hackers/CA+TgmoYNBs68DQifceGb4SNS8iaLt_wQCKuVJ8rNqeSvU4LiKA@mail.gmail.com/), though, that the original proposal might have merit, and should be taken seriously.

The conversation continued inconclusively; two months later, Haas [complained](/ml/pgsql-hackers/CA+TgmoaCwtTn7QEuRYFUWypXUv6La5_-oYiiMJgw4LeC-ZujBw@mail.gmail.com/) that not much progress was being made. That notwithstanding, he said: "<q>As far as I can see from reading the thread, most people agree that it's reasonable to have some way to disable ALTER SYSTEM</q>". There were, though, six competing ways in which that objective could be accomplished. These included the command-line option and configuration parameter originally proposed, along with an event trigger, pushing it into an extension module, recognizing a sentinel file created by the administrator, or just changing the permissions on `postgresql.auto.conf`. He suggested that the configuration option and the sentinel file were the most viable options.

Lane [answered](/ml/pgsql-hackers/2366852.1710443616@sss.pgh.pa.us/) that any such restriction, implemented by any of the above mechanisms, could still be bypassed by a hostile administrator. Haas [replied](/ml/pgsql-hackers/CA+TgmobRx215ussbQgjM4dgds8xAcMwKeKmHN4UQoSV5jP4uzQ@mail.gmail.com/) that the proposal was not a security feature, but Lane [dismissed](/ml/pgsql-hackers/2372973.1710446903@sss.pgh.pa.us/) it as "<q>a loaded foot-gun painted in kid-friendly colors</q>" that would lead to more bogus CVE numbers being filed against the project.

#### A new patch

On March 15, Jelte Fennema-Nio posted [a patch](/ml/pgsql-hackers/CAGECzQRrLpLzoOHmmUzo4g_Ed0CchJrez90s1tzZNoH00v-eFA@mail.gmail.com/) implementing the restriction as a configuration parameter. It was, in the end, just an updated version of the patch posted by Bartolini in September with some documentation tweaks. Various comments resulted in a number of newer versions; [the sixth of which](/ml/pgsql-hackers/CAGECzQQttoyf5EtjZvG6Biyx=cTKON4zgo2Ok27HgmvkayCb_A@mail.gmail.com/) came out a few days later. At this point, the patch consisted mostly of documentation, including this admonition:

> Note that this setting cannot be regarded as a security feature. It only disables the `ALTER SYSTEM` command. It does not prevent a superuser from changing the configuration remotely using other means.

Haas [welcomed this version](/ml/pgsql-hackers/CA+TgmoZ26YPK0to4Ac-oTz56Ae_uU2zearWDwPLHAFeVtcjqsA@mail.gmail.com/) and asked whether there was a consensus on proceeding with it. Lane seemingly had [changed his view somewhat](/ml/pgsql-hackers/2517206.1711388833@sss.pgh.pa.us/) at this point, saying: "<q>I never objected to the idea of being able to disable ALTER SYSTEM</q>". Bruce Momjian, instead, [worried](/ml/pgsql-hackers/ZgHNeR2qlxofPH_W@momjian.us/) that an administrator could enter an `ALTER SYSTEM` command disabling `ALTER SYSTEM`, at which point recovery could be difficult. In fact, as Fennema-Nio [answered](/ml/pgsql-hackers/CAGECzQQ5CaNo=JnL5_sxFCmyV8_76A9L7X-hRQYzhGLpJYzt4Q@mail.gmail.com/), that parameter cannot be set that way, so that particular trap does not exist.

Momjian also [took issue](/ml/pgsql-hackers/ZgHm36UALPgfD5GB@momjian.us/) with the name of the parameter (which was `externally_managed_configuration`), saying that it didn't really describe what was being restricted. He suggested `sql_alter_system_vars` as an alternative. Haas [agreed](/ml/pgsql-hackers/CA+TgmoaX_XJMORzkTvG0D6+KZP4mYUSq1Paa8YOSmZUy0=tJSQ@mail.gmail.com/) with the complaint, but thought that `allow_alter_system` made more sense. That is, in the end, the name that was chosen for this option.

The discussion was not over yet, though. Lane [wanted](/ml/pgsql-hackers/3868891.1710859910@sss.pgh.pa.us/) the server to ensure that `postgresql.conf` and `postgresql.auto.conf` were not writable by the `postgres` user if `allow_alter_system` is disabled. Otherwise, the database administrator would still be able to modify the configuration simply by editing one of the files. Fennema-Nio [disagreed](/ml/pgsql-hackers/CAGECzQQhvV1K-pDgrd8NVqXP0PUY9bCDs+CvE31KXX6hKqNB-Q@mail.gmail.com/), saying that disabling `ALTER SYSTEM` was sufficient for the intended use case, but Lane [said](/ml/pgsql-hackers/3879337.1710864320@sss.pgh.pa.us/) that the configuration parameter on its own is "<q>a fig leaf that might fool incompetent auditors but no more</q>". Fennema-Nio [reminded](/ml/pgsql-hackers/CAGECzQQoZC0tG1xpi_+-O7uc3ESA7vJ+-Gsnb18WF0e1X78JKA@mail.gmail.com/) Lane that the point of `allow_alter_system` is not security. Haas [complained](/ml/pgsql-hackers/CA+TgmoYsyrCNmg+Yh6rgP7K8r-bYPjCeF1tPxENRFwD4VZAZvw@mail.gmail.com/) that: "<q>I don't understand why people who hate the feature and hope it dies in a fire get to decide how it has to work.</q>" The file-permission check was, in the end, not added.

Even then, the discussion was not quite done; Momjian [questioned](/ml/pgsql-hackers/ZgSceowq_HL3VI_l@momjian.us/) merging this change so late in the PostgreSQL development cycle. "<q>My point is that we are designing the user API in the last weeks of the commitfest, which usually ends badly for us</q>". Fennema-Nio [pointed out](/ml/pgsql-hackers/CAGECzQSZ9Z1cs_Ld0YcMrF6HyPVvzvtKmq9RGuRVcwmnSH_9BQ@mail.gmail.com/) that the API was essentially unchanged from its initial, September form, and that months had been spent discussing alternatives. Haas [said](/ml/pgsql-hackers/CA+Tgmob3G5AL_fL0LGK8xBe1-TA6GhUCsJ5ohpaY348TLSv_-Q@mail.gmail.com/) that such a small patch would not improve by being held up for another release cycle: "<q>I think it has to be right to get this done while we're all thinking about it and the issue is fresh in everybody's mind.</q>"

On March 28, Momjian [agreed](/ml/pgsql-hackers/ZgXF7CEVb61aDRzY@momjian.us/) that the patch could be merged. One day later, Haas [did exactly that](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commitdiff;h=d3ae2a24f265a028f4b9e8df79ea7b075c6cf016;hp=0075d78947e3800c5a807f48fd901f16db91101b). And, with that, one of the longest-running development discussions in recent PostgreSQL history came to an end. The success of this effort is a testament to the persistence of a small number of developers who saw it through months of opposition and "helpful" implementation suggestions. Having decided that difficult issue, the project can turn its attention to the [small list of simple topics](https://commitfest.postgresql.org/48/) to be resolved during the upcoming July commitfest.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/968300/)

to post comments)