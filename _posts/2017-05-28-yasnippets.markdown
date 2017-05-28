---
layout: post
comments: true
title:  "YASnippet Template System for Emacs"
date:   2017-05-28 00:00:00 -0500
categories: emacs spacemacs
---

## Crank Out Boiler Plate Code Like a Boss with YASnippet
![YASnippet Demo](/assets/images/yasnippet_angular_component_demo.gif)

Let me explain the above demo.

I sometimes work in Angular which happens to have a moderate amount
of boiler plate code.  For example, creating a new component means 
importing the `Component` decorator, passing in a config object with 
metadata to the component decorator and then exporting the class so 
it can be imported elsewhere.

In this example, I just type `comp` followed by `M-/` to expand 
out the component (I'm using Spacemacs. In Emacs, use `TAB` to expand). 
The cursor is automically dropped at the position where I need to name 
the component's selector (line 4). As I type the selector name, it is 
copied and appended `styleUrls` file name (line 5), as well as camel cased 
on the export line (line 8). When done with the selector name, I hit `TAB` 
which places me on line 6 where I can add the template.  Now I am ready 
to build my component. Let's see how this works.

Since I'm using Spacemacs my examples use `SPC SPC`, which for emacs is `M-x`.
## Creating a Snippet
# Where are snippets stored?
First a quick note on where snippets are stored.  When you save a YASnippet,
it's going to be placed in `~/.emacs.d/private/snippets/` in a sub directory
that corresponds to the mode the snippet is for. For exmaple, the Typescript 
snippet in the demo above is in `.emacs.d/private/snippets/typescript/`.

Just remember that the stuff you put in the `.emacs.d/private` is git ignored.
I like to manage this folder the way I do with my dotfiles so they get are added 
to version control.
# Create the template
I like to start in the file type of the mode for which I am building the snippet (in this case a `.ts` file).
That way the snippet will automically be placed in the right mode directory. For me,
this happens naturally because I usually get inspired to make a snippet while coding.

`SPC SPC yas-new-snippet` creates a new empty snippet that looks like this:
{% highlight lisp linenos %}
# -*- mode: snippet -*-
# name: Angular component
# key: comp
# --
{% endhighlight %}
- `name` is the description of the snippet - usefull when listing snippets with `SPC i s`.
- `key` is the string you want to type that will be expanded with `M-/`
- everthing below the metadata block (line 4) is where you write the snippet.

Here is a an simple example snippet that justs adds a Angular component import line:
{% highlight lisp linenos %}
# -*- mode: snippet -*-
# name: Angular component
# key: comp
# --
import { Component } from '@angular/core';
{% endhighlight %}
When your finished with the snippet, just `SPC f s` to save it in the `.emacs.d/private/snippets/` 
directory. Recall from [above](#where-are-snippets-stored) that it should go in a 
subfolder that is the name of mode. 
# Using Tabstops
But why stop there? Let's scaffold an entire Angular component and drop the cursor at 
locations where we need to make edits. Use `$` followed by a number.  After expanding the 
snippet, the cursor is placed at `$1`.  Pressing `TAB` takes the cursor to `$2`.  `TAB` 
again and go to `$3`, etc. `$0` is the last stop, which also exits the snippet mode.

Exapanding on the previous example:
{% highlight lisp linenos %}
# -*- mode: snippet -*-
# name: Angular component
# key: comp
# --
import { Component } from '@angular/core';

@Component({
  selector: 'app-$2',
  styleUrls: ['$3.component.scss'],
  $0
})

export class $1Component {}
{% endhighlight %}
After we expand the snippet, in order as we `TAB` through:
1. Start by naming the exported class (line 13)
2. name the selector (line 8)
3. finish the stylesheet file name (line 9)
4. exit inside the config object where we're likely to start the componet's `template` (line 10).

# Mirrors and text transformations
If we follow certain naming convetions, we might end up typing the same thing several times in the above
example.  We can use mirrors and text transformations to save us the hassle.  A field like `$1` can be repeated 
in the template where what we type at that tabstop will also be inserted at another locations.
We can also use Elisp to perform text transformations on the mirrors - like upcasing, capitalizing, etc. 

Here is the snippet that I used to make the Angular component in the 
[animated gif](#crank-out-boiler-plate-code-like-a-boss-with-yasnippet) at the start of this post.
{% highlight lisp linenos %}
# -*- mode: snippet -*-
# name: Angular component
# key: comp
# --
import { Component } from '@angular/core';

@Component({
  selector: 'app-$1',
  styleUrls: ['$1.component.scss'],
  $0
})

export class ${1:$(replace-regexp-in-string "-" "" (capitalize yas-text))}Component {}
{% endhighlight %}
Now all we need to do is type the selector name at the first `$1` and `TAB` out to the exit point.
The following now happens automatically:
- the name is copied to the styleUrls
- the name is copied to the class name. the class name is using a tabstop with a 
  placeholder the syntax of which looks like this: `%{N:placeholder_value}`
- The class name, represented by `yas-text` is transformed from kebab case to camel 
  case using a bit of Elisp.

# Conclusion
YASnippet is great to cranking out boiler plates or code that is strucurally predictive by virture 
of following convetions. It's highly customizable with tabstops, mirros and transformations.

The is by no means at all exhaustive.  There's a lot more you can do with YASnippet. Check out the
resouces below to learn more.

# Resources

Lean more with these awesome resources!

- [Youtube YASnippet tutorial](https://www.youtube.com/watch?v=-4O-ZYjQxks)
- [YASnippet Documentation](https://joaotavora.github.io/yasnippet/) (Very helpful)
- [YASnippet Github repo](https://github.com/joaotavora/yasnippet)
- [YASnippet Emacs Wiki](https://www.emacswiki.org/emacs/Yasnippet)
- [Yasnippet Spacemacs](https://github.com/syl20bnr/spacemacs/tree/master/layers/%2Bcompletion/auto-completion#yasnippet)
- [jr0cket's awesome YASnippet tutorial](http://jr0cket.co.uk/2016/07/spacemacs-adding-your-own-yasnippets.html)
