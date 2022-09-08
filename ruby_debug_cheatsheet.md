---
title: ruby/debug cheatsheet
slug: ruby-debug-cheatsheet
seriies: ruby-debug
tags: ruby, ruby-on-rails, debugging, debug
domain: st0012.hashnode.dev
---

This cheatsheet can help you get started with [ruby/debug](https://github.com/ruby/debug) as well as use it in your daily development. It's **not** an exhausting list of its features or commands, so please go through its document as well.

If you're migrating from `byebug`, I also recommend checking my [byebug to ruby/debug migration guide](https://st0012.dev/from-byebug-to-ruby-debug).

## Getting Started

1. Add `gem "debug"` to your `Gemfile`
2. Run `bundle install`
3. Add `require "debug"`
4. Use `binding.b` or `debugger` to set initial breakpoints (they're same)

### The `rdbg` Executable

(Running with `bundle exec` is usually recommended)

| Action | Command |
|---|---|
| Debug a Ruby script with executable | `$ rdbg foo.rb` |
| Debug a Ruby command with executable | `$ rdbg -c -- bundle exec <cmd>` |
| Debug without the initial stop | `$ rdbg -n ...` |

### Need Any Help?

| Action | Command |
|---|---|
| See document of all the commands | `h[elp]` |
| See document of `<cmd>` | `h[elp] <cmd>` |
| See configurations | `config` |
| Learn how to configure the debugger | `h config` |

## Step Debugging & Frame Navigation

| Action | Command |
|---|---|
| Step in | `s[tep]` |
| Step over | `n[ext]` |
| Finish | `fin[ish]` |
| Move to `<id>` frame | `f[rame] <id>` |
| Move up a frame | `up` |
| Move down a frame | `down` |
| Continue the program | `c[ontinue]` |
| Quit the debugger | `q[uit]!` |
| Kill the program | `kill` |

## Breakpoint Commands

### Setting Breakpoints

| Target | Command |
|---|---|
| A line | `b[reak] <line>` |
| A file:line | `b[reak] <file>:<line>` |
| An instance method | `b[reak] <class>#<method>` |
| An object's method | `b[reak] <obj>.<method>` |
| A class method | `b[reak] <class>.<method>` |
| An exception | `catch <ExceptionClass>` |
| Mutation of instance variable | `watch <@ivar>` |

#### Options (applies to all 3 breakpoint commands)

| Condition Type | Option |
|---|---|
| With an if condition | `... if: <expr>` |
| With a path condition | `... path: /path/` |
| To run a command and continue | `... do: <cmd>` |
| To run a command before stopping | `... pre: <cmd>` |

### Managing Breakpoints

| Action | Command |
|---|---|
| List all breakpoints | `b[reak]` |
| Delete a breakpoint | `del[ete] <id>` |
| Delete all breakpoints | `del[ete]` |

## Information Commands

| Information | Command |
|---|---|
| Current scope's outlne (methods and avialble ivars, locals) | `ls` |
| `obj`'s outlne (methods and avialble ivars, locals) | `ls obj` |
| Backtrace | `bt` |
| Only `<n>` backtrace | `bt <n>` |
| Source code | `l[ist]` |
| Local variables, instance variables, and current scope's constants | `i[nfo]` |
| Local varibales | `i[nfo] l` |
| Instance variables | `i[nfo] i` |
| Global varibales | `i[nfo] g` |
| Constants | `i[nfo] c` |
| Filter variables/constants by names | `i[nfo] ... /regexp/` |

## Scriptable Breakpoints

| Description | Code |
|---|---|
| Run the `<cmd>` and stop the program | `binding.b(pre: "<cmd>")` |
| Run the `<cmd>` and continue the program | `binding.b(do: "<cmd>")` |
| Run multiple commands | `binding.b(do: "<cmd1> ;; <cmd2>")` |

### My Favourites

| Description | Code |
|---|---|
| Glance through variables before I start typing | `binding.b(pre: "i")` |
| Quickly get the surrounding context | `binding.b(pre: "ls ;; bt 10")` |
| Start tracing exceptions | `binding.b(do: "trace exception")` |
| Stop me when there's an exception | `binding.b(do: "catch StandardError")` |

