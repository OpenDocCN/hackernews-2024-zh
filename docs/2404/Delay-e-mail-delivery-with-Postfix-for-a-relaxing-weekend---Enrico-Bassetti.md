<!--yml
category: 未分类
date: 2024-05-27 13:25:18
-->

# Delay e-mail delivery with Postfix for a relaxing weekend | Enrico Bassetti

> 来源：[https://www.enricobassetti.it/2024/04/delay-e-mail-delivery-with-postfix-for-a-relaxing-weekend/](https://www.enricobassetti.it/2024/04/delay-e-mail-delivery-with-postfix-for-a-relaxing-weekend/)

One good piece of advice is never to read e-mails if you want to have a pleasant and relaxing weekend. Unfortunately, it is not so easy: open-source projects you are contributing to, friends, hobbies, and newsletters are all good reasons to read e-mails once in a while.

But we all have that friend, the one that sends many e-mails, and you don’t want to read them on the weekend. Let’s implement some “delay-until-Monday” feature in Postfix!

## Postfix queues

Did you know that [Postfix has different *queues*](https://www.postfix.org/QSHAPE_README.html#queues)? In fact, Postfix has:

*   **maildrop** is the queue for e-mails sent locally using `sendmail`. This is the only queue that is actually usable even when Postfix is not running;
*   **incoming**, used by the `cleanup` Postfix process to store e-mail that just arrived. It exists because the `active` queue (see below) can accommodate only a fixed number of active e-mails;
*   **active** is the queue containing e-mail scheduled to be sent. This queue has a fixed capacity to avoid overloading Postfix, as these e-mails are the next to be sent (you don’t want your system to try to send more than 200k e-mails at the same time, do you?);
*   **deferred** is used when there is a **transient** error (i.e., temporary problems), so e-mails are queued here and picked up later for retry;
*   Finally, the **hold** is a special queue where messages go if some administrative ACL says so, and they are moved to other queues using the `postsuper` command.

Theoretically, we may just need to keep the e-mail in the *incoming* queue until the right moment or move it to the *active* queue and schedule the delivery later. However, neither approaches are really in line with the description of those queues, and, as far as I know, there is no easy way to do that.

We will use the **hold** queue instead!

## Postfix lookup tables

Here is the plan: we will match the incoming e-mail with some ACL using the `HOLD` action so that Postfix will move incoming e-mails in the *hold* queue. Then, we can use some bash script and crontab to release them when the moment comes.

Enters the [Postfix lookup *tables*](https://www.postfix.org/DATABASE_README.html)!

Postfix can look up information in *tables* by matching specific parts of the e-mail (or the envelope) in them to get back other information, such as actions, alias destinations, allowed hosts, and many more. These tables can be stored in a local file or in some network services. It is very handy, as you can basically [link Postfix to a database server to query for e-mails and stuff](https://www.postfix.org/pgsql_table.5.html), and this is basically what everyone (who has a large number of addresses) does.

In our case, we need to use [an `access` table](https://www.postfix.org/access.5.html). `access` tables return actions for a match. Specifically, we are looking for the `HOLD` action. In theory, a file like this should suffice:

```
[[email protected]](/cdn-cgi/l/email-protection)    HOLD 
```

If you save a file like this, and you use the `file:` table type in a specific parameter in the Postfix configuration (as we see later), all e-mails coming from `[[email protected]](/cdn-cgi/l/email-protection)` will be moved to the *hold* queue, yay!

The problem is that there doesn’t seem to be any way to assign a time period to this rule. This means that the rule will always match, even if you use `postsuper` to remove it from the queue! It does not work that well. What else can we use?

It turns out we can use a database for that. If you are already using a database, that’s completely fine. However, I don’t have it, as my Postfix setup is very minimal. In this case, we can just use `sqlite` as a database and query a local file with some SQL magic.

First, we need a SQLite database. Make sure that Postfix is compiled with the `sqlite` table type (it is by default) and that you have the `sqlite3` executable in your path. Then:

```
$ sqlite3 /etc/postfix/time_based_access_sqlite.db SQLite version 3.40.1 2022-12-28 14:03:47 Enter ".help" for usage hints. sqlite> 
```

Perfect! Now, let’s create the database table that we will use in order to store information for the Postfix table:

```
CREATE TABLE time_access_dow (  email TEXT, day_of_week INTEGER, action TEXT ); 
```

Now we can close the database file, and we can create the file that holds the “database connection information”. Let’s say, `/etc/postfix/time_based_access`:

```
dbpath = /etc/postfix/time_based_access_sqlite.db
query = SELECT action FROM time_access_dow WHERE email = '%s' AND day_of_week = strftime('%%w', current_date) 
```

As you can see, I am using the “day of the week” (0-6, where 0 is Sunday). We can hold e-mails from `[[email protected]](/cdn-cgi/l/email-protection)` on Sunday by inserting the following:

But we are not finished yet. We need to tell Postfix to query this. We will modify the `/etc/postfix/main.cf`. Specifically, you want to alter the `smtpd_sender_restrictions` parameter to add your table. Keep in mind that the configured actions are executed using the exact order you specify in the configuration. So, you want your query to be executed last, or at least after the usual checks for blocklists, unknown recipient domains, etc.

```
smtpd_sender_restrictions = reject_non_fqdn_sender,  permit_sasl_authenticated, ... check_sender_access sqlite:/etc/postfix/time_based_access 
```

Restart Postfix to apply. Now, e-mails from `[[email protected]](/cdn-cgi/l/email-protection)` on Sundays will be on hold!

## Release e-mails on hold

Let’s create a script to release them (create the file, then +x it):

```
#!/usr/bin/env bash set -eou pipefail   # Get all e-mail addresses that are configured for HOLD for email in $(sqlite3 /etc/postfix/time_based_access_sqlite.db "SELECT DISTINCT email FROM time_access_dow"); do  # Release all e-mails from HOLD for this e-mail for MSGID in $(postqueue -j | jq -r "select(.queue_name == \"hold\" and .sender == \"$email\").queue_id"); do postsuper -H "$MSGID" done done 
```

Now we can schedule this command to run, say, every morning at 5am:

```
0 5     * * *   root    /etc/postfix/time_based_access.flush 
```

Unfortunately, `postsuper` must run as a super-user. However, as good practice, you should already know how to configure a user with the necessary permissions to use `sudo` and AppArmor.

That’s all. You have your own Postfix-powered system to delay e-mail delivery!

## Caveats

As you might have noticed, the SQL query only checks for weekdays. So, if the person sends you an e-mail at 00:01 on Monday, that e-mail will be delivered to your account (also, before the others that will be released at 5am). If your friend is a night owl, you may want to tweak the database structure to store also some time information there.

## Future improvements

The next improvement will be querying my calendar for holidays and time off. I have a CalDAV server, so it should not be so challenging to work with (I see you laughing, stop!). Maybe next weekend I can work on th… wait, wasn’t I looking forward to a pleasant weekend?!?