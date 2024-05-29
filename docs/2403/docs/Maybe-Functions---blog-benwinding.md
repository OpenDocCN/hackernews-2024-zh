<!--yml
category: 未分类
date: 2024-05-27 14:50:40
-->

# Maybe Functions | blog.benwinding

> 来源：[https://blog.benwinding.com/maybe-functions/](https://blog.benwinding.com/maybe-functions/)

The maybe function is a subtle monster that spreads it’s tentacles across the code-base. It’s alternating functionality of “*does/doesn’t do something*” makes code hard to understand, maintain and debug.

### Example

Here’s a specific example, it’s a “maybe” function as it only returns the friends of a user, if the user is logged in. Basically it introduces a possible `return null`.

|  
```
function maybeGetUser(): User &#124; null {
 if (!loggedIn) {
 return null;
 }
 return fetchUser();
} 
```

 |

 *Notes:*

*   This is highly related to the “Null Object Pattern”, but I thought I would explain it from the perspective of functions.
*   It’s also important to note here, that in the wild “maybe functions” might not be conveniently prefixed with “maybe”…

So what are the trade-off’s when using “maybe functions”?

## Advantages

*   Validation logic coupled with implementation
*   Easy for the caller, can be called in many places

## Disadvantages

*   Caller has less control, they depend on validation hidden in “maybe function”
*   Hides complexity but doesn’t actually remove it
*   Things that depend on the output now need to handle the maybeness case

When a “maybe function” is implemented, it makes it far more likely that **more** “maybe functions” will be introduced to deal with all the `null` values. Here’s some pseudo code to show the basic problem:

|  
```
 function Page() {
 const bestFriends = maybeRenderBestFriends();
 return <>
 {bestFriends}
 </>;
}
 function maybeRenderBestFriends() {
 const user = maybeGetUser();
 const friends = maybeGetFriends(user);
 const bestFriends = maybeFilterBestFriends(friends);
 return render(bestFriends);
}
 function maybeGetUser(): User &#124; null {
 if (!loggedIn) {
 return null;
 }
 return fetchUser();
}
 function maybeGetFriends(user: User &#124; null): Friend[] &#124; null {
 if (!user) {
 return null;
 }
 return fetchUserFriends(user);
}
 function maybeFilterBestFriends(friends: Friend[] &#124; null): Friend[] &#124; null {
 if (!friends) {
 return null;
 }
 return friends.filter(f => f.bestFriend);
} 
```

 |

You can see how this pattern *maybe* getting out of control… it does look clean in the consumer, with no control logic. However the `null` handling spreads to all new functions, and obscures what functions are actually doing… which is usually “nothing”, so why even call it?

How can we improve this?

## Solution 1 - Lift the maybeness up

The easiest way to remove the “maybeness” from functions is to lift up the “maybe” logic into the consumer, this removes the possible `null` aspects of the function, which allows other functions to remove theirs too!

|  
```
 function Page() {
 const bestFriends = loggedIn
 ? renderBestFriends()
 : null;
 return <>
 {bestFriends}
 </>;
}
 function renderBestFriends() {
 const user = getUser();
 const friends = getFriends(user);
 const bestFriends = filterBestFriends(friends);
 return render(bestFriends);
}
 function getUser(): User {
 return fetchUser();
}
 function getFriends(user: User): Friend[] {
 return fetchUserFriends(user);
}
 function filterBestFriends(friends: Friend[]): Friend[] {
 return friends.filter(f => f.bestFriend);
} 
```

 |

This is much easier to understand, as each function does what it says and there’s much less “maybeness” to account for.

## Solution 2 - Monads

It’s also possible to generalize and isolate the “maybeness” by using monads. In the below example, all `null` checks go through “runSafely”, which removes a lot of “maybeness” in remaining functions.

|  
```
 class Maybe<T> {
 constructor(
 value: T &#124; null
 )
 runSafely(fn: (val: T) => V): Maybe<V> {
 if (!this.value) {
 return new Maybe(null);
 }
 return new Maybe(fn(this.value));
 }
}
 function Page() {
 const bestFriends = maybeRenderBestFriends();
 return <>
 {bestFriends}
 </>;
}
 function maybeRenderBestFriends() {
 const user = new Maybe(getUser());
 const friends = user.runSafely(getFriends);
 const bestFriends = friends.runSafely(filterBestFriends);
 return render(bestFriends);
}
 function getUser(): User {
 if (!loggedIn) {
 return null
 }
 return fetchUser();
}
 function getFriends(user: User): Friend[] {
 return fetchUser();
}
 function filterBestFriends(friends: Friend[]): Friend[] {
 return friends.filter(f => f.bestFriend);
} 
```

 |

This might be appropriate for a lot of situations, but I believe the 1st solution should definitely be reached for first, as it doesn’t rely on another level of abstraction.

## Conclusion

Maybe functions have always annoyed me personally and they can found in many codebases across any language that allows `null`. They seem to be trivial to add, but difficult to remove. But hopefully this illustrates the concern and ways to fix it.

> “Functions should do something, not *maybe* do something…”