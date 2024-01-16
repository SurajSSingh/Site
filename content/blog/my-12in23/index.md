+++
title = "Reflecting on #12in23 Challenge"
description = "A post reviewing my work for Exercism's 12in23 Challenge"
date = 2024-01-15
# updated = 2024-01-15
draft=true

[taxonomies]
programming-language = []

[extra]
allow_comments = true
+++

This post will go over my experience with the Exercism [#12in23 Challenge](https://exercism.org/blog/12in23-calendar) and what will happen next.
Much of this will be a retrospective of what I remember feeling for each language rather than a walkthrough of my solutions, 
so expect a more stream of consciousness style of writing.
<!-- more -->

## Prior experience with Exercism

Before I get into my thoughts through each of the months in the challenge, a bit of background of myself.
I joined towards the beginning of 2022, as I was looking for some ways to help my students practice programming in Python.
At the time, I think we would work through some problems together to get a feel for how Python works.

On my own time, I did some exercises in the C# and F# tracks.
C# mostly came from refreshing my knowledge for Unity and F# to compare it to a functional programming language.
The F# exercises transitioned into some Haskell, as one of my Master's courses used Haskell ([CS 421 - Programming Languages & Compilers](https://cs.illinois.edu/academics/courses/cs421)).

However, after that, I didn't really have much time to work on the exercises (mostly due to my coding instructor work ramping up and finishing up my Master's degree).

## 12in23 Challenge

In 2023, I had just graduated, and I was finishing up some student projects.
I had some free time and wanted to learn more about programming languages.
Initially, I tried to work on the [Comp Lang Comp](https://complangcomp.github.io) project, though I didn't have much motivation to continue due to other commitments.

Instead, when I learned about the 12in23 Challenge by Exercism, I thought it would be a good way to get exposed to a bunch of languages.
While there were only 12 themes for the year, so I couldn't deep dive into any one language, it would at least introduce a bunch of different paradigms for me to work through.

One extra challenge I set for myself was trying to do all the coding inside the Exercism web editor.
This was more so to try out each of the languages in its simplest form.
This way, it would require me to read the docs and take notes for how to code.
It did work for the docs, but I didn't actually have a note-taking system at the time, so that part fell by the wayside.

### January
I did not do much for January since I didn't realize #12in23 had started, so I didn't officially start until February. 
However, I did a finish a couple of unfinished Haskell exercises and looked into Python for mentoring.
Besides that, there wasn't much I did for Exercism until February rolled around.

### Functional February
For February, I worked on the [Elixir track](https://exercism.org/tracks/elixir) and finished 9 exercises. 
Originally, I wanted to do Haskell, but I wanted to try out a [BEAM language], since I never really worked with one before.
This lead me to Elixir and Gleam.
I eventually settled on Elixir, mostly down to wanting to check out the Phoenix framework (though I didn't actually have the time to create a project with it).

I liked learning Elixir, especially with it simple syntax and being functional without being too overbearing with types.
The concept I had the hardest time grasping was `atoms`, mostly since I had never experienced it from other languages.
The main thing I got tripped up on was thinking of `atoms` as variable identifiers, rather than predefined constant.

The biggest thing I remember for this month was the [Interview with Jose Valim](https://youtu.be/LknqlTouTKg).
Some ideas brought up were some things I had already considered working through, such as having language cheat sheets and creating objects using regex.

### Mechanical March
For the March theme, I focused on the [Rust track](https://exercism.org/tracks/rust).
I had some small experience with Rust during my graduate program, specifically with implementing algorithms for my Data Mining course work.
Beyond that, I wanted to learn more about Rust and thought working through some exercises would help with that.

For this month, I had a mix of functional programming with some procedural code, which Rust excels at.
I really loved the way I could chain functions (usually through the Iterator trait), 
but could always add small imperative elements like mutable variables to help build my solution.
The hardest problem for me was the [Simple Linked List](https://exercism.org/tracks/rust/exercises/simple-linked-list) exercise.
Mostly that was down to how I needed to conceptualize the `Box` type and the interaction between the `SimpleLinkedList` and `Node`.

Like you've probably heard from others, Rust is an awesome language to work in.
Even without help from an IDE, Rust was fairly simple to work in, at least for getting the basics down.
The compiler also provided great help when I made some simple to fix mistake, and usually guided me towards a working solution.
For the remaining of 2023, I would continue working on and off in Rust for some projects.

### Analytical April
For April, I did the [Julia track](https://exercism.org/tracks/julia).
I mostly chose this since I already had some experience in Python and R, and wanted to compare Julia to it.
For the exercises, I mostly came at it from a Python based approach, which might bias my solutions towards that.
However, with that, I found Julia to be a more compact Python.
I especially liked the way lambdas are done in Julia over Python, since it is just an arrow `->` away instead of the word `lambda`.
Also, I found the idea of Unicode identifier to be really cool to add executable math equations, although I didn't have much of a chance to use that as much.

### Mindshift May
For the month of May, I worked on the [Prolog track](https://exercism.org/tracks/prolog).
I made this choice mostly to learn about a different declarative paradigm from functional programming.
Working with Prolog was fairly strange at first, as I had to unlearn my imperative way of thinking.
For example, learning not to think in assigning variables directly, but having atoms bound to a value based on the logical relations.
Once I got my head around the concepts, things clicked into place form me.
The biggest hurdle for me was learning how to use the cut operator correctly.
However, this month was one of the most rewarding.
It was the only time in this challenge where I woke up as I was drifting off to sleep to write[^voice-memo] a solution that popped into my mind.

### Summer of Sexp (June)
For June, I focused on the [Clojure track](https://exercism.org/tracks/clojure).
I hadn't worked with a LISP dialect before and none of them particularly called to me, 
so I just chose the one that had I had heard about the most.
For the most part, nothing seem to be too out of the ordinary for me besides the prefix function/operator application.
Lisp languages seemed fairly straightforward once you get past the unfamilarity of S-Expressions.

For this month, I also worked on implementing [Make-A-Lisp](https://github.com/kanaka/mal) in [Rust](https://github.com/SurajSSingh/make_a_lisp_typed_and_readable), which is a Clojure inspired Lisp.
The synergy with this month gave me a better grasp of how Lisps work and some concepts related to language development.

### Jurassic July
For July, I worked through the [C track](https://exercism.org/tracks/c) exercises.
I only knew C from what I vaguely remember from my undergraduate introduction to C++, so I initially was hesitant with how the coding would go.
However, surprisingly, I found working in C to be really simple once you get the major concepts down.
It was also interesting to note that: 1) `bool` type is not built-in, but 2) C actually pointed out that I should include `<stdbool.h>`.
I guess I expected C to be more obtuse (though it could just be that I haven't programmed in a large C project).
Although the language itself doesn't actually provide the safeguards of modern languages, I didn't really run into issues with out-of-bound or off by 1 errors (thanks Rust for baking the borrow checker into my brain, I guess). 

This month, I also finished a bunch more Haskell exercises.

### Appy August
For August, I worked through the [Elm track](https://exercism.org/tracks/elm).
This was mostly down to trying out something I could use for web app development.
Considering what I learned from Elixir and Haskell, Elm's functional programming features felt really simple to grasp.
Just like Rust, I liked the helpful compiler guiding me towards a working solution.
While I did like working in Elm, I actually settled on Svelte for web development.
This was mostly since Svelte's HTML-focus templating felt more natural for me.
However, I will say that Elm's ease of creating type-safe models and functions was much better than TypeScript.

I also finished a few more exercises for Elixir this month.

### Slim-line September
For September, I worked on the [Raku track](https://exercism.org/tracks/raku).
I was going between Perl and Raku, and what decided it for me was Raku's parsing tools.
The hardest part for Raku was figuring out how to work with strings.
Surprisingly, the regex wasn't that bad, though that could just be how my brain works.
I would have wanted to spend more time going through the Grammar tool (and especially the named regex to help me with a programming language I am considering creating).

### Object-Oriented October
For October, I focused on [Ruby](https://exercism.org/tracks/ruby).
I had already had some experience with Java and C#, and Ruby was the only popular object-oriented language I hadn't actually used before.
Ruby itself was a fairly easy language for me to pick up and nothing felt too out of the ordinary.
Two things that stood out for me were the native use of regex (which made string search really easy) and the concept of blocks (which felt very much like [Rust's closures](https://doc.rust-lang.org/reference/influences.html)).

### Nibbly November
For November, I worked on the [WASM track](https://exercism.org/tracks/wasm).
The main reason I chose this was to get an idea for working with WebAssembly from other languages like Rust or TypeScript (via [AssemblyScript](https://www.assemblyscript.org)).
I felt it much like C, simple once you got the major concepts down.
The big thing I struggled with was trying to visualize thinking about strings.
Since WebAssembly itself only has the concept of numbers, strings are just a series of bytes stored in memory.
The hard part was figuring out the way memory works in WebAssemby.
The rest would then be handled by a memory "pointer" (memory offset) and string length, just like C.

Something I did for this month was work on a [visual debugger](https://github.com/SurajSSingh/WASVD) to help understand the stack-based processing.
This gave a better idea about how WebAssembly is implemented.
Also, this was the first time I posted in the Exercism Discord, 
where I listed some helpful resources for working directly in Wasm.
Overall, it felt nice learning a lower level language and appreciating the affordances provided by higher level ones.

### December
For December, I worked through the [Wren track](https://exercism.org/tracks/wren).
The big reason I wanted to try Wren was to compare it to Lua.
For the most part, Wren felt very much like a traditional object-oriented programming language.
Nothing stood out too much, though that could be because I approach much of the problems from an imperative view.
Looking at some other solutions, I saw that they were using block like Ruby, and had a more functional style,
which I should have tried more of.

### What I learned from this challenge
There can be a lot to say about all that I learned throughout this challenge, here are my major takeaways:
1. The challenge gave me a focused point to learn a specific language in a good amount of time (enough to understand where to go next).
2. I learned and practiced a specific paradigm by language.
3. Gave me some inspiration for projects for learning languages (Comp-Lang-Comp and WASVD). 

Here are some things I will try differently moving forward:
1. Trying out other kinds of solutions in the same language (Exercism saves different iterations)
2. Picking more obscure programming languages to help expand my thinking for programming languages
3. Build more projects that support the language I am working in

## What's next

Here's what's next for me after the challenge.

### Comp Lang Comp
In late 2022, I started working on a website to compare a bunch of programming/computer languages syntaxes against each other.
The key think I wanted to do for this was provide links to resources for learning about the syntaxes from primary sources.
Originally, I had planned to work on this during the challenge, but time slipped away from me, and I didn't really have time to contribute.

However, with what I learned and some resources I have found, I still plan to continue working on this.
The main thing I want to do before opening it up to public contributions is updating the website to make it easier to work with.
Currently, it is a Slate website, which I mostly chose because it had the side by side code view.
The only issue was that I didn't really have a good grasp of Ruby to make changes on my own. 

### #48in24 challenge
[#48in24](https://exercism.org/challenges/48in24) is the successor to the #12in23 challenge.
Every Tuesday starting on January 16, 2024, a new exercise will be highlighted.
Don't know the full plan, but I will be doing some blog post for (some of) these exercises.

## Footnotes
[^voice-memo]: By "write", I mean make a voice memo, I didn't want to ruin my sleep by getting my laptop and coding something in the middle of the night.