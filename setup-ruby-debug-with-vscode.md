---
title: Setup ruby/debug with VSCode
subtitle: setup-ruby-debug-with-vscode
tags: ruby, ruby-on-rails, debugging, debug, vscode
domain: st0012.hashnode.dev
---

Do you know Ruby's official debugger [`ruby/debug`](https://github.com/ruby/debug) provides out-of-box integration with VSCode? If you haven't tried it yet or having difficulty making it work, I hope this short post will help you set it up.

### Basic Setup

1. Install the [VSCode rdbg extension](https://marketplace.visualstudio.com/items?itemName=KoichiSasada.vscode-rdbg) in VSCode
2. Create the `launch.json` file
   1. Click `Run and Debug` button on the left side
   2. Click `create a launch.json file` - this is quite small and under `To customize Run and Debug`
   3. Save the created `launch.json`
3. Put `gem "debug", require: false` in your `Gemfile` and run `bundle install`

### Debug Simple Ruby Script

If you want to debug a simple Ruby script, you can follow [these steps](https://github.com/ruby/debug#using-vscode).

### Debug Rails/Web Applications

If you want to debug a Rails/web application, do these instead:

1. Open the files in VSCode and add some breakpoints.
2. Start your program with the `rdbg` executable - `bundle exec rdbg --open -n -c -- bundle exec rails s`
    - `--open` or `-O` means starting the debugger in server mode
    - `-n`  means don't stop at the beginning of the program, which is usually somewhere at rubygems, not helpful
    - `-c`  means you'll be running a Ruby-based command
3. Go back to VSCode's `Run and Debug` panel, you should see a grean play button
4. Click the dropdown besides the button and select `Attach with rdbg`
5. Click the play button
6. It should now connect the VSCode to the debugger
  - If it stops at somewhere in your web server (like [puma](https://github.com/puma/puma)). Hit `continue` or `F5`. This will be resolved in the next `1.6.0` release.
7. Send some requests and it should stop at your breakpoints

**Video**

![Connect the debugger with your Rails server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v0elr5otucfojq1vdoyz.gif)

I also have built this [`repl.it`](https://replit.com/@st0012/rubydebug-demo?v=1) to let you try out [`ruby/debug`](https://github.com/ruby/debug)'s console commands in your browser (it'll only take you 5 minutes).

If you want to see more articles/tips about the powerful features [`ruby/debug`](https://github.com/ruby/debug) has, you can follow me on [Twitter](https://twitter.com/_st0012) 🙂




