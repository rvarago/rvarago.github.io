---
layout:	"post"
title:	"Be proud of your Shell Scripts"
---

The goal you delegate to computers the tedious and error-prone manual tasks
that we used to do. We can enumerate a few tasks with various degrees of
complexity that deserves a good level of…

* * *

![](/assets/img/2018-12-13-be-proud-of-your-shell-scripts_0.png)

Cowemoji [CC0], from Wikimedia Commons

> Automation is more than a tendency, it's a reality that plays a vital role
in the Software Engineering world. Besides the multitude of "modern" tools,
the Shell, our good and old friend from '70s, still plays and will keep
playing a spotlight position. Therefore, it deserves much more care from us.
Let's be prouder of our Shell Scripts again.

* * *

 _This article was mainly inspired by the talk: "Shell Ninja: Mastering the
Art of Shell Scripting" given by Dr. Roland Huß at DevOpsCon 2018._

* * *

 **NOTE** : I'll be using the _Bourne Again SHell_ ( _bash_ ). Hence, some
snippets may not be fully functional on other Shells.

#### Automate it all!

The goal is to delegate to computers the tedious and error-prone manual tasks
that we used to do in the past. We can list a few tasks with varying degrees
of complexity that deserve automation:

  1. Backups
  2. Updates
  3. Releases
  4. Deploys

By putting these tasks in a reproducible and automatic script, we can avoid
wasting our time typing the sequence of commands over and over again, and, of
course, the bugs that come with this process.

Moreover, we can now put our minds to work on solving problems and delivering
awesome applications to our customers. Well…At the end of the day, that is
what really matters.

#### Shell Scripts are code

When we're writing production code we strive to apply the best practices of
Software Engineering, for instance:

  * Don't Repeat Yourself
  * Keep It Simple
  * You Ain't Gonna Need It

Think about: what do we do when we see multiple representations for the same
piece of knowledge in the code?

 **Answer:** We extract the common knowledge to a module (function, class,
package, etc) and reuse it wherever necessary. It 's almost done by instinct.
It's our nature, we strive to deliver the best code possible, simpler and most
efficient. Effective!

How about naming guidelines? It's been a long time since we decided that the
name _count_ for a variable that represents the number of items in a
repository isn 't a good idea. Am I right?

That cryptic style of programming that required one comment per each line of
code to explain something that should've been obvious in the code is in the
past! Sure? Not exactly, not always.

We've been putting most of our efforts to write elegant production code, the
one that solves the business problem at first place, we've been using awesome
tooling to orchestrate our containers in the cloud, etc. But, unfortunately,
we're still writing cryptic Shell Scripts that are responsible for some
aspects of our application's infrastructure. Therefore, part of the
application.

We should NOT do this.

No matter if we're applying cutting-edge technology to develop our
applications, we probably have some Shell Script underneath it. Even for
simple tasks like pushing a git tag and then running kubectl. They're part of
your application, they're part of the process to get your application up and
running. Hence, adding value to our customers.

> Shell Script **is** code and should be treated as such.

Yes, I know, Shell Script doesn't offer the most beautiful syntax, and it
lacks a lot of abstractions that other languages support. But still, it's a
valuable tool, and it means that we should be careful. Come on, writing code
for Shell can be fun too.

#### Some Guidelines to Write Better Shell Scripts

Write idiomatic, modular, and maintainable Shell Scripts is not the easiest
task, but with discipline and practice, it's possible to achieve.

I'd like to share some guidelines to help. But first, please, pay attention at
this:

> They are guidelines, not unbreakable rules

#### Enable Restriction Flags

It's pretty easy to commit mistakes when writing Shell Scripts. Let's talk
about two of them.

* * *

 **Abort if a command fails**

A Shell Script, at the very basic level, represents a series of commands that
are executed one after the other. Each command yields an execution status that
says if the command has succeeded, or not. By convention, the execution status
0 means success and all other status mean failure.

In general, if a command fails, we'd like to stop the script, because we're
probably at an invalid state and we shouldn't proceed.

But, by default, Shell ignores the execution status, and simply moves forward
to the next command and so on.

Fortunately, it's pretty easy to change this behaviour and fails the script if
some command has failed. To do it, we need to enable the _-e_ flag by adding
the following line before any other command that should be validated **:**

    
    
    set -e

So, as soon as the first command fails, the Shell will stop the script.

But, in some specific cases, we'd like to continue the script even if some
command has failed. For instance, an optional action may or may not be
executed, thus its status is irrelevant. For this scenario, we can use an OR
operator that will "fake" the execution status of the whole command to be
successful:

    
    
    commandThatMightFail || true 

* * *

**Abort if an undefined variable is used**

Another common mistake related to Shell Scripts is reading an undefined
variable, this usually happens by mistyping the variable name.

When that happens, the default behaviour of the Shell is to expand the
reference to an empty string.

That isn't the behaviour we normally want, or, at least, expect to have.
Normally, we'd like to stop the script as soon as we try to reference an
undefined variable.

Fortunately, it's also pretty easy to change it. To do so, we need to enable
the _-u_ flag by adding the following line before any other command that
should be validated:

    
    
    set -u

So, as soon as the first undefined variable is referred, the Shell will stop
the script.

But, once again, there are some specific cases where an undefined variable
makes sense, and so, we wouldn't like to stop script. An example is a variable
that represents an optional command line argument.

For this scenario, we can provide an alternative value to be used when the
variable hadn't been previously defined:

    
    
    ${variable:-defaultValue}

#### Always Provide an Execution Status

Still regarding the discussion about execution status. We're working on a more
and more automated environment. Where tools use other tools without human
intervention. For instance, we have pipelines, where processes are
sequentially executed towards an end result, like deploy to production.

In order to have that level of automation working properly, we need to provide
a way such that a tool can monitor the execution of the tools that were
invoked by the former. So it can decide where it should take one or other
action, like roll back the deploy to production.

The most basic way to monitor the execution of a process it by inspecting its
execution status. A large number of tools are based on the assumption that
we're going to provide an execution status, for failure and success.

It's straightforward to provide the execution status in Shell Script, we just
need to use the _exit_ command, supplying an integer called status code as its
argument:

    
    
    exit statusCode 

Of course, we need to remember to add it to each possible branch inside your
control flow, where we want to finish the script.

#### Extract the Knowledge into Functions

Whenever we see a duplication of the same piece of knowledge in our code base,
it's nearly impossible to not extract them into a module and then reuse it at
all the places. It's so natural, it's our instinct.

But somehow, it's not always the case when we're talking about Shell Scripts,
we don't put so much attention to it, and it's easy to come up with the same
step being repeated in different scripts inside the same project, or even
inside the same project. We should extract them and reuse. There are so many
(already known) benefits of such practice. For example:

  * Simple code bases
  * We only need to change one place
  * We avoid inconsistency

It's important to write concise functions that have unique concerns. Moreover,
avoid coupling by providing to the function the information that its needs by
arguments whenever makes sense to reuse the function with another set of
conditions. Hence, the function can be easily plugged. But don't accumulate
multiple concerns inside the same function.

An important thing to bear in mind when writing a function is to keep the
variables declared inside it at the minimum necessary scope. To achieve this
behaviour in Shell Scripts, we prefix the variables with _local_ , so the
variable is only visible in the block of in which it appears. Therefore, for
functions, its scope is the function itself:

    
    
    local myVariable=myValue

#### Example

To illustrate the concepts, I'll provide an example. It's a simplistic
version, but models a fairly common use-case:

> Suppose that we have a CI/CD pipeline that is executed each time we push a
change to our git repository. But, when the change is merged into the _master_
branch, we deploy to the staging environment.

By analyzing the requirements, we can detect two concerns:

  1. Check if we're at the master branch
  2. Run the deploy

Firstly, we can think about an ad-hoc script, where the commands are executed
one after the other, without any structure. It's enough for simple cases, but
it can easily make things more complex than they need.

So, let's keep it structured by having each concern wrapped in a specialized
function, with a proper name that emphasizes its purpose.

For sake of demonstration, I'm faking the deploy to be a simple echo. But it
serves as an illustration.

If you run the script in a git repository at the _master_ branch, it 'll print
deploy, if you're at any other branch, it'll print the failure message _deploy
is only allowed for the master branch_.

The most important thing here is, that we divided our logic into separate
functions that are coordinated by the main function, representing the "entry
point" of the script after its invocation at line 32. Also, note that we
invoked the function _isBranch_ as a sub-process by using _$()_ , and the
function simulates a predicate by echoing _true_ if the current branch 's name
equals to the one provided by argument. So the condition checked by the _if_
is satisfied, then we called the _deploy_ function to run our super complex
deploy  :-).

#### Conclusion

Nowadays, we have a vast number of awesome tools that help us to improve our
products and make our lives easier by automating manual tasks. But, they don't
replace Shell Scripts, which is, most of the time, the least common
denominator for Linux based applications.

Instead of continuously struggling to replace a tool by another, they should
be used together, enhancing the positive attributes of each tool for the right
scenario.

But unfortunately, sometimes we overlook our Shell Scripts, not applying the
well-known best practices that we're used to applying to our production code.
The outcome is obvious: a significant amount of scripts that take much of our
time when we need to debug or change them. Hence, the time that we'd saved by
automating, is now wasted because of careless design when writing that script
at first place.

This shouldn't be like this, we can and we should strive to write clearer
scripts and leverage its power in our favour, not against us.

I know, it's not the most user-friendly tool, but it doesn't mean that we
can't enjoy and extract the best of it. Remember: Abstraction can be our ally
here too.

#### References

[1] DevOpsCon 2018. Shell Ninja: Mastering the Art of Shell Scripting. Dr.
Roland Huß.

[2] The Linux Documentation Project. <https://www.tldp.org/guides.html>

[3] Safer bash scripts with 'set -euxo pipefail'.
[https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail](https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/)

