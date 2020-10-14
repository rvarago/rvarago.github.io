---
layout: "post"
title:  "Be Proud of your Shell Scripts"
tags:   linux bash
---

> Automation is not only a fancy trend but rather a reality that plays a vital role in the Software Engineering world. Besides the multitude of "modern" tools, the Shell, our good and old friend from '70s, still plays and will keep playing a major role. Therefore, it deserves more care. Let's be prouder of our shell scripts.

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

## Shell Scripts _are_ Code

When we're writing production code we strive to apply the best practices of Software Engineering, for instance:

  * Don't Repeat Yourself.
  * Keep It Simple.
  * You Ain't Gonna Need It.

Think about it: What do we when we see multiple representations for the same piece of knowledge in the code?

 **Answer:** We extract the common knowledge to a module (function, class, package, etc) and reuse it wherever needed. It's almost done by instinct.

It's our nature, we strive to deliver the best code possible, elegantly and most efficiently, that is effective!

How about naming guidelines? It's been a long time since we decided that naming everything _a_, _aux_ or _count_ may not always be a good idea.

That cryptic style of programming that required one comment per each line of code to explain something that should've been obvious in the code is in the past! Sure? Not exactly, not always.

We have been dedicating loads of efforts in writing clean production code (code that solves the business problem), we have awesome tooling orchestrating our containers in the cloud, and so on. Yet, we're still writing cryptic Shell Scripts, the scripts that usually responsible for some aspects of our application's infrastructure (therefore, part of our applications).

We should NOT do that.

No matter if we're employing cutting-edge technologies to develop our applications, we probably have some Shell Script underneath it here and there (the same argument applies to other scripting languages as well). Even for simple tasks like pushing a git tag and then running `kubectl`. They are part of our applications, they usually are the glue which gets applications up and running. Hence, adding value to our customers.

> A Shell Script **is** code and shall be treated with care.

Granted, programming languages used understood by shells don't have the most pleasant syntaxes compared to modern languages and lack many abstractions as well. Still, it's a tool, which should be carefully used.

Writing shell scripts might be fun too, sort of.

## Guidelines

Writing idiomatic, modular, and maintainable scripts is not the easiest task. It involves a lot of discipline and practice, but we can get close.

I would like to share a few guidelines to help. Before that, keep this in mind, please:

> Those are simply guidelines based on my experience, **NOT** rules.

### Be Restrict

It's quite easy to commit mistakes when writing shell scripts. Let's talk about two of them and see how to prevent them from happening.

#### Abort on Failure

Shell scripts are made of a series of commands that execute one after the other. Each command yields an execution status that states whether the command has succeeded or not. Conventionally, execution status `0` means success, whereas anything other than `0`  means failure.

Generally, when a command fails, we would like to abort the script immediately. That's because we might have entered in an invalid state and we should not proceed further.

However, by default, shells normally ignore execution status within a script. Instead, they simply move forward to the next command as if nothing bad had happened.

Fortunately, its quite simple to change this behaviour, and aborts the script whenever a command fails. We just need to enable set the `-e` flag by before any other command that should be checked for failure (most often at the very begging of the file):
    
    set -e

As soon as the first command fails, the script will abort.

Rarely, yet sometimes desirable, we want to ignore the status of command and continue the script even when such a command fails. Say, some action is expected to occasionally fail so that we can retry later. For those scenarios, we can use the `||` operator:
    
    commandThatMightFail || true 

#### Abort on Undefined Variables

Another mistake we sometimes commit is reading an undefined variable, e.g. by mistyping its name.

In that case, by default, the expression expands to an empty string. That might come as surprising, and from to my mind, it is. Normally, we would prefer to abort the script as soon as we attempt to reference an undefined variable.

Fortunately, we can change that. Again, we have to enable the `-u`:
    
    set -u

Should we now attempt to reference an undefined variable, then the script will abort.

Again, there might be specific cases where an undefined variable makes sense (perhaps an optional command-line argument?), and hence we want the script to move on even if we attempt to reference such a variable. Similarly to what we saw before, we can provide a fallback value for when the variable was not previously declared:
    
    ${variable:-defaultValue}

### Return Proper Execution Status

We are generally working towards more automation, where tools drive other tools, and most often without human intervention. Say, we have a bunch of pipelines, where processes execute in sequence to achieve the result, e.g. deploy to production.

Therefore, we need to have ways so that tool A can monitor the execution of tool B, and makes decisions based the result of B. Say, if an automated-test failed, then we should rollback, instead of committing a deploy to production.

The simplest way to monitor the execution of a process is by inspecting its execution status. A large number of tools rely on the assumption that we return proper execution statuses to report success or different sorts of failures.

We return the execution status in shell scripts with the `exit` command, which accepts an integer representing the status code:
    
    exit statusCode 

See [FreeBSD sysexits](https://www.freebsd.org/cgi/man.cgi?query=sysexits&apropos=0&sektion=0&manpath=FreeBSD%204.3-RELEASE&format=html) for a list of well-known status codes that we can make use. 

### Encapsulate Knowledge into Functions

Whenever we see a duplication of the same piece of knowledge in our code base, it's nearly impossible to not extract them into a module and then reuse it at all the places. That gives suitable names to pieces of logic and leverages code sharing and reuse. 

But somehow, it's not always the case when we talk about shell scripts. We sometimes don't put so much attention to it and may end up with similar steps repeated over and over again. Ideally, we should extract commonalities and reuse them.

There are many benefits of such practice, for example:

  * Attributes names to piece of logics.
  * Consolidates changes into a single place.
  * Prevents inconsistency between modules.

> Structure matters.

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

main() {
  turn_display_on "gen2" # This should send 'On' to 'usb0'.
}

main
```

The "logical" entry-point is the `main` function, which we immediately invoked upon start-up.
Moreover, the call to `get_device_node_from_generation` from `turn_display_on` occurs within a sub-process by using `$()`, where the callee communicates the result back to the caller by `echo`ing it without a trailing newline character.

Further, I usually assign the argument (`$1`, `$2`, etc) to a named variable at the beginning of each function. That's a matter of style, and I like it because it's the closest I could get to proper formal-parameters, e.g. `get_device_node_from_generation(generation){ # use generation }`.

## Example: Deployment System

To illustrate the concepts, I'll provide an example. It's a simplistic version, but models a fairly common use-case:

> A Command-Line Interface (CLI) that deploys an application into a given environment.

    For the sake of demonstration, I will be faking the real deployment with simple prints, but it could evolve to complex use-cases (Docker, Kubernetes, cloud providers, etc).

We could code up an ad-hoc script, in which we would have a single block of commands without a strong notion of a structure. It's enough for simple cases, but it can easily make things hard to understand and maintain once new requirements come in. That's why we want to keep it structured, with concerns encapsulated into functions whose names emphasize their purposes.

We require that our CLI should accept the environment and the application as arguments, switch to the cluster corresponding to the given environment, and then install the application into that cluster.

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

The most important thing is that we followed our advice and split the logic into separate functions, which are coordinated by the `main` function ("entry-point" of the script). We have wrapped each concern into distinct functions with names telling us what they do

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

Nowadays, we have a large number of tools helping us to improve our products, while making our lives easier by automating manual tasks. But, they don't replace Shell Scripts, which is, most of the time, the least common denominator for Linux based applications. Furthermore, learning how to script can also help you when hacking on Linux, perhaps you could maintain part of a Distro in the future?

Although unfortunately, sometimes we may overlook our scripts and forget to follow well-established practices that we're used to applying in our production code. The outcome is scripts that are difficult to understand, debug, fix, and ultimately improve.

We should strive to avoid this kind of scenario. We should write cleaner scripts and leverage their power to their maximum.

Granted, the syntax might feel awkward and archaic, but that doesn't mean we absolutely can't enjoy writing our beloved scripts.

I've covered just the tip of the iceberg, that are many more topics, such as organizing scripts and sourcing, helpful commands and their syntax, traps, portability, etc. Yet, I encourage the reader to look up for more information and above all things, have fun while doing so.

## References

[1] DevOpsCon 2018. Shell Ninja: Mastering the Art of Shell Scripting. Dr Roland Huß.

[2] [The Linux Documentation Project.](https://www.tldp.org/guides.html)

[3] [Safer bash scripts with 'set -euxo pipefail'.](https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/)

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
