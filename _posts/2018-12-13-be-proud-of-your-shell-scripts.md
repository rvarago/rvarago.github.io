---
layout: "post"
title:  "Be Proud of your Shell Scripts"
---

> Automation is not only a fancy trend but rather a reality that plays a vital role in the Software Engineering world. Besides the multitude of "modern" tools, the Shell, our good and old friend from '70s, still plays and will keep playing a major role. Therefore, it deserves more care. Let's be prouder of our Shell Scripts.

* * *

|![Cowemoji.](/assets/img/2018-12-13-be-proud-of-your-shell-scripts_0.png)|
|:--:| 
| *Cowemoji [CC0], from Wikimedia Commons.*|

_This article was mainly inspired by the talk: "Shell Ninja: Mastering the Art of Shell Scripting" given by Dr Roland Huß at DevOpsCon 2018_.

 **NOTE**: I'll be using the _Bourne Again SHell_ (_bash_). Hence, snippets may need adjustments on other Shells.

## Automate All the Things!

The goal is to delegate to computers the tedious and error-prone manual tasks that we used to do in the past. We can list a few tasks with varying degrees of complexity that deserves automation:

  1. Backups.
  2. Updates.
  3. Releases.
  4. Deploys.

By putting these tasks in a reproducible and automatic script, we can avoid wasting our time typing the sequence of commands over and over again, and, of course, the bugs that come with this process.

Moreover, we can now put our minds to work on solving problems and delivering awesome applications to our customers. Well…At the end of the day, that is what matters.

## Shell Scripts Equal Code

When we're writing production code we strive to apply the best practices of Software Engineering, for instance:

  * Don't Repeat Yourself.
  * Keep It Simple.
  * You Ain't Gonna Need It.

Think about it: What do we when we see multiple representations for the same piece of knowledge in the code?

 **Answer:** We extract the common knowledge to a module (function, class, package, etc) and reuse it wherever needed. It's almost done by instinct.

It's our nature, we strive to deliver the best code possible, elegantly and most efficiently, that is effective!

How about naming guidelines? It's been a long time since we decided that naming everything _a_, _aux_ or _count_ may not always be a good idea.

That cryptic style of programming that required one comment per each line of code to explain something that should've been obvious in the code is in the past! Sure? Not exactly, not always.

We've been dedicating loads of efforts in writing clean production code (code that solves the business problem), we've been using awesome tooling to orchestrate our containers in the cloud, etc. Yet we're still writing cryptic Shell Scripts, the scripts that usually responsible for some
aspects of our application's infrastructure (therefore, part of our applications).

We should NOT do that.

No matter if we're employing cutting-edge technologies to develop our applications, we probably have some Shell Script underneath it here and there (the same argument could be applied to other scripting languages as well). Even for simple tasks like pushing a git tag and then running `kubectl`. They're part of our applications, they're part of the process to get applications up and running. Hence, adding value to our customers.

> A Shell Script **is** code and shall be treated with care.

Yes, I know, shells don't offer the most beautiful and modern syntaxes, and they lack many abstractions that modern languages support. Still, it's a tool, which should be carefully used. Come on, writing scripts might be fun too, sort of.

## Guidelines

Write idiomatic, modular, and maintainable scripts is not the easiest task, but with discipline and practice, it's surely possible to do.

I'd like to share some guidelines to help. But before, keep this in mind:

> Those are simply guidelines based on my experience, **NOT** rules.

### Be Restrict

It's pretty easy to commit mistakes when writing Shell Scripts. Let's talk about two of them.

#### Abort on Failure

A Shell Script, at the very basic level, represents a series of commands that are executed one after the other. Each command yields an execution status that says if the command has succeeded, or not. By convention, the execution status `0` means success and all other statuses mean failures.

In general, if a command fails, we'd like to stop the script, because we're probably at an invalid state and we shouldn't proceed.

But, by default, Shell ignores the execution status, and simply moves forward to the next command and so on.

Fortunately, it's easy to change this behaviour and fails the script if some command has failed. To do it, we need to enable the `-e` flag by adding the following line before any other command that should be validated:
    
    set -e

So, as soon as the first command fails, the Shell will stop the script.

But, in some specific cases, we'd like to continue the script even if some command has failed. For instance, an optional action may or may not be
executed, thus its status is not relevant. For this scenario, we can use the `||` operator that will yield a success status code even if the command has failed:
    
    commandThatMightFail || true 

#### Abort on Undefined Variables

Another common mistake related to Shell Scripts is reading an undefined variable, this usually happens by mistyping the variable name.

When that happens, the default behaviour of the Shell is to expand the reference to an empty string.

That isn't the behaviour that we normally expect. Normally, we'd like to stop the script as soon as we try to reference an undefined variable.

Fortunately, it's also easy to change it. We just need to enable the `-u` flag by adding the following line before any other command that
should be validated:
    
    set -u

Now, as soon as the first undefined variable is referred, the script will abort.

Once again, there might be specific cases where an undefined variable makes sense, and so, we wouldn't like to stop the script. An example is a variable that represents an optional command-line argument. In this case, we can provide a fallback value to be used when the variable wasn't previously declared:
    
    ${variable:-defaultValue}

### Return Proper Execution Status

We're generally working towards more automation, where tools drive other tools, sometimes without human intervention. For instance, we have pipelines, where processes are sequentially executed towards a result, like deploy to production.

To have that level of automation working properly, we need to provide a way such that a tool can monitor the execution of other tools that it invokes and hence decide whether it should take "this" or "that" action, like rollback or commit a deploy to production.

The most basic way to monitor the execution of a process by inspecting its execution status. A large number of tools are based on the assumption that we're going to provide an execution status, to report success or different kinds of failure.

It's straightforward to provide the execution status in Shell Script, we just need to use the `exit` command, supplying an integer called status code as its argument:
    
    exit statusCode 

Of course, we need to remember to add it to each possible branch inside your control flow, where we want to finish the script.

See [FreeBSD sysexits](https://www.freebsd.org/cgi/man.cgi?query=sysexits&apropos=0&sektion=0&manpath=FreeBSD%204.3-RELEASE&format=html) for a list of well-known status codes that we can make use. 

### Encapsulate Knowledge into Functions

Whenever we see a duplication of the same piece of knowledge in our code base, it's nearly impossible to not extract them into a module and then reuse it at all the places. That gives suitable names to pieces of logic and leverages code sharing and reuse. 

But somehow, it's not always the case when we're talking about Shell Scripts, we don't put so much attention to it, and it's easy to come up with the same step being repeated in different scripts inside the same project, or even inside the same project. We should extract them and reuse. 

There are so many benefits of such practice, for example:

  * Attributes names to piece of logics.
  * Consolidates changes into a single place.
  * Prevents inconsistency between modules.

It's important to write concise functions that have unique concerns. Moreover, avoid coupling by providing to the function the information that it needs by arguments whenever makes sense to reuse the function with another set of conditions. Hence, the function can be easily plugged. But don't accumulate multiple concerns inside the same function.

An important thing to bear in mind when writing a function is to keep the variables declared inside it at the minimum necessary scope. To achieve this behaviour in Shell Scripts, we declare such variables as `local`, so the variables are only visible within the block in which they appear. Therefore, for functions, the scope is the function's body:
    
    local myVariable=myValue

As an example of functions, let's say we have an embedded system with a script that controls an LCD through a serial port, say we want to turn it on. However, the exact device node that corresponds to the display varies depending on the generation of the embedded system (gen1, gen2, and gen3).

Presumably, that's one way that could write such a script: 

```bash
get_device_node_from_generation() {
  local generation="$1"
  
  case "${generation}" in
    gen1)
      echo -n "ttyS1"
      ;;
    gen2)
      echo -n "usb0"
      ;;
    gen3)
      echo -n "lcd"
      ;;
  esac
}

turn_display_on() {
  local generation="$1"

  local display_id=$(get_device_node_from_generation "${generation}")

  echo "On" > "${display_id}"
}

turn_display_on "gen2" # This should send 'On' to 'usb0'.
```

Notice that we invoked `get_device_node_from_generation` from `turn_display_on` within a sub-process by using `$()` where the callee communicates the result back to the caller by `echo`ing.

Further, I usually assign the argument (`$1`, `$2`, etc) to a named variable in the first line of each function. That's a matter of style, I like this practice because it's the closest I could get to proper parameters, e.g. `get_device_node_from_generation(generation)`.

## Example: Deployment System

To illustrate the concepts, I'll provide an example. It's a simplistic version, but models a fairly common use-case:

> A Command-Line Interface (CLI) that deploys an application into a given environment.

    For the sake of demonstration, I will be faking the real deployment with simple prints, but it could be adapted to complex use-cases (Docker, Kubernetes, cloud providers, etc).

We could code up an ad-hoc script, in which we would have a single block of commands without a strong notion of a structure. It's enough for simple cases, but it can easily make things hard to understand and maintain once new requirements come in. That's why we want to keep it structured, with concerns encapsulated into functions whose names emphasize their purposes.

We require that our CLI should accept the environment and the application as arguments, switch to the cluster that corresponds to the specified environment, and then install the application into that cluster.

Based on the requirements, we can already identify two major concerns:

1. Switch to the cluster.
2. Install the application.

Those probably deserve their functions. Besides the major requirements, we also need to:

1. Map an environment to a cluster.
2. Parse command-line options, which we shall do using [getopts](https://sookocheff.com/post/bash/parsing-bash-script-arguments-with-shopts/).
3. Display help messages.
4. Log information and errors.
 
Once again, those might be better off encapsulated into functions.

Here's how an implementation might look like: 

<script src="https://gist.github.com/rvarago/26a1d7000f9f05c48a6db4b666ec080f.js"></script>

The most important thing is that we followed our advice and split the logic into separate functions, which are coordinated by the `main` function ("entry-point" of the script). Each concern is wrapped into a function with a name that describes what it does.

Here's the output when requesting for help:

```bash
λ ./fake-deploy.sh -h
Deploy my awesome application into a given environment

USAGE:
  fake-deploy.sh [OPTIONS] APPLICATION

OPTIONS:
  -h                      Show this message
  -e <ENVIRONMENT>        Select the environment: testing, production [default: testing]
```

## Conclusion

Nowadays, we have a vast number of awesome tools that help us to improve our products and make our lives easier by automating manual tasks. But, they don't replace Shell Scripts, which is, most of the time, the least common denominator for Linux based applications. Furthermore, learning how to script can also help you when hacking on Linux, perhaps you could maintain part of a Distro in the future?  

Although unfortunately, sometimes we may overlook our scripts and forget to follow well-established practices that we're used to applying in our production code. The outcome is: scripts that are difficult to understand, debug, fix, and improve.

We should strive to avoid this kind of scenario. We should write clearer scripts and leverage their power to their maximum.

Granted, the syntax might feel awkward and archaic, but that doesn't mean we absolutely can't enjoy writing our beloved scripts.

I've covered just the tip of the iceberg, that are many more topics, such as organizing scripts and sourcing, helpful commands and their syntax, traps, portability, etc. Yet, I encourage the reader to look up for more information and above all things, have fun while doing so. 

## References

[1] DevOpsCon 2018. Shell Ninja: Mastering the Art of Shell Scripting. Dr Roland Huß.

[2] [The Linux Documentation Project.](https://www.tldp.org/guides.html)

[3] [Safer bash scripts with 'set -euxo pipefail'.](https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/)

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
