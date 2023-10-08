---
title: "Accessing cheatsheets from the command line"
date: 2023-10-08
summary: The cheat.sh utility allows you to access a lot of comprehensive, community-driven cheatsheets for many popular tools.
---

## The problem

Very often, I know which command line tool would come in handy for a given task at hand, but I can't recall the exact syntax I should use or the options that are available to me. 

Looking into the documentation is definitely an option, but they sometimes lack practical examples. Also, usually, I'm already familiar with this tool, but I just need a quick refresher.

## The solution

Meet [cheat.sh](https://github.com/chubin/cheat.sh). An open-source tool that gives you access to community-driven cheatsheets for programming languages, DBMSes, and command line tools through a simple interface.

Want a [quick overview](https://xkcd.com/1168/) of the `tar` utility? Just type this in your terminal, no installation required:

```bash
curl cht.sh/tar
```

You'll get answers from multiple cheatsheets repositories at the same time. The screenshot below shows the part of the response that comes from the https://tldr.sh/ website:

{{< figure 
align=center
src="/cheatsh/tar.png" 
>}}

It also works for programming languages. For example, to learn about list comprehension in Python, you could use this:

```bash
curl cht.sh/python/list_comprehension
```

> To get a list of topics available for a given language, execute `curl cht.sh/python/:list`. Additionally, most languages have a special `:learn` page that describes the basics to get you started quickly.

### Command line client

If you don't want to constantly type the `curl` command to access the cheatsheets, you can install the command line client according to the [instructions](https://github.com/chubin/cheat.sh#command-line-client-chtsh) on the GitHub page. There is also an option to activate tab completion support.

I did all of the above and created a custom bash function called `c` that executes the script and also pipes the result to the `less` program for convenience (the `"$@"` part refers to arguments passed to the function).

```bash
function c() {
    $HOME/.dotfiles/shell/scripts/cht.sh "$@" | less
}
```

After doing that, I can start typing `c ff` and instantly get suggestions matching the query:

{{< figure 
align=center
src="/cheatsh/ffmpeg.gif" 
>}}

If you're interested, here's a [commit](https://github.com/ClouddJR/dotfiles/commit/6b95fd1ee4e2d343d148336fa4d85ad8a230ad97) in my dotfiles repository where I added this tool (notice the `#compdef c` line at the top of the `_cth` file that was necessary to get tab completion for my custom function).