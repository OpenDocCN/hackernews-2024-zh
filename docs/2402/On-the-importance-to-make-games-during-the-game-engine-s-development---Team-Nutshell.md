<!--yml
category: 未分类
date: 2024-05-29 13:25:52
-->

# On the importance to make games during the game engine’s development | Team Nutshell

> 来源：[https://www.team-nutshell.dev/nutshellengine/articles/making-games-during-development.html](https://www.team-nutshell.dev/nutshellengine/articles/making-games-during-development.html)

# On the importance to make games during the game engine’s development

If you are making a game engine, this won’t surprise you but making a game and making a game engine are two very different things, that require very different skills.

You can consider that making a game alone is more suited to **artistic profiles**, such as game designers, texture artists, musicians, with programming being facilitated by modern games engines, which can even allow you to make a game without writing a single line of code, and that making a game engine is more suited to **technical profiles**, as it involves low-level programming, mathematics, physics, etc. And it is true for the most part, but as a game engine developer, and especially if you work alone, not ignoring these artistic aspects can help you a lot while writing your game engine.

Obviously, it all depends on the **purpose** of your game engine, you maybe don’t even want to make games with it, and are just interested by the technical aspects of it, and it is completely fine. But if you want some games to be made with your game engine, you must **try to make a game** on it.

It especially applies **if you hope people will use it to make their own games**, but also applies **if you plan on being the only user**. There are so many different things to do while working on a game engine that **it is easy to forget making games with it**, especially as new features are generally simply tested using simple scenes, as bigger scenes are difficult to setup and could make any issue harder to detect and debug.

If you are far into the game engine’s development, like *many years of development* far, and you never made a game with it, **it may be possible that your game engine… cannot make games**. There is a huge difference between creating simple scenes to test new features and being able to *make something* that has a real gameplay, and that can be **distributed to other users** without any issues. Maybe your game engine relies on **some files or configuration that are only available in your development environment**, or it maybe **only works on your machine**. Simply being able to create an executable (with some external files if needed) and sharing it to a friend is a good first step to make sure your game engine can actually make games. Obviously, you could argue that being distributable is not a requirement for a game, and it is true, but if you want to share them, make sure that you can.

Alright, but what about the *actual process of making games*. You will probably realize that making a game with your game engine is a **huge pain**, and, again, even if you plan on being the only user of your game engine, **knowing your engine perfectly won’t stop you from being bothered by many aspects of it** while actually using it, **making a game with your game engine should still be as painless as possible**.

There are **many aspects that you should be looking for** when making a game with your own game engine, for example:

*   **Is it hard to make scenes?** Actually placing objects in a scene, especially if you don’t have a scene editor yet, can be really tricky. Rotations should be looked for too, radians or quaternions should indeed be used internally to make the calculations but specifying them as euler angles in degrees in your scene files, and converting them when loading the scene, will make rotations easier to use when making a scene.
*   **Is scripting simple?** It’s really easy to think that all sanity checks (for example: do I have to check if an entity still exists before destroying it?) should be done by the user writing the scripts, but actually writing scripts where you have to check everything all the time makes the process really painful. Try moving some of these checks inside your scripting API and use logging to warn the user if something goes wrong, so they don’t have to make all these checks themselves. Obviously, too many checks may add a performance penalty so be cautious with the checks you can add to the game engine at a low cost and the ones the user should be doing.
*   **Are assets actually usable?** Your assets will probably not be packed inside your executable, so be sure that the paths are right when exporting your game for distribution.
*   **Where are the bugs?** This one is pretty obvious, but creating something bigger than what you used to do will certainly make some issues show up, such as crashes, performance issues, graphical or sound glitches, etc.

When making a game, take notes on **all things that bothered you**, things **that were repetitive when they should not** or the bugs you encounter, to be able **to fix all of this and have a better experience** when you will be making your next game. Making a game will also force you **to get familiar with tools** used by game developers. You don’t have to become a Blender or Aseprite expert but being able to make simple 3D models with textures, or 2D sprites, will help you a lot when making your game engine.

Now, the question lies: ***when* should you start making games with your game engine?** A game engine being a huge project, the sooner you identify these issues, the better. Every system doesn’t have to be perfectly ready to start making games with them. As long as your game engine has basic rendering and scripting, you should be able to do something.

Obviously, **the goal is not to start with a AAA open-world game**, a **small game that you can finish in a few days** is enough. **Game jams are great opportunities** to make these kind of games, as you won’t have to find a theme, which will be given by the game jam, and will be forced to work on a short time limit, generally a week-end or a week. So if you don’t have any ideas for your game, you can start by joining a game jam, and to find one, [itch.io has a game jam planning](https://itch.io/jams) to find one that fits you.

If you have friends that are willing to use your game engine (lucky you!), you can also organize a game jam where everyone will work on their own game, on your game engine. **Ask them to note every issue they encounter** and you will probably get a lot of very useful feedback from people that aren’t as familiar with your game engine as you.

To conclude, making games while making a game engine is important, don’t wait for it to be completely ready before starting. If you don’t, you may realize your game engine cannot actually make games, and it may be too late to fix it easily.