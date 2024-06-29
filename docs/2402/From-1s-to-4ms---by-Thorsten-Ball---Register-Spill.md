<!--yml

类别：未分类

日期：2024-05-27 14:58:35

-->

# 从1秒变成4毫秒 - Thorsten Ball - Register Spill

> 来源：[https://registerspill.thorstenball.com/p/from-1s-to-4ms](https://registerspill.thorstenball.com/p/from-1s-to-4ms)

当Zed开源时，HackerNews上有人[评论](https://news.ycombinator.com/item?id=39122280)说，Sublime Text在搜索当前缓冲区中所有出现的当前单词时更快。Zed花费1秒钟，而Sublime大约在200ms左右。

搜索所有出现意味着：你将光标放在一个单词上，按下`cmd-shift-l`，当前缓冲区中该单词的所有出现都被选择，并在每个出现的地方都有一个光标，随时可以进行多光标操作。

在这里，看看这个：

所以，Sublime在200ms内完成，而Zed则需要1秒钟？哦。

[安东尼奥](https://twitter.com/as__cii)，Zed的联合创始人之一，立即自信地说：“我们可以让它更快。”我的还不太熟悉这段代码库的思维在悄悄地问：“我们可以吗？”在我们深入研究之前，我的思维还不知道。

我们查看了[相关代码](https://github.com/zed-industries/zed/blob/8cc7a023906a283b91b84bd790106500497779aa/crates/editor/src/editor.rs#L6065-L6087)。这里是它的原始形式，需要1秒钟才能完成：

```
 `pub fn select_all_matches(
     &mut self,
     action: &SelectAllMatches,
     cx: &mut ViewContext<Self>
 ) -> Result<()> {
        self.push_to_selection_history();
        let display_map = self.display_map.update(cx, |map, cx| map.snapshot(cx));

        loop {
            self.select_next_match_internal(&display_map, action.replace_newest, cx)?;

            if self.select_next_state.as_ref().map(|selection_state| selection_state.done).unwrap_or(true)
            {
                break;
            }
        }

        Ok(())
    }`
```

忽略细节。重要的是中间的关键词：`loop`。这段代码可能是许多人自然而然地实现`select_all_matches`方法所做的事情：在没有更多匹配项可选择之前，使用`select_next_match`在循环中进行选择。完成，所有匹配项都被选择了。

当与安东尼奥一起查看时，我对这段代码的了解与你现在一样深，但他知道底层发生了什么。他的想法是：通过内联`select_next_match_internal`的方式进行优化，然后批量处理。

这类似于你在Web应用程序中优化N+1查询的方式。而不是在请求路径中执行类似这样的操作：

```
`loop {
  user = loadNextUser()
  if user == null {
    break
  }
  profilePicture = loadUserProfilePicture(user)
  blogPosts = loadLastFiveBlogPosts(user)

  render_template("user_profile", user)
}`
```

你会这样做：

```
`users = loadAllUsers()
pictures = loadUserProfilePicturesForUsers(users)
blogPosts = loadLastFiveBlogPostsForUsers(users)
for user in users {
  render_template("user_profile", user)
}`
```

或类似的操作。你明白我的意思。

这就是我们用上面的那段代码做的事情。我将向你展示我们最终的成果，但在看代码之前，请记住以下事项：不要担心细节！只需像阅读新牙刷说明一样自信地阅读代码，但同时保持好奇心（因为，嘿，也许你一直都搞错了）：

```
`pub fn select_all_matches(
    &mut self,
    _action: &SelectAllMatches,
    cx: &mut ViewContext<Self>,
) -> Result<()> {
    self.push_to_selection_history();
    let display_map = self.display_map.update(cx, |map, cx| map.snapshot(cx));

    self.select_next_match_internal(&display_map, false, None, cx)?;
    let Some(select_next_state) = self.select_next_state.as_mut() else {
        return Ok(());
    };
    if select_next_state.done {
        return Ok(());
    }

    let mut new_selections = self.selections.all::<usize>(cx);

    let buffer = &display_map.buffer_snapshot;
    let query_matches = select_next_state
        .query
        .stream_find_iter(buffer.bytes_in_range(0..buffer.len()));

    for query_match in query_matches {
        let query_match = query_match.unwrap(); // can only fail due to I/O
        let offset_range = query_match.start()..query_match.end();
        let display_range = offset_range.start.to_display_point(&display_map)
            ..offset_range.end.to_display_point(&display_map);

        if !select_next_state.wordwise
            || (!movement::is_inside_word(&display_map, display_range.start)
                && !movement::is_inside_word(&display_map, display_range.end))
            {
                self.selections.change_with(cx, |selections| {
                    new_selections.push(Selection {
                        id: selections.new_selection_id(),
                        start: offset_range.start,
                        end: offset_range.end,
                        reversed: false,
                        goal: SelectionGoal::None,
                    });
                });
            }
    }

    new_selections.sort_by_key(|selection| selection.start);
    let mut ix = 0;
    while ix + 1 < new_selections.len() {
        let current_selection = &new_selections[ix];
        let next_selection = &new_selections[ix + 1];
        if current_selection.range().overlaps(&next_selection.range()) {
            if current_selection.id < next_selection.id {
                new_selections.remove(ix + 1);
            } else {
                new_selections.remove(ix);
            }
        } else {
            ix += 1;
        }
    }

    select_next_state.done = true;
    self.unfold_ranges(
        new_selections.iter().map(|selection| selection.range()),
        false, false, cx,
    );
    self.change_selections(Some(Autoscroll::fit()), cx, |selections| {
        selections.select(new_selections)
    });

    Ok(())
}`
```

在空腹的情况下，70行代码而且没有语法高亮 — 抱歉。但即使你以前没有见过类似的代码片段，我相信你也理解正在发生的事情：

1.  检查是否有下一个选择，如果没有则返回。

1.  获取缓冲区中的所有当前选择（`let mut new_selections = …`）

1.  在当前缓冲区中查找所有匹配项（`select_next_state.query.stream_find_iter`）

1.  对于每个匹配项：将其添加到`new_selections`中，除非有一些单词边界检查。

1.  对选择进行排序并移除重叠部分。

1.  展开包含选择的代码。

1.  把编辑器中的选择更改为我们刚刚构建的选择(`self.change_selections`)，这将导致它们被渲染。

除了中间那个做一些邪恶的`plus-1`的`while`循环（我肯定会搞砸的，但安东尼没有），这代码真的很高级，对吧？

它甚至看起来都没有经过优化。没有优化代码通常穿戴的那些伤痕：没有用来保存另一个循环的辅助数据结构，没有掉入到裸指针的混战，没有SIMD，没有引入花哨的数据结构。一个都没有。

但是，问题在于这里。这就是我向你展示这个并且为什么我思考了这段代码三个星期的原因。

当我们第一次运行优化后的代码时，运行时间从1秒*降到了4ms*。4毫秒！

我简直不敢相信。4ms！用这么高级的代码！用`unfold_ranges`调用，找到所有匹配项，检查单词边界，扩展和排序，可能还要删除和渲染选择 — 4ms！

如果你在读这段并且对“那又怎样，对计算机来说4ms是一段漫长的时间”的反应是耸耸肩，那么是的，你是对的，4ms *对计算机来说* 是一段漫长的时间，是的，我同意，*但是* 根据那种反应，我敢打赌你不像我一样是作为一个程序员长大的。你看，我在建设网站、Web应用程序、后端等方面长大，在那个世界里基本上*没有*什么是需要4ms的。如果我在10ms内ping到法兰克福最近的数据中心，那么我怎么能在不到这个时间的情况下将东西传递给你呢？

所以我就站在那里，盯着这个4ms，想着：这就是Rust爱好者所说的零成本抽象吗？是的，我们以前都听过这种说法（是的，可能太多次了），我现在已经写了几年的Rust了，所以Rust快这个想法对我来说并不新鲜。

但看到像这样的高级代码在[5184行埃德加·爱伦·坡诗集的XML文件](https://github.com/zed-industries/zed/issues/6440)中找到了[2351次出现的](https://github.com/zed-industries/zed/issues/6440) `<span` ，*4ms*？

我不知道，伙计。我觉得这可能改变了我。
