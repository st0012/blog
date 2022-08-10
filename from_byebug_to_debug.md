---
title: From byebug to ruby/debug
slug: from-byebug-to-ruby-debug
seriies: ruby-debug
tags: ruby, ruby-on-rails, debugging, debug, vscode
domain: st0012.hashnode.dev
---

Switching to a new debugger and potentially changing your debugging process could be scary. So I hope this post can help you get familiar with [`ruby/debug`](https://github.com/ruby/debug) and make the migration smoother.

(In the rest of the article, I'll use `debug` to refer to `ruby/debug`)

Disclaimers:

- I'm not as experienced with `byebug` as with `debug`. So please let me know if I listed incorrect/outdated information.
- Its purpose is to give a higher-level comparison. To learn more about `debug`'s specific usages, please check its [official documentation](https://github.com/ruby/debug).
- It doesn't contain all the features but should already cover most of them.

# Advantages of `debug`

Before we get into individual features, I want to quickly mention some advantages of `debug`:

- Colorized output

    <img width="50%" src="https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ir3yykmwcaw3ct6h7y54.png">

- [Backtrace with method/block arguments](#backtrace)

    <img width="60%" src="https://user-images.githubusercontent.com/5079556/183871321-b2feb33e-4733-41d5-9266-3e4c1f1daa29.png">

- Immune from the [compatibility issue with Zeitwerk](https://github.com/deivid-rodriguez/byebug/issues/564)
- Powerful [breakpoints](#breakpoint) and [tracers](#tracer)
- [Convenient remote debugging and VSCode integration](#remote-debugging)
    - [Setting it up with VSCode](https://st0012.dev/setup-ruby-debug-with-vscode)

# Disadvantages of `debug`

- Less flexible on [thread control](#thread-control)
- Doesn't work well with Fiber yet
- Doesn't have `pry` integration like `pry-byebug`
- No per-project configuration

# `byebug` vs `debug`

## Installation

(Although Ruby `3.1` comes with `debug` `v1.4`, I recommend always using its latest release)

|| byebug | debug |
|---|---|---|
| Supported Ruby version | 2.5+ | 2.6+ |
| Gem install | `gem install byebug` | `gem install debug` |
| Bundler | `gem "byebug"` | `gem "debug"` |
| Dependencies | No | `irb`, `reline` |
| Has C extension | Yes | Yes |
| Latest version | [11.1.3](https://github.com/deivid-rodriguez/byebug/releases/tag/v11.1.3) (23 Apr 2020) | [1.6.2](https://github.com/ruby/debug/releases/tag/v1.6.2) (10 Aug 2022) |

## Start Debugging

|| byebug | debug |
|---|---|---|
| Via executable | `byebug foo.rb` | `rdbg foo.rb` |
| Via debugger statement | `byebug` | `binding.break`, `binding.b`, `debugger` (the same) |
| Via requiring | No | `require "debug/start"` |

## User Experience Features

|| byebug | debug |
|---|---|---|
| Colorizing | No | Yes |
| Command history | Yes | Yes |
| Help command | `h[elp]` | `h[elp]` |
| Edit source file | `edit` | `edit` |

## Evaluation

Code evaluation in REPL is more or less the same between `byebug` and `debug`, that is:

- If the expression doesn't match a console command, like `my_method(arg)`, it'll be evaluated as Ruby code
- If the expression matches a console command, like `n`, you can use console commands to evaluate it

|| byebug | debug |
|---|---|---|
| Execute debugger commands | `<cmd>` | `<cmd>`
| Avoid expression/command conflicts | `eval <expr>` | `pp <expr>`, `p <expr>`, or `eval <expr>`

## Flow Control & Frame Navigation

`debug` provides the same stepping commands as `byebug` does. So `byebug` users can switch to `debug` without learning new behaviors.

|| byebug | debug |
|---|---|---|
| Step in | `s[tep]` | `s[tep]` |
| Step over | `n[ext]` | `n[ext]` |
| Finish | `fin[ish]` | `fin[ish]` |
| Move to `<id>` frame | `f[rame] <id>` | `f[rame] <id>` |
| Move up a frame | `up` | `up` |
| Move down a frame | `down` | `down` |
| Move up n frames | `up <n>` | No |
| Move down n frames | `down <n>` | No |
| Continue the program | `c[ontinue]` | `c[ontinue]` |
| Quit the debugger | `q[uit]` | `q[uit]` |
| Kill the program | `kill` | `kill` |

## Thread Control

As previously mentioned, `debug` stops all threads when suspended and doesn't allow per-thread management at the moment. So it has simpler thread-related commands.

|| byebug | debug |
|---|---|---|
| Thread suspension | Only the current thread | All threads |
| List all threads | `th[read] l` | `th[read]` |
| Switch to thread | `th[read] switch <id>` | `th[read] <id>` |
| Stop a thread | `th[read] stop <id>` | No |
| Resume a thread | `th[read] resume <id>` | No |

## Breakpoint

Although `byebug` already has decent breakpoints support, `debug` takes it to another level by:

- Allowing breakpoints to execute commands with `pre:` and `do:` options
- Supporting call-location-based triggering condition with the `path:` option
- Supporting specialized `catch` and `watch` breakpoints

### Setting Breakpoints

|| byebug | debug |
|---|---|---|
| On line | `b[reak] <line>` | `b[reak] <line>` |
| On file:line | `b[reak] <file>:<line>` | `b[reak] <file>:<line>` |
| On a method | `b[reak] <class>#<method>` | `b[reak] <class>#<method>` |
| With a condition | `b[reak] ... if <expr>` | `b[reak] ... if: <expr>` |
| With a path condition | No | `b[reak] ... path: /path/` |
| To run a command and continue | No | `b[reak] ... do: <cmd>` |
| To run a command before stopping | No | `b[reak] ... pre: <cmd>` |
| Exception breakpoint | No | `catch <ExceptionClass>` |
| Instance variable watch breakpoint | No | `watch <@ivar>` |

### Managing Breakpoints

|| byebug | debug |
|---|---|---|
| List all breakpoints | `info breakpoints` | `b[reak]` |
| Set a breakpoint | `b[reak] ...` | `b[reak] ...` |
| Delete a breakpoint | `del[ete] <id>` | `del[ete] <id>` |
| Delete all breakpoints | `del[ete]` | `del[ete]` |

## Information

### Backtrace

Backtrace in `debug` contains values of method call or block arguments. This small improvement can save you from inspecting them manually between frames.

<img width="60%" src="https://user-images.githubusercontent.com/5079556/183871321-b2feb33e-4733-41d5-9266-3e4c1f1daa29.png">

The filtering support can help you focus on relevant frames and ignore the ones from frameworks or libraries.

|| byebug | debug |
|---|---|---|
| Show backtrace | `where`, `backtrace`, `bt` | `backtrace`, `bt` |
| Displaying method/block arguments in backtrace | No | Yes |
| Filter backtrace | No | `bt /regexp/` |
| Limit the number of backtraces | No | `bt <n>` |

### Varibles/Constants

|| byebug | debug |
|---|---|---|
| Show local varibales | `var local` | `info l` |
| Show instance variables | `var instance` | `info i` |
| Show global varibales | `var global` | `info g` |
| Show constants | `var const` | `info c` |
| Show only arguments | `var args` | No (included in `info l`) |
| Filter variables/constants by names | No | `info ... /regexp/` |

### Methods

While `debug` doesn't have a dedicated command to list an object's methods, it has a `ls` command that's similar to
`irb` or `pry`'s.

|| byebug | debug |
|---|---|---|
| `obj.methods` | `method instance obj` | No |
| `obj.methods(false)` | `method obj.class` | `ls obj` |

## Tracing

I will write another dedicated article to introduce `debug`'s tracing functionalities, but I encourage you to already give them a try from time to time, especially:

- `trace exception` - to monitor exceptions raised in your code. You may be surprised by the exceptions raised and rescued in your application under the surface.
- `trace object <expr>` - to observe an object's activities: receiving a method call or being passed to a method call.

    ![trace object](https://user-images.githubusercontent.com/5079556/183893134-f7e3e26b-c5ef-49cb-9c6b-9595c373c8c1.png)

|| byebug | debug |
|---|---|---|
| Line tracer | `set linetrace` | `trace line` |
| Global variable tracer | `tarcevar` | No |
| Allow multiple tracers | No | Yes |
| Method call tracer | No | `trace call` |
| Exceptions tracer | No | `trace exception` |
| Ruby object tracer | No | `trace object <expr>` |
| Filter tracing output | No | `trace ... /regexp/` |
| Disable tracer | `set linetrace false` | `trace off [tracer type]` |
| Disable the specific tracer | No | `trace off <id>` |

## Configuration

`debug` supports a [wide range of configurations](https://github.com/ruby/debug#configuration) to make it fit your needs. But it doesn't support per-project RC files due to security concerns.

|| byebug | debug |
|---|---|---|
| List all configs | No | `config` |
| Show a config | `show <name>` | `config show <name>` |
| Set a config | `set <name> <value>` | `config set <name> <value>` |
| RC file name | `.byebugrc` | `.rdbgrc` (or `.rdbgrc.rb` for Ruby script) |
| RC file locations | `$HOME` and project root | `$HOME` |

## Remote Debugging

Remote debugging functionality is becoming crucial as we start containerizing our development environment and don't always have direct standard IO access.
It's also crucial for connecting to different debugger clients like VSCode or Chrome.

So if you have needs in this area, migrating to `debug` is your best option.

|| byebug | debug |
|---|---|---|
| Connect via TCP/IP | Yes | Yes |
| Connect via Unix Domain Socket | No | Yes |
| VSCode integration through [Debug Adapter Protocol (DAP)](https://microsoft.github.io/debug-adapter-protocol/specification) | No | Yes |
| Chrome integration through [Chrome DevTools Protocol (CDP)](https://chromedevtools.github.io/devtools-protocol/) | No | Yes |

# Wrapping Up

To most users, switching from `byebug` to `debug` should take little effort. But the potential productivity gain from `debug`'s features could be significant.

`byebug` is a great debugger and has served the community well for many years.
But as its development slows down (last release was 2 years ago) and `debug` receives constant updates from the Ruby core, the gap between them will only widen.

So adding `debug` to your toolbelt and exploring its powerful features will be a great productivity investment for the long term.

# References

- [Debugger commands comparison sheet](https://docs.google.com/spreadsheets/d/1TlmmUDsvwK4sSIyoMv-io52BUUz__R5wpu-ComXlsw0/edit#gid=0) by [@ko1](https://github.com/ko1)
- [Byebug's official guide](https://github.com/deivid-rodriguez/byebug/blob/master/GUIDE.md)
- [ruby/debug's official documentation](https://github.com/ruby/debug)
