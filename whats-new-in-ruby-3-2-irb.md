---
title: What's new in Ruby 3.2's IRB?
slug: whats-new-in-ruby-3-2-irb
seriies: ruby-irb
tags: ruby, ruby-on-rails, debugging, irb
domain: st0012.hashnode.dev
---

# What's new in Ruby 3.2's IRB?

IRB `1.6` has been released and will become Ruby 3.2's built-in IRB version.

It and a few recent releases include many enhancements [@k0kubun](https://twitter.com/k0kubun) and I made, and I want to introduce them in this article:

## New Commands

In recent releases, we added a bunch of new commands to IRB:

- `show_cmds`
- `show_doc`
- `edit`
- `debug` and other debugging commands:
  - `break`
  - `catch`
  - `next`
  - `delete`
  - `step`
  - `continue`
  - `finish`
  - `backtrace`
  - `info`

The number of new commands may look intimitating, so let me introduce them one by one:

### `show_cmds`

This command prints all available IRB commands with their description. It's similar to `Pry`, `Byebug`, and `debug`'s `help` command.

(It's not named `help` because IRB already uses `help` to look up API documents. I'll explain more about this in the next section.)

Output of `show_cmds` in `v1.6`:

```txt
IRB
  cwws           Show the current workspace.
  chws           Change the current workspace to an object.
  workspaces     Show workspaces.
  pushws         Push an object to the workspace stack.
  popws          Pop a workspace from the workspace stack.
  irb_load       Load a Ruby file.
  irb_require    Require a Ruby file.
  source         Loads a given file in the current session.
  irb            Start a child IRB.
  jobs           List of current sessions.
  fg             Switches to the session of the given number.
  kill           Kills the session with the given number.
  irb_info       Show information about IRB.
  show_cmds      List all available commands and their description.

Debugging
  debug          Start the debugger of debug.gem.
  break          Start the debugger of debug.gem and run its `break` command.
  catch          Start the debugger of debug.gem and run its `catch` command.
  next           Start the debugger of debug.gem and run its `next` command.
  delete         Start the debugger of debug.gem and run its `delete` command.
  step           Start the debugger of debug.gem and run its `step` command.
  continue       Start the debugger of debug.gem and run its `continue` command.
  finish         Start the debugger of debug.gem and run its `finish` command.
  backtrace      Start the debugger of debug.gem and run its `backtrace` command.
  info           Start the debugger of debug.gem and run its `info` command.

Misc
  edit           Open a file with the editor command defined with `ENV["EDITOR"]`.
  measure        `measure` enables the mode to measure processing time. `measure :off` disables it.

Context
  show_doc       Enter the mode to look up RI documents.
  ls             Show methods, constants, and variables. `-g [query]` or `-G [query]` allows you to filter out the output.
  show_source    Show the source code of a given method or constant.
  whereami       Show the source code around binding.irb again.
```

### `show_doc`

`show_doc` is an alias to the `help` command.

As mentioned above, IRB's `help` command is very different than its major counterparts', which could be confusing or even annoying.

So we want to encourage users to use `show_doc` from `v1.6` onward. And we'll convert `help` to displaying command information in the next major release.

**If you've never used `help` before, try `show_doc String#gsub` now :-)**

### `edit`

`edit` opens files in your editor (defined with `ENV["EDITOR"]`):

- `edit` - opens the current context's file.
- `edit path/to/foo.rb` - opens `foo.rb`.
- `edit Foo` - Looks up `Foo`'s source location and opens it.
- `edit Foo#bar` - Looks up `Foo#bar`'s source location and opens it.

#### Demo

![irb edit command](https://user-images.githubusercontent.com/5079556/207047097-473a547b-99ea-4142-a1aa-18ebcd5d208e.gif)

### `debug` and other debugging commands

When you look at the output of `show_cmds`, you'll notice a big `Debugging` section:

```txt
Debugging
  debug          Start the debugger of debug.gem.
  break          Start the debugger of debug.gem and run its `break` command.
  catch          Start the debugger of debug.gem and run its `catch` command.
  next           Start the debugger of debug.gem and run its `next` command.
  delete         Start the debugger of debug.gem and run its `delete` command.
  step           Start the debugger of debug.gem and run its `step` command.
  continue       Start the debugger of debug.gem and run its `continue` command.
  finish         Start the debugger of debug.gem and run its `finish` command.
  backtrace      Start the debugger of debug.gem and run its `backtrace` command.
  info           Start the debugger of debug.gem and run its `info` command.
```

All of them were added recently to bridge IRB and the [`debug` gem](https://github.com/ruby/debug).

The `debug` command does 2 things:

1. If you haven't required the `debug` gem in the current IRB session, the command would require it for you.
2. It'll then start a `debug` debugging session from the current context. It's the same as you enter a `binding.b` breakpoint.

```txt
From: test.rb @ line 4 :

    1: a = 1
    2: b = 2
    3:
 => 4: binding.irb
    5:
    6: c = 3

irb(main):001:0> debug
[1, 6] in test.rb
     1| a = 1
     2| b = 2
     3|
=>   4| binding.irb
     5|
     6| c = 3
=>#0    <main> at test.rb:4
(rdbg) # you're now using the debugger
```

(Note: it only works if the IRB session is started with `binding.irb` as it's pointless debugging the `irb` executable.)

And the rest of debugging commands (`<cmd>`) are shortcuts of `debug` + `<cmd>`.

For example, we may `step` after running `debug`. In this case, you can just run `step` in IRB, and it will start the debugging session and then run `step`.

```txt
From: test.rb @ line 1 :

 => 1: binding.irb
    2:
    3: a = 1
    4: b = 2
    5:

irb(main):001:0> step
(rdbg:irb) step
[1, 4] in test.rb
     1| binding.irb
     2|
=>   3| a = 1
     4| b = 2
=>#0    <main> at test.rb:3
(rdbg)
```

The long-term goal is to make IRB an interface of the `debug` gem, like what `pry-byebug` provides. This means you will run
certain `debug` commands without leaving the IRB session.

But for now, we hope to make the scene transition easier with these command shortcuts.

#### Debugging with a REPL (`binding.irb`) V.S. Debugging with a debugger (`binding.b`)

A REPL only gives you access to the breakpoint (`binding.irb`)'s surrounding context.

For example, if you put `binding.irb` inside a `bar` method, you can only see the locals available inside that method.

```rb
def foo(n)
  bar(n + 1)
end

def bar(x)
  binding.irb # you can only see x
  baz(x + 10)
end
```

But if you're inside a debugger, either through the `binding.b` breakpoint or the new `debug` command, you get the power to:

- Move along with the program's execution through commands like `step` or `next`.
    - `step` will let you enter the `baz` method's context.
- Navigate between the frames on the current callstack with the `up` or `down` commands.
    - You can go back to `foo` and see the value of `n`.
- Dynamically set breakpoints with the `break` or `catch` commands.

To learn more about how to utilise a debugger, please watch my talk: [ruby/debug - The best investment for your productivity](https://assets.lrug.org/videos/2022/november/stan-lo-ruby-debug-the-best-investment-for-your-productivity-lrug-nov-2022.mp4).

## Command Improvements

- `show_source` now supports Pry-like syntax: `show_source Class#method`.
- New symbol aliases:
    - `$` is an alias to `show_source`.
    - `@` is an alias to `whereami`.
    - You can add/modify aliases with `IRB.conf[:COMMAND_ALIASES]` in `.irbrc`.
        - For example, `IRB.conf[:COMMAND_ALIASES].merge!({ :'&' => :show_cmds })` aliases `&` to `show_cmds`.
- `ls` now takes `-g`/`-G` argument for greping output with regexp.
    - For example, `ls -g irb` only prints methods that match `irb`.

## Other Improvements

- Several crashing issues have now been addressed.
- It now has a refreshed [README](https://github.com/ruby/irb), where you can see all available commands.

## New Configurations

- `ENV["EDITOR"]` - Will be used by the `edit` command to open files.
- `ENV["IRB_USE_AUTOCOMPLETE"]` - When set to `"false"`, it'll disable IRB's autocompletion feature.
  - Rails `7.0.5` and `7.1+` will use this to disable autocompletion in production Rails console ([PR](https://github.com/rails/rails/pull/46656)).

## Wrapping Up

We hope these changes can make IRB more convenient and helpful. If you can, please install the latest `1.6` version and give these new features a try.

When you see any problems, please [open an issue](https://github.com/ruby/irb/issues/new). We want to detect as many issues before Ruby 3.2 as possible.

Finally, if you find posts like this interesting, please subscribe to the [Rails at Scale blog](https://railsatscale.com/). The Shopify Ruby and Rails Infrastructure Team will post articles covering different aspects of our works, such as this one.

I'll keep posting news like this here too, but you'll see a lot more content if you subscribe to the Rails at Scale blog ;-)
