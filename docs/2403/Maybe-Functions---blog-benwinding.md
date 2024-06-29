<!--yml

category: 未分类

date: 2024-05-27 14:50:40

-->

# Maybe Functions | blog.benwinding

> 来源：[https://blog.benwinding.com/maybe-functions/](https://blog.benwinding.com/maybe-functions/)

Maybe函数是一个微妙的怪物，它的触手在代码库中扩散。它交替的“*做/不做某事*”功能使得代码难以理解、维护和调试。

### 例子

这里有一个具体的例子，它是一个“maybe”函数，因为它只返回用户的朋友，如果用户已登录。基本上它引入了可能的`return null`。

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

*注：*

+   这与“Null Object Pattern”密切相关，但我想从函数的角度来解释它。

+   这里也很重要的一点是，在实际使用中，“maybe functions”可能不会方便地以“maybe”前缀开头…

当使用“maybe functions”时的权衡是什么？

## 优点

+   验证逻辑与实现耦合

+   对调用者很容易，在许多地方都可以调用

## 缺点

+   调用者控制较少，它们依赖于隐藏在“maybe function”中的验证。

+   隐藏复杂性但实际上并未消除它

+   现在依赖于输出的东西现在需要处理maybeness的情况

当实现一个“maybe function”时，更有可能引入更多的“maybe functions”来处理所有的`null`值。以下是一些伪代码，展示了基本问题：

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

您可以看到这种模式*可能*已经失控了… 在消费者端看起来很干净，没有控制逻辑。然而，`null`处理扩展到所有新函数，并且模糊了函数实际上在做什么… 通常是“什么也没做”，那么为什么还要调用它？

如何改进这个？

## 解决方案1 - 提升“maybeness”

从函数中移除“maybeness”的最简单方法是将“maybe”逻辑提升到消费者，这消除了函数可能的`null`部分，允许其他函数也去除它们！

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

这样更容易理解，因为每个函数都按照其名称执行，并且需要考虑的“maybeness”更少。

## 解决方案2 - Monads

通过使用单子（monads）也可以概括和隔离“maybeness”。在下面的示例中，所有的`null`检查都通过“runSafely”进行，这消除了其余函数中的大部分“maybeness”。

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

对于许多情况来说这可能是合适的，但我认为第一种解决方案绝对应该首先被采纳，因为它不依赖于另一个抽象层级。

## 结论

对我个人来说，maybe函数一直很让我烦恼，它们可以在任何允许`null`的语言的许多代码库中找到。它们似乎很容易添加，但很难移除。但希望这能说明问题并提出解决方法。

> “函数应该做某事，而不是*或许*做某事…”
