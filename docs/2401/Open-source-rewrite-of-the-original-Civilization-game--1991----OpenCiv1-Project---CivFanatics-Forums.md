<!--yml
category: 未分类
date: 2024-05-27 15:15:56
-->

# Open source rewrite of the original Civilization game (1991) - OpenCiv1 Project | CivFanatics Forums

> 来源：[https://forums.civfanatics.com/threads/open-source-rewrite-of-civilization-1-source-code-openciv1-project.682623/](https://forums.civfanatics.com/threads/open-source-rewrite-of-civilization-1-source-code-openciv1-project.682623/)

As far as we're aware, the actual in-game mechanics are very close, if not identical (although I'm sure our resident expert reverse engineers

[@darkpanda](https://forums.civfanatics.com/members/130195/)

,

[@tupi](https://forums.civfanatics.com/members/212558/)

and others would enjoy comparing the details of the source to their own reversed DOS code), it's mostly presentation that lets it down

-There's a bad bug in the save logic which means that if you save whilst some units still have moves left in your turn, all units will have all their moves back on the same turn when you resume (DOS actually had this bug too, but it only effected autosaves, so to trigger it, you had to manually shift a save to the autosave slot then load it). Windows does this every time you save.

-Alot of functionality in the city menu has been shifted to the windows menu bar, I think most would prefer to do all their work in the city screen like in DOS.

-Windows uses the same shared music for the title screen and intro, and cuts off most of the intro music meaning you watch most of the intro in silence (see this post for details

[https://forums.civfanatics.com/thre...dows-complete-soundtrack-overhaul-mod.633237/](https://forums.civfanatics.com/threads/civilization-1-for-windows-complete-soundtrack-overhaul-mod.633237/)

)

-The civ specific music played during diplomacy is instantly cut off as soon as actual diplomacy begins, in DOS it continues playing (also, as with the title screen, the tracks are very short unless you mod them)

-The windows version of the save file does not save unit names for some reason. This means you can't have scenarios with custom unit names, you can with DOS.

-No animations for aquaduct, hanging gardens, other city screen water features. Animated shoreline and rivers are present in 256 colour mode only.

The graphics are obviously subjective, but alot of people prefer the terrain and unit graphics/colour scheme from the DOS version. This could be an optional choice of course. Particularly the choice of burgundy and yellow for the city screen is a bit of a shocker in my opinion!

EDIT - one more, the windows version uses the happiness model of the version 3 patch which introduces 'very unhappy citizens', in DOS they are coloured red on the city menu, in Windows there's no visual clue to differentiate them from 'normal' citizens (see this thread for discussion

[https://forums.civfanatics.com/threads/civil-disorder-why.638921/post-16425928](https://forums.civfanatics.com/threads/civil-disorder-why.638921/post-16425928)

)