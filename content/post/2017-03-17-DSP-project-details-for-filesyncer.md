---
date: 2017-03-17T01:31:17-07:00
author: karol_stepniewski
comments: true
tags:
  - dajsiepoznac2017
  - rust
  - filesyncer
title: Project details for filesyncer
description: Details of my filesyncer project I'm working on as a part of DSP contest.
---

I've became a huge fan of describing ideas using single sentence. If you can't describe an idea this way, it may be that this idea needs more thought or it's too complex and could be divided into smaller components (the latter is rare). Let me describe my project for this year's edition of [DSP contest]({{< ref "2017-03-12-daj-sie-poznac-attempt-2.md" >}}) using single sentence then:
<!--more-->

*An application that continously runs on a computer, **monitors for changes in selected path** and **communicates these changes to other instances** of the same application running on different computers connected together in **single L2 network**.

There, I did it. I hope it conveys something meaningful to you. Of course, this is still very vague and, depending on the person who reads it, could be visualised as simple rsync running in daemon mode as well as very sophisticated distributed file system. Let me further clarify by providing some simple use cases.

## Feature requirements ##
A short list of features I would like to have, ordered by increasing complexity:

* Once executed, application runs continously, until user stops it using ctrl-C combination (or it receives TERM signal some other way).
* Application should take an argument which is a path to a directory on local filesystem.
* Application should monitor the specified directory by registering any changes to any files/subdirectories in that directory. Specifically, it should register new file or directory being created, existing file/directory being deleted, existing file or directory being renamed (why rename is a separate requirement I will write on a different occasion), and existing file's content being modified.
* Application should be able to detect other instances of itself running in the same network by using network broadcasting
* Application should share the changes it detects with other instances of itself running in the same network.
* Application should listen for changes coming from other instances of itself
* Application should apply changes coming from other instances of itself to it's local state.

I will stop here. It doesn't seem like a lot of requirements, especially given that I won't spend too much time veryfing if these features have been implemented correctly. **The main goal here is to deliver MVP**, and the challenge is using language knew to me to achieve that MVP.

## One more thing - rust installation ##

I don't want to produce another introductory post again, so let's document some work. This will be trivial, but... you have to start somewhere. I'm using [Rust programming language](https://www.rust-lang.org), which means I should have installed on my system. Let's do it then. I'm using MacOS, and de facto a standard [package manager](https://brew.sh/) for MacOS. Rust installation couldn't be simpler:

{{< highlight bash >}}
$ brew install rust
{{< / highlight >}}

Which after a minute or so gives us `rustc` command:

{{< highlight bash >}}
$ rustc --help
Usage: rustc [OPTIONS] INPUT

Options:
    -h --help           Display this message
    --cfg SPEC          Configure the compilation environment
    .
    .
    .
    # Omitted
{{< / highlight >}}

Let's finish this post with simple, classic and mandatory "Hello world":

{{< highlight rust >}}
fn main() {
    println!("Hello, world!");
}
{{< / highlight >}}

that we compile with `rustc`:

{{< highlight bash >}}
$ rustc main.rs
$ ./main
Hello, world!
{{< / highlight >}}

That's it! Environment ready. I will follow up in the next post, where I will attempt to write a daemon that handles signals gracefully.

> **Note**: I'm learning Rust as I write these posts. Hence, there will be a lot of modifications and sudden changes of course in my code.