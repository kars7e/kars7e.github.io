---
author: karol_stepniewski
comments: true
date: 2016-03-04T14:05:45Z
image:
  feature: elixir-screen2.png
tags:
- dajsiepoznac2016
- programming
- elixir
- erlang
- functionalprogramming
title: My introduction to Elixir programming language
url: /2016/03/04/dsp-my-introduction-to-elixir/
---

Today I'm going to talk a bit about Elixir, the programming language that I chose to write my [project]({{< ref "2016-02-12-daj-sie-poznac-challange-accepted.md" >}}) in. I will start by saying that this is not a comprehensive introduction, but just my take on what are the Elixir's most interesting features, and why I think this language has a bright future ahead of it. If you would like to learn more about Elixir, there are really good resources out there in the Internet, notably:

- [A getting started guide](http://elixir-lang.org/getting-started/introduction.html) - Official guide on Elixir website, one of the best I've seen.
- [Programming Elixir 1.2](https://pragprog.com/book/elixir12/programming-elixir-1-2) - A book by the one and only [Dave Thomas](https://twitter.com/pragdave). I've read it and I wholeheartedly recommend it.
- [Portal Game in Elixir](https://howistart.org/posts/elixir/1) - by [Jose Valim](https://twitter.com/josevalim), the author of Elixir. I remember reading this article was the first thing that got me into the language.

## Erlang - the battle-tested foundation of Elixir
I will start my introduction by talking about a different language, called [Erlang](https://en.wikipedia.org/wiki/Erlang_(programming_language)). Erlang is a 30 years old language built specifically to be used in distributed and highly available applications. It has proved its value over the years in telecommunications industry where it had been employed to deal with telephone connections at massive scale, but it also has been used by companies like [Whatsapp](https://vimeo.com/44312354) or [Facebook](https://www.quora.com/Why-was-Erlang-chosen-for-use-in-Facebook-chat) for their most connections-heavy applications. Such strong foundations are very important for me - as a pragmatic programmer, I'm looking for solutions that will last, and Erlang ecosystem is definitely a good ingredient to build one.

So how does Elixir utilize Erlang? First of all, it's written in it. You can look at the [Elixir's Github repository](https://github.com/elixir-lang/elixir), You will obviously notice that Elixir is a huge Erlang's code base. But there's more than that. When you write code in Elixir, you can utilize [standard Erlang library](http://erlang.org/doc/apps/stdlib/). You will find a lot of useful 3rd party libraries written for [Elixir or Erlang](https://hex.pm/) itself. In my opinion this helps a lot in Elixir's adoption, because lack of decent library ecosystem is often a main problem of young programming languages. And last but not least, Elixir utilizes the power of BEAM, an Erlang's virtual machine. It's very interesting concept itself, and I won't go much into detail of it here, let me just mention that Erlang VM in some way behaves very similarly to the Operating System - in fact, there is a special version of it running [directly on Xen](http://erlangonxen.org/) - without any additional operating system in between.

## Elixir - let's go meta
Enough about Erlang, let's talk about our actor in a leading role, Elixir, and what have made me fall in love with that language. There are many reasons to that, but I will start with a small piece of code that comes from the book ["Programming Elixir"](https://pragprog.com/book/elixir12/programming-elixir-1-2) by Dave Thomas:

{{< highlight elixir >}}
  def decode_response({:ok, body}), do: body
  def decode_response({:error, error}) do
    {_, message} = List.keyfind(error, "message", 0)
    IO.puts "Error fetching from Github: #{message}"
    System.halt(2)
  end

  def process({user, project, count}) do
    Issues.GithubIssues.fetch(user,project)
    |> decode_response
    |> convert_to_list_of_maps
    |> sort_into_ascending_order
    |> Enum.take(count)
    |> print_table_for_columns(["number", "created_at", "title"])
  end

  def printable(str) when is_binary(str), do: str
  def printable(str), do: to_string(str)
{{< / highlight >}}

This excerpt is not complete (there are few functions missing, as well as module definition, which means the example will not compile), but it perfectly depicts 2 out of my 3 favorite features of Elixir: pattern matching and pipe operator. let's talk about them in detail.

### Pattern matching - the beautiful "if"
I will start with the fact that Elixir is a [functional](https://en.wikipedia.org/wiki/Functional_programming) programming language - the central concepts of it are functions and data immutability. This is important especially if you come from the object-oriented background like Java or C++. In Java your main friends are a class and an object instance of that class, and you have a lot of these. In Elixir, you have a lot of functions, and you want them to be clean and easily understandable. Your main enemy in such situations is a conditional statement, whether it is "if" or "switch", it makes code harder to reason about - especially if you have nested several conditionals together (that's something you always try to avoid).

In Elixir, you have a great help in form of pattern matching:
{{< highlight elixir >}}
  def decode_response({:ok, body}), do: body
  def decode_response({:error, error}) do
    {_, message} = List.keyfind(error, "message", 0)
    IO.puts "Error fetching from Github: #{message}"
    System.halt(2)
  end
{{< / highlight >}}
As you can see in the example above, function **_decode_response_** is defined twice, with different function bodies.
They both accept single parameter, which is a [tuple](http://elixir-lang.org/getting-started/basic-types.html#tuples), however it goes even further - they accept only two-element tuple, and specify that first element must be either an **_:ok_** or **_:error_** [atom](http://elixir-lang.org/getting-started/basic-types.html#atoms). If we try to break any of those constraints and for example execute that function with first element set to **_:warning_**, we will get:

{{< highlight elixir >}}
iex(3)> ExampleModule.decode_response({:warning, "body"})
** (FunctionClauseError) no function clause matching in ExampleModule.decode_response/1
    iex:2: ExampleModule.decode_response({:warning, "body"})
{{< / highlight >}}

This gives a tremendous value to limit the "ifs" in our code, but also makes code much easier to reason about - the error handling case like the code above is a great example where we see exactly which part is responsible for dealing with errors, and which processes the happy code path.

There is also a mechanism called **[guard clauses](http://elixir-lang.org/getting-started/case-cond-and-if.html#expressions-in-guard-clauses)** which I consider to be a logical part of pattern matching (although technically it might not be). In the code example I already used before we can observe:
{{< highlight elixir >}}
  def printable(str) when is_binary(str), do: str
  def printable(str), do: to_string(str)
{{< / highlight >}}
Which boils down to a check if an argument is a binary (which indicates [it's a string](http://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html)) and should just be returned, or is it something else and should be first converted to string. The way it's done though is by using **_when_** keyword, which gives more visibility into what's the actual purpose of this code. **_when_** keyword causes function to match only if the expression after it is true. For example if we had a function that takes 2 arguments, and multiplies them if the first argument is larger than the second one, and sums them otherwise, we could write:
{{< highlight elixir >}}
  def combine(a, b) when a > b, do: a * b
  def combine(a, b), do: a + b
{{< / highlight >}}
It's very important to remember that functions are matched in the same order they are defined, so match-all functions should be defined last.

### Pipe operator - from unix with |love>
The second feature that attracted me to Elixir is **[pipe operator](http://elixir-lang.org/getting-started/enumerables-and-streams.html#the-pipe-operator)**. Pipe operator is not something unique to Elixir - in one of [The Changelog's episodes](https://changelog.com/194/), Jose Valim (author of Elixir) mentions that he borrowed that idea from F#, functional programming language that belongs to .NET family. Before we talk more about this operator though, let's first talk about another Elixir's property, which is data immutability. Its basic principle is that data, once created, cannot be modified in place, which in turn means that if we want store that modified data, we will create a new data entry for it. It's very similar to Unix _pipe_ ("|") use case - when we use **_cat "file.txt" | grep "some text"_**  in linux, we almost never want to save it back to _file.txt_ - instead we'll save it to a new file, or use it instantly. In that case we can talk about **data transformation** - and Elixir is all about transforming the data. So similarly to Unix, Elixir has its pipe operator, which differs from the standard "|" character by having an additional "greater than" sign after it.


Let's see the example again:
{{< highlight elixir >}}
  def process({user, project, count}) do
    Issues.GithubIssues.fetch(user,project)
    |> decode_response
    |> convert_to_list_of_maps
    |> sort_into_ascending_order
    |> Enum.take(count)
    |> print_table_for_columns(["number", "created_at", "title"])
  end
{{< / highlight >}}
So how does this work?<br>
**_&#124;>_** operator causes the output of **_Issues.GithubIssues.fetch(user,project)_** function call to be bound with the first argument of function **_decode_response_**, which output of is in turn bound to the first argument of function **_convert_to_list_of_maps_**, and so on. Notice that some of the functions in that chain still take arguments - those arguments will be passed to the function starting from the second argument, as first is reserved for pipe operator.

Such code composition gives a great insight into what data transformation is happening here, and in what order. Also, if we need to modify/remove/add one of the transformation's steps, it's very clear how it should be done.

### Meta programming in Elixir - the scary powerful wizard in a high tower
I mentioned that pattern matching and pipe operator are my 2 of 3 favorite Elixir features, and the third one couldn't be anything else then macros and meta programming. It's a big addition compared to the Erlang, and I remember reading an article where Jose mentions Meta programming and [Protocols](http://elixir-lang.org/getting-started/protocols.html) as the most important language features. I used the word "scary" because meta programming is indeed super powerful, but at the same time it might be dangerous - once you learn how many things can be done with help of macros, you might be tempted to use them even where it doesn't really make much sense. Macros are expressions in code that are executed at compile time, and replaced with computed code. I won't go into detail about macros and meta programming, because this topic requires really good explanation, so I will recommend two sources to learn more about it:

- [Great presentation](https://vimeo.com/131643017) by Chris Mccord, where he talks about meta programming in Elixir at NDC conference in Norway,
- [Metaprogramming Elixir](https://pragprog.com/book/cmelixir/metaprogramming-elixir) - an awesome book, also by Chris Mccord, which goes into detail of macros wizardry.

But to give some perspective into how powerful macros in Elixir are, consider following piece of code (shamelessly taken from Chris' presentation):
{{< highlight elixir >}}
defmodule Notifier do

  def ping(pid) do
    if Process.alive?(pid) do
      Logger.debug "Sending ping!"
      send pid, :ping
    end
  end

end
{{< / highlight >}}
So what's so special about it? Every line in this code except _send_ function is a **macro** - including **_defmodule_** and **_def_** keywords! Even **_Logger.debug_** is a macro, and there is a good reason to that. Because macros are evaluated at compile time, debug messages can be completely removed from the actual compiled code if logging level has been set to omit them, which results in saving CPU cycles required to handle them. Isn't that just awesome?

## Summary - just the tip of the iceberg
This introduction was only a brief summary of features that attracted me to the language. There are so, so many of them they may attract you to Elixir, and I encourage you to [read about it](http://elixir-lang.org/getting-started/introduction.html) and try it yourself! As of me, you can expect more posts about Elixir and its ecosystem, including Protocols, Streams, Processes, OTP, and others, as my work on the project progresses.

In next post, I will talk more about my project (game), and how can we model it Elixir language. Stay tuned!




