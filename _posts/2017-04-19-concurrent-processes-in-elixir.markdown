---
layout: post
comments: true
title:  "Concurrent Processes in Elixir"
date:   2017-04-19 14:00:00 -0500
categories: elixir
---

### An Elixir Noob Stumbles into Concurrency-oriented Programming ###

Like many Rubyists today, I too am learning Elixir.  So far, it's been an
exciting endeavor.  I started out by reading [**Elixir in
Action**](https://www.manning.com/books/elixir-in-action) and was totally blown
away by the first chapter in which the author, Saša Jurić, teaches the reader
about Erlang concurrency.  Indeed, the Erlang VM (aka BEAM), is an amazing thing: it
supports it's own processes (rather than OS processes) that execute code concurrently 
accross CPU cores. Whats more, the BEAM processes are super cheap; they don't 
share memory; and, if long-running, schedulers will prevent them from 
halting the entire system by giving processes limited execution windows. 
When I finished that first chapter, I was hooked.

Since then, I've been spending time learning Elixir as basically a weekend
project.  I read a couple more chapters of [**Elixir in
Action**](https://www.manning.com/books/elixir-in-action) and familiarized
myself with the basic concepts of functional programming in Elixir.  I learned
about the power of pattern matching, recurision and how to work with immutable data.
While all of these concepts are great, what about the concurrent awesomeness 
of the BEAM?

On March 2-3, 2017, I attended [**Elixir Daze**](http://www.elixirdaze.com/)
where I heard Jesse Anderson's [**The ABCs of OTP**](https://www.youtube.com/watch?v=8X0IWW2GJoQ).
His fantastic talk pointed me in the right direction.  Per his recommendation, I picked up a copy of
[**The Little Elixir & OTP Guidebook**](https://www.manning.com/books/the-little-elixir-and-otp-guidebook)
and got to work.  Now, I'm on the good stuff: OTP.

### Concurrent Processes and Blasting Off Rockets ###

If you have just started learning Elixir but want a quick and 
tiny taste of concurrent programming, here is something you can do in quickly 
in IEx. It super high level, absurdly trivial, tip-of-the-iceberg stuff using primitives, 
but it's a just an introduction. Down below, I've provided some links to some real 
resources to learn more.

1. What's cooler than blasting off a rocket?  Blasting off two rockets at the same time. 
   Create a file called `blast_off.ex` and add the following code. 

    ```elixir
    defmodule BlastOff do
      def get_ready do
        receive do
          :countdown ->
            Enum.each(10..0, fn(i) ->
              IO.puts i
              :timer.sleep(1000)
            end)
            IO.puts "Blast Off!"
        end
      end
    end
    ```
    
    Here we define a module `BlastOff` with one function called `get_ready`. The interesting thing 
    here is the `receive/1` function.  The processes running the `receive/1` function will wait for
    a message that matches the pattern defined within the `do` block.  In our `receive/1` function
    we're expecting a message that is the symbol, `:countdown`.  Upon receipt of the `:countdown` 
    message we start a countdown from 10 to 0 printing each number and sleeping for 1 second. 

2. Now lets start up IEx session with our `BlastOff` module loaded.

   ```elixir
   iex blast_off.ex
   ``` 
   
3. Use `spawn/3` to start up a process that runs our `BlastOff.get_ready/0` function. It's going to wait for
   the `:countdown` message. We bind the result of this function to `pid1`.

   ```elixir
   iex> pid1 = spawn(BlastOff, :get_ready, [])
   #PID<0.92.0>
   ```
   
   The `spawn/3` function creates a process out of the given module, function and arguments.  In our case
   the augment list is empty since our `get_ready` function doesn't take any arguments. 
   
4. Lets spawn another process and bind the result to `pid2`.

   ```elixir
   iex> pid2 = spawn(BlastOff, :get_ready, [])
   #PID<0.94.0>
   ```
   
5. Fire up the [Erlang Observer](http://erlang.org/doc/apps/observer/observer_ug.html) ```iex> :observer.start```.
   We should see our processes.

   ![observer](https://monosnap.com/file/paqffXcUKalvKIubFXLCbfRDHVyI1W.png){:height="500px" width="700px"}
   
   Notice now little memory these processes are consuming (those numbers are in bytes).
   
6. Now, let's launch a couple rockets. Send the `:countdown` message to our processes and observe the results.
   In the example below, we're iterating over the list of pids we created earlier then 
   using `send/2` to send each pid the `:countdown` message.

   ```elixir
   iex> Enum.each([pid1, pid2], fn(pid) -> send(pid, :countdown) end)
   ```
   You should now see the results of both processes counting down at the same time.
   
This example is painfully trivial, but it serves as a very simple introduction into concurrent
processes in Elixir.  Going beyond this, means diving into GenServer and OTP: the really good stuff that makes 
Elixir truly awesome and powerful. Here are some great resources to learn more.

### Resources for Further Learning ###

* [**The Little Elixir & OTP Guidebook**](https://www.manning.com/books/the-little-elixir-and-otp-guidebook):
  Do you want learn OTP now?  This do yourself a favor and get this book.  I highly recommend it.
* [**The ABCs of OTP**](https://www.youtube.com/watch?v=8X0IWW2GJoQ): Jesse Anderson's Elixir talk.
* [Jessie Anderson's **ABCs of OTP** resource guide](https://gist.github.com/jessejanderson/16cbc0614e9194fa1b64460f775777ab): This is really cool - he lists some great resouces for learning OTP.  I found this to be an excellent guide.
* [**Elixir Daze**](http://www.elixirdaze.com/): A super cool Elixir Conference in Saint Auguestine, FL.
