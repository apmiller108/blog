---
layout: post
comments: true
title: "Include Arbitrary Files in an Elixir Project"
date: 2018-05-26 09:10:00 -0400
categories: elixir
---

# Include Arbitrary Files in Elixir Project

There might be occasions in which we would want to include arbitrary files in our
Elixir projects.  Maybe we need to include executable.  Or perhaps we're
building some kind of renderer and need some assets.

Such files should go in the optional`priv/` directory in the project root.
Assets placed here will be available during the application runtime.
Executables are supposed to go in `priv/bin`.  See
the [Erlang application docs](http://erlang.org/doc/design_principles/applications.html#id81839)
for more details.

### Accessing Files in `priv/` at Runtime
Let's say you have `priv/my_file.txt` and need to access it during the
application's runtime.  We can do so with the following code:

```elixir
Path.join(:code.priv_dir(:my_app), "my_file.txt")
```

### In escripts
Escripts do not support loading artifacts in the `priv/` directory at runtime.
To work around this issue, we can store the file or file contents in a module
attribute at compile time, as Jose Valim explains in
[this ElixirForm post](https://elixirforum.com/t/is-it-possible-to-include-resource-files-when-packaging-my-project-using-mix-escript/730).
Here's an example:

```elixir
defmodule SomeModule do
  @my_file_contents File.read! "priv/my_file.txt"

  def my_file_contents, do: @my_file_contents
end

```

Now, when we compile our project we'll have the contents of `my_file.txt`
stored in the module attribute `@my_file_contents` which is accessible in escripts.

### Resources
- [Erlang Application Docs](http://erlang.org/doc/design_principles/applications.html#id81839)
