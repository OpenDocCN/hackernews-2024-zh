<!--yml
category: 未分类
date: 2024-05-29 12:44:45
-->

# Linux text manipulation

> 来源：[https://yusuf.fyi/posts/linux-text-manipulation](https://yusuf.fyi/posts/linux-text-manipulation)

Today, I wanted to make a module for my AwesomeWM status bar that shows the currently playing Spotify song, it’d be nice if it could show something human-readable like this: *Wild World by Yusuf/Cat Stevens*.

Through a quick search, I found a tiny bash script, [sp](https://gist.github.com/streetturtle/fa6258f3ff7b17747ee3), that does what I need. except the output must be cleaned up.

```
joe@popos> sp current
Album Tea For The Tillerman (Remastered 2020)
AlbumArtist Yusuf / Cat Stevens
Artist Yusuf / Cat Stevens
Title Wild World 
```

I could ask chatgpt to format this or write a Lua script, but that wouldn’t be much fun, *would it?* in this article, we’ll learn how to use `awk`, `sed` and other commands to get the desired output.

Intended Audience

this is intended for people who have used Linux but are otherwise new to the terminal, if you’ve at least created a file from the command line, you’re good to go, I’ll cover everything else throughout the article

If you want to follow along, download the bash script, and experiment with the commands we’ll learn, but that’s not required per se.

### Mini Problem #1

To get started `sp current` returns the album name, album artist, song artist, and song title, whereas we only need the artist and title. we can use `sed` to pick the specific lines we need.

sed works by looping through lines of the input and doing a given action, by default it prints the resulting lines to stdout. we can give sed a file to work with but more commonly you’ll pipe a command into sed, as we’ll do later on.

> <details><summary>What do you mean by ***pipe***?</summary> Generally, in Linux, a command’s input is read from the keyboard and its output (along with any errors) is printed on your screen. we can, however, change this behavior.
> 
> piping essentially lets you pass a command’s output onto another command’s input. programs can choose whether or not to support piping, but most basic Linux programs (and all the ones we’ll discuss today) support it.</details>

let’s look into sed in more detail

```
sp current | sed '' # no action given, sed will print the lines as is
sp current | sed 's/Cat/Dog/g' # replace Cat with Dog and print every line 
```

```
# the n flag suppresses the automatic printing
# this will replace Cat with Dog but print nothing!
sp current | sed  -n 's/Cat/Dog/g' 
```

```
# p prints a line
# so this will print each line twice (default + p action)
sp current | sed  'p'

# if we combine -n with the p action we can do this
# this is equivalent to plain sed ''
sp current | sed -n  'p'

# we can print a specific line too (we'll need this later!)
# this will print every line AND print the second line an extra time
sp current | sed '2p' 
```

```
# finally, we can chain actions.
# this substitutes Cat with Dog and prints ONLY the 2nd line, because of the n flag.
sp current | sed -n 's/Cat/Dog/g;2p' 
```

and now—as you may have already figured out—we can print the artist and song name only like so:

```
sp current | sed -n '3p;4p' 
```

and we end up with

```
Artist       Yusuf/Cat Stevens
Title        Wild World 
```

### Mini Problem #2

Next, we need to get rid of the column labels, we can use `cut` for this. As with `sed`, `cut` accepts a file as its input, if no file is given it’ll read from stdin.

Here is how we usually work with `cut`, you give it a delimiter and a set of columns to print, and it will split each line by that delimiter and return the selected columns.

in our case, columns are separated with a space, so we can do the following

```
# we're splitting the input by spaces and picking the 2nd column
sp current | cut  -d ' '  -f2 
```

our columns are seperated by multiple spaces though, so cut will treat the empty-ness between each space as its own column, to avoid this we can inject the command `tr` to trim the extra spaces:

```
# the s flag specifies what to trim
sp current |  tr -s ' ' | cut -d ' ' -f2 
```

But this is still not totally right. if a song’s title contains more than one word we’ll get this

```
Artist       Yusuf/Cat Stevens
Title        Wild World

Yusuf/Cat
Wild 
```

you can pass `cut` a column range to print, as in `cut -d ' ' -f2-4` to select the 2nd, 3rd, and 4th columns, **if the range is open-ended**, `cut` **will print every column until the end of the line**, so let’s incorporate that into our script.

```
sp current | tr -s ' ' | cut -d ' '  -f2- | sed -n '3p;4p' 
```

output:

```
Yusuf/Cat Stevens
Wild World 
```

**note**: as I’ve shown you `tr` needs to be applied before `cut` on the contrary, `cut` and `sed` can be swapped because they serve entirely different purposes.

### Mini Problem #3

Now, what’s left is to simply join the lines and format them nicely. we can use `awk` to do this.

`awk` is based on a condition-script model, It’s similar to `sed` it checks if the current line matches the condition and executes the appropriate script, although I find `awk` to have a more expressive syntax.

awk scripts are written in quotes like this:

```
sp current | awk '{print $1}' 
```

let’s look at a few examples. any of these can be passed to awk in quotes.

```
{print} # no condition given, prints every line.

# NR==1 matches the 1st line only
# thus, this will print the 1st line
NR==1{print} 

# any subsequent script isn't part of the condition. 
# despite them being on the same line.
# this will print the print every line + the first line again
NR==1{print}{print} 
```

```
# awk has pre-defined globals corresponding to columns in your input
# this behaves like sed and will print the first column.
{print $1}

# $0 matches the entire line
# this is equivalent to {print}
{print $0} 
```

Lastly, you can assign globals to variables,

this assigns a to the 1st column of the 1st line and then prints `x` for every line in the input. note that the first script only runs once, thus a will stay the same for every line

```
NR==1{x=$1}{print x} 
```

now we can combine everything we know to get this script:

```
sp current | ... | awk 'NR==1{a=$0; next}{print $0 " by " a}' 
```

this does the following:

1.  we assign `a` to the artist name (entire first line)
2.  `skip` tells `awk` to *skip* the execution of the second script and to move to the next line
3.  we’re on the second line now so `NR==1` isn’t matched and the first script is skipped.
4.  print the song name (`$0`) then the string literal `by` then the artist name (content of variable `a`)

```
Yusuf/Cat Stevens
Wild World

Wild World by Yusuf/Cat Stevens 
```

## Final Thoughts

That concludes our script, here is what we ended up with:

```
sp current | tr -s ' ' | cut -d ' ' -f2- | sed -n '3p;4p' | awk 'NR==1{a=$0; next}{print $0 " by " a}' 
```

Now this script could be optimized in different ways, but instead of showing you how to do it here, I leave that as *an exercise to the reader*. here are a couple of things you can do, ranging from easiest to hardest:

*   replace the `cut` command with an additional `sed` action
*   re-write the entire thing from memory
*   make the script as short as possible
*   write the entire script in `awk`

Finally, I hope you learned something new today and if you attempt any of these, please share what you did either in the mastodon replies or through an article on your blog. I promise to take a look and I’ll probably learn something myself!