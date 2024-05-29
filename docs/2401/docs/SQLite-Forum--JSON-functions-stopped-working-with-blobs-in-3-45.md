<!--yml
category: 未分类
date: 2024-05-27 15:02:09
-->

# SQLite Forum: JSON functions stopped working with blobs in 3.45

> 来源：[https://sqlite.org/forum/forumpost/012136abd5292b8d](https://sqlite.org/forum/forumpost/012136abd5292b8d)

> I believe that if a certain feature worked for N years, and then stopped working — that is a breaking change.

The development team is well aware of this. SQLite users should be aware that when they rely on undocumented behavior, or behavior contrary to the documentation, they bear the risk of such changes. Nevertheless, the developers often elect to preserve undocumented behavior, and sometimes elect to document it so as to make it part of the public API.

Worth noting in relation to this issue is that the JSON binary encoding feature, with its effect upon the JSON functions, was exposed for several weeks prior to the 3.45 release. The pre-release drops and announcements are intended to give attentive users a chance to bring out issues, such as the one instigating this thread, before they become less subject to remedy by altering behavior or the public API. After a release, behavior changes that conflict with the published API are much less likely to be effected.

> At the very least, it deserves to be mentioned in the release notes.

That is a sensible suggestion.

> Also, maybe the SQLite team should not encourage ad hominem on their forum.

Technically, the post to which I replied contained no ad hominem.^([1](https://sqlite.org/forum/forumpost/012136abd5292b8d#footnoteP-1-a)) I would agree that labeling your complaint as "crying" would be rude, insensitive, and out of place in this forum. Abusive posts are often deleted or partially elided in moderation. We moderators try to keep discussions civil here. However, you were simply urged to not cry. While I saw the weak implication that you might do so or were doing so as a bit over the edge of clear civility, I elected to ignore that minor transgression -- in part because it is good advice when applied to the set of people who may encounter similar difficulties.^([2](https://sqlite.org/forum/forumpost/012136abd5292b8d#footnoteP-2-a))

That said, we (moderators) much prefer unquestionably civil discussion. Those who depart from it may either never see their uncivil posts or may see them deleted or correctively edited. Further, no appearance of uncivil remarks or phrasing should be considered to be encouragement of such. At most, it represents neglect or very limited tolerance by the moderator(s).