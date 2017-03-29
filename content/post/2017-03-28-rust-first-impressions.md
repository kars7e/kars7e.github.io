---
date: 2017-03-28T17:09:31-07:00
author: karol_stepniewski
comments: true
tags:
  - dajsiepoznac2017
  - rust
  - filesyncer
  - golang
title: "Rust: first impressions"
description: My first impressions after learning some Rust.
---

It's been a while since my last [DSP post]({{< ref "2017-03-17-DSP-project-details-for-filesyncer.md" >}}), mainly due to my trip to Poland and unavoidable result of it - jetlag. Let's get back to the business, though. **In this post, I will share my first impressions on Rust, as well as some code in filesyncer project**.

Let's dig in!
<!--more-->

## Implementing simple daemon ##
I will reverse the order a bit, and start with the code of filesyncer first. It will give some "meat" to discuss. Goals for the first iteration of filesyncer code are:

* *Application should run continuously, until it's explicitly stopped by the user or an unrecoverable error occurred.*
* *Application should be able to listen to and handle `INT` (Ctrl+c) signal*

These goals seem relatively clear. The first requirement sounds very much like a [daemon](https://en.wikipedia.org/wiki/Daemon_(computing)). In old days, one would add special logic to "daemonize" his/her application: ensure application is owned by `init` process, standard output/input/error is correctly handled, file descriptors are closed, etc. This is not needed anymore, as today's init systems (like systemd or upstart) are able to handle "normal" applications so that they can run in the background without an issue. We can also follow The [Twelve-factor app document](https://12factor.net/), specifically the "[Treat logs as event streams](https://12factor.net/logs)" recommendation, and just print our messages to standard output, leaving their proper logging/storing to dedicated systems.

We can then implement our first goal using the following piece of code:

{{< highlight rust >}}
fn main() {
    println!("Starting our application...");
    // Initialization logic
    println!("Application started");

    loop {
    	// Handle file events
    }

    println!("Shutting down...")
}

{{< / highlight >}}

We can compile and run this code:

{{< highlight bash >}}
$ rustc main.rs
warning: unreachable statement, #[warn(unreachable_code)] on by default
 --> main.rs:9:5
  |
9 |     println!("Shutting down...");
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: this error originates in a macro outside of the current crate

$ ls
LICENSE   README.md main      main.rs
$ ./main
Starting our application...
Application started
^C
{{< / highlight >}}

**Woah!** There are several things that happened here. **Let's break it down.** First of all, this code compiles and produces an executable binary (we can see "main" file in **ls** command output). However, it produces a warning: "unreachable statement". Also, after running it, we can see it's running continuously, but after sending INT signal (with the ctrl+c combination), we don't see our "Shutting down..." message! The compiler already warned us about that - the `println!` statement is unreachable because there is no logic to break out of the loop. And since we haven't implemented any `INT` signal handler, the default action is taken, which is to immediately terminate program execution. Let's fix that, and handle the `INT` signal explicitly.

## Adding signal handling ##

Before we dive into signal handling in Rust, I would like to first mention how this is being done in Go - my "native" language. Go includes signal handling in its standard library. The [`os/signal` package](https://golang.org/pkg/os/signal/) provides simple and elegant API to deal with signals through [channels](https://tour.golang.org/concurrency/2). Here's a quick example:

{{< highlight go >}}
package main

import "fmt"
import "os"
import "os/signal"
import "syscall"

func main() {

    sigs := make(chan os.Signal, 1)
    done := make(chan bool, 1)

    signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

    go func() {
        sig := <-sigs
        fmt.Println()
        fmt.Println(sig)
        done <- true
    }()

    fmt.Println("awaiting signal")
    <-done
    fmt.Println("exiting")
}
{{< / highlight >}}

This sample creates two channels, registering one of them as an endpoint for signal handling. Next, it spins out a [goroutine](https://tour.golang.org/concurrency/1) which blocks until a signal is received, and when that happens it prints it to stdout and sends a message to "done" channel, effectively exiting the application (main function blocks until a message from "done" is received).

This example shows how easy it is to handle signals in Go, and I hoped to find something similar in Rust. Turns out Rust does not have anything dedicated to signal handling in the standard library yet - there is an [RFC](https://github.com/rust-lang/rfcs/issues/1368), however with no actual proposal yet. Reading through the discussion I stumbled upon crate called [chan-signal](https://crates.io/crates/chan-signal). The docs say: *"This crate provides a simplistic interface to subscribe to operating system signals through a channel API."*. Sounds exactly like what I was looking for! **Let's give it a try**.

First, I need to obtain this crate into my project. so far I was using single `main.rs` file directly compiled using `rustc` compiler. Let's switch to the official way of doing things: [Cargo](https://crates.io/). To "add" Cargo to our project, we need to create Cargo config file. `Cargo` binary gives us an easy way to do that:

{{< highlight bash >}}
$ Cargo init --bin
     Created binary (application) project
{{< / highlight >}}

`--bin` switch was needed since our project is a binary (default mode is to create Rust library). We ended up with `Cargo.toml` file, which looks more or less like this:

{{< highlight ini >}}
[package]
name = "syncer"
version = "0.1.0"
authors = ["Karol Stepniewski <kstepniewski@vmware.com>"]

[dependencies]

[[bin]]
name = "syncer"
path = "main.rs"
{{< / highlight >}}

we have three sections in this file: package, dependencies and bin. The bin section was created because syncer's source code (`main.rs` file) lives directly in project main directory. Rust convention is to store source code in `src/` subdirectory. Let's move our code and remove the bin section:

{{< highlight bash >}}
$ mkdir src
$ move main.rs src/
{{< / highlight >}}

{{< highlight ini >}}
[package]
name = "syncer"
version = "0.1.0"
authors = ["Karol Stepniewski <kstepniewski@vmware.com>"]

[dependencies]
{{< / highlight >}}

Let's check if it works, by running `cargo build`:

{{< highlight bash >}}
$ cargo build
   Compiling syncer v0.1.0 (file:///Users/kstepniewski/projects/syncer)
warning: unreachable expression, #[warn(unreachable_code)] on by default
  --> src/main.rs:10:5
   |
10 |     println!("Shutting down...")
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: this error originates in a macro outside of the current crate

    Finished dev [unoptimized + debuginfo] target(s) in 0.81 secs
$ ls
Cargo.lock Cargo.toml LICENSE    README.md  src        target
$ ls target/debug/
build       deps        examples    incremental native      syncer      syncer.d
$ target/debug/syncer
Starting our application...
Application started
^C
{{< / highlight >}}

Looks like it does! Now, let's add the `chan-signal` dependency (it also requires `chan` crate):

{{< highlight ini "linenos=inline,hl_lines=7 8">}}
[package]
name = "syncer"
version = "0.1.0"
authors = ["Karol Stepniewski <kstepniewski@vmware.com>"]

[dependencies]
chan = "0.1.19"
chan-signal = "0.2.0"
{{< / highlight >}}

now, we build our application again:

{{< highlight bash >}}
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading chan-signal v0.2.0
 Downloading chan v0.1.19
 Downloading lazy_static v0.2.5
 Downloading libc v0.2.21
 Downloading bit-set v0.4.0
 Downloading rand v0.3.15
 Downloading bit-vec v0.4.3
   Compiling lazy_static v0.2.5
   Compiling libc v0.2.21
   Compiling bit-vec v0.4.3
   Compiling bit-set v0.4.0
   Compiling rand v0.3.15
   Compiling chan v0.1.19
   Compiling chan-signal v0.2.0
   Compiling syncer v0.1.0 (file:///Users/kstepniewski/projects/syncer)
warning: unreachable expression, #[warn(unreachable_code)] on by default
  --> src/main.rs:10:5
   |
10 |     println!("Shutting down...")
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: this error originates in a macro outside of the current crate

    Finished dev [unoptimized + debuginfo] target(s) in 5.30 secs
{{< / highlight >}}

It builds correctly! Time to use it. I've adapted the sample code from `chan-signal` documentation:

{{< highlight rust >}}
#[macro_use]
extern crate chan;
extern crate chan_signal;

use chan_signal::Signal;

fn main() {
    println!("Starting our application...");
    // Initialization logic
    let signal = chan_signal::notify(&[Signal::INT, Signal::TERM]);

    // We create a channel to be used when application wants to stop itself.
    let (sdone, rdone) = chan::sync(0);

    // Run our application logic in a separate thread.
	::std::thread::spawn(move || run(sdone));

    // Wait for a signal or for application to stop itself.
    chan_select! {
        signal.recv() -> signal => {
            println!("Shutting down... (received signal: {:?})", signal)
        },
        rdone.recv() => {
            println!("Application stopped.");
        }
    }
}

fn run(_sdone: chan::Sender<()>) {
    println!("Application started");
    loop {
    	// Application logic
    }
}
{{< / highlight >}}

**Woah (again)!** There is a lot going on here. First, we've imported two external crates: `chan` and `chan_signal`. `chan` is a multi-producer/multi-consumer channel library. Standard rust library already contains channels support, however, their implementation follows multi-producer/single-consumer semantics.

Our main function grew considerably. We define a `signal` variable via `chan_signal::notify` static method. This method returns a special channel, that we can further read from to obtain our signal (if such is sent). We also declare a synchronous `(sdone, rdone)` channel (it's used for the same purpose as its Go version). We spawn new OS thread used to run our application logic through `run()` function, passing `done` channel as a mean to stop application execution if needed. Finally, we use [`chan_select!`](http://burntsushi.net/rustdoc/chan/macro.chan_select.html) macro to "poll" our channels for messages - it will block until either of them returns a message. Let's see how that works:

{{< highlight bash >}}
$ cargo build
   Compiling syncer v0.1.0 (file:///Users/kstepniewski/projects/syncer)
    Finished dev [unoptimized + debuginfo] target(s) in 0.89 secs
$ target/debug/syncer
Starting our application...
Application started
^CShutting down... (received signal: Some(INT))
{{< / highlight >}}

Tada! We successfully handled the INT signal!

## Caveats ##

There are always some! You probably spotted differences between Go and Rust version. Firstly, Go version uses only standard library packages, while Rust uses external creates to achieve a similar effect. Secondly, while Rust version uses native OS thread to run application logic, Go version utilizies lightweight goroutines (note: There seems to be [WIP on library](https://github.com/rustcc/coroutine-rs) to support coroutines in Rust). Finally, `chan-signal` crate has no support for Windows. This is not very surprising, as signals are a POSIX thing, however Go is able to INT signal (as invoked through ctrl+c or ctrl+break) in Windows through `os/signal`. I imagine such support could be added to `chan-signal` as well if needed.

## What about those first impressions? ##

When reading through Rust documentation, I've had few "OMG This is awesome!" moments. Pattern matching, variable bindings, `Option<T>` type, all these sound familiar from languages like Haskell or Elixir, and seem like a great addition to a system programming language, which is Rust's main priority. As such, however, Rust focused on providing value in different areas, so certain things (like these I described above) are harder to achieve compared to Go. Having written that, I definitely see value in Rust. It's too early for me to give full comparison of these languages (I will write such post once I get more proficient in Rust), but the much broader control over your program that Rust gives by default is already appealing.

**What about you? Go or Rust? What do you pick and why?**


