<!--yml
category: 未分类
date: 2024-05-27 12:58:47
-->

# Replacing complicated hashmaps with SQLite | Wilson Husin

> 来源：[https://husin.dev/ephemeral-sqlite/](https://husin.dev/ephemeral-sqlite/)

In a recent project, I was working on a command line application with tricky data manipulation scenario. Imagine if you were moving members from mailing lists (e.g. Google Groups) to group chat service (e.g. Slack), where the goal is for members of the mailing list will find a new home in group chat channels.

It is perfectly sensible for engineers with hacker spirit to say “script this out and run it for each of the groups” and I agree — surely it’s a thing that a few HTTP calls and hashmaps can solve?

Turns out, this situation comes with some curve balls:

*   Mailing lists may have other mailing lists as members (nested mailing list), but group chat channels only have members, thus some may need to be merged in migration.
    *   Potential of many-to-one relationship, where 2+ entities of mailing lists ends up in 1 channel.
*   Some mailing lists already have an existing group chat channel counterparts, but the membership is not guaranteed to be consistent.
    *   Not all requests are “create channel”.
    *   Same applies to users and memberships — take the existing records to consideration.
*   Mailing list have members which are identified by email address. Group chat service can lookup users by email, but need the user ID for API calls to link channel membership.
    *   We would need to lookup users of group chat service by both email and user ID; 2 lookup keys (hashmaps) for the same value kind.

Now that doesn’t look so simple anymore, does it? As I was thinking about it, there is a lingering thought at the back of my head: “If I could use `WHERE` on any field and `JOIN` several entities, the new memberships could be solved in a single SQL query.”

And then it hit me — maybe I really could after all. So I did exactly that.

Much of what I had originally implemented is now much simpler, without having to worry about passing around states in every function. Initial implementation has my function signatures looking like this:

```
// MatchUsers performs best effort matching of users, returning: // - map[string]User of email with their User counterpart // - []string of emails without User counterpart // - error, if any func MatchUsers(  ctx context.Context, emailsToMatch []string, existingUsers map[string]User, ) (map[string]User, []string, error) 
```

Pretty much all functions require the states to be passed around that creates noise and bug-prone (e.g. what if a function unintentionally modified the hashmap?).

With SQLite as state management, the same method now look like this:

```
// MatchUsers performs best effort matching of users by comparing emails. // Users without any match will have `slack_user_id` as NULL. func MatchUsers(ctx context.Context, conn *sql.DB) error 
```

And thus when I want to lookup the list of users which did not find any matches, it ends up being an intuitive flow:

```
SELECT  *  FROM  mailing_list_users  WHERE  slack_user_id  IS  NULL; 
```

If you find yourself passing around hashmap, reindexing the map, writing too many conditions to settle state management, and thought that it would be trivial in SQL: consider using ephemeral SQLite instance!

Miscellaneous interesting bits from the project:

*   Because database is ephemeral (using `:memory:`), I need to run migration every time the application boots. Corollary to that, I don’t need migration tools to manage up / down migrations!
*   I can use `sqlite3` CLI to inspect the database, which is a nice bonus when debugging. Of course I would have to change the setup from `:memory:` to a file path that exists in file system.
*   The final form of `MatchUsers` end up taking only `context.Context`, since it worked out better for the rest of the project to access database connection from context.