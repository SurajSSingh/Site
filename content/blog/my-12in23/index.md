+++
title = "Reflecting on #12in23 Challenge"
description = "A post reviewing my work for Exercism's 12-in-23 Challenge"
date = 2024-01-15
# updated = 2024-01-15
draft=true

[extra]
allow_comments = true
+++

This post will go over my experience with the Exercism #12-in-23 Challenge and what will happen next.
Much of this will be a retrospective of what I remember feeling for each language rather than a walkthrough of my solutions, 
so expect a more stream of consciousness style of writing.
<!-- more -->

## Prior experience with Exercism

Before I get into my thoughts through each of the months in the challenge, a bit of background of myself.
I joined towards the beginning of 2022, as I was look for some ways to help my students practice programming in Python.
At the time, I think we would work through some problems together to get a feel for how Python works.

On my own time, I did some exercises in the C# and F# tracks.
C# mostly came from refreshing my knowledge for Unity and F# to compare it to a functional programming language.
The F# exercises transitioned into some Haskell, as one of my Master's course used Haskell ([CS 421 - Programming Languages & Compilers](https://cs.illinois.edu/academics/courses/cs421)).

However, after that, I didn't really have much time to work on the exercise (mostly due to my coding instructor work ramping up and finishing up my Master's degree).

## 12-in-23 Challenge

In 2023, I had just graduated and I was finishing up some student projects.
I had some free time and wanted to learn more about programming languages.
Initially, I tried to work on the [Comp Lang Comp] project, though I didn't have much motivation to continue due to other commitments.

Instead, when I learned about the 12-in-23 Challenge by Exercism, I thought it would be a good way to get exposed to a bunch of languages.
While there were only 12 themes for the year, so I couldn't deep dive into any one language, it would at least introduce a bunch of different paradigms for me to work through.

One extra challenge I set for myself was trying to do all the coding inside the Exercism web editor.
This was more so to try out each of the languages in its simplest form.
This way, it would require me to read the docs and take notes for how to type.
It did work for the docs, but I didn't actually have a note-taking system at the time, so that part failed.

### January
I did not do much for January since I didn't realize #12-in-23 had started, so I didn't official start until February. 
However, I did a finish a couple of unfinished Haskell exercises and looked into Python for mentoring.
Besides that, there wasn't much I did for Exercism until February rolled around.

### Functional February
For February, I worked on the [Elixir track](https://exercism.org/tracks/elixir) and finished 9 exercises. 
Originally, I wanted to do Haskell, but I wanted to try out a [BEAM language], since I never really worked with one before.
This lead me to Elixir and Gleam.
I eventually settled on Elixir, mostly down to wanting to check out the Phoenix framework (though I didn't actually have the time to create a project with it).

I like learning Elixir, especially with it simple syntax and being functional without being too overbearing with types.
The concept I had the hardest time grasping was `atoms`, mostly since I had never experienced it from other languages.
The main thing I got tripped up on was thinking of `atoms` as variable identifier, rather than predefined constant.

The biggest thing I remember for this month was the [Interview with Jose Valim](https://youtu.be/LknqlTouTKg).
Some ideas brought up were some things I had already considered working through, such as having language cheat sheets and creating objects using regex.

### Mechanical March
For the March theme, I focused on Rust track.
I had some small experience with Rust during my graduate program, specifically with implementing algorithms for my Data Mining course work.
Beyond that, I wanted to learn more about Rust and thought working through some exercises would help with that.

For this month, I had a mix of functional programming with some procedural code, which Rust excels at.
I really loved the way I could chain functions (usually through the Iterator trait), 
but could always add small imperative elements like mutable variable to help build my solution.
The hardest problem for me was the [Simple Linked List](https://exercism.org/tracks/rust/exercises/simple-linked-list) exercise.
Mostly that was done to how I needed to conceptualize the `Box` type and the interaction between the `SimpleLinkedList` and `Node`.

Like you've probably heard from others, Rust is an awesome language to work in.
Even without help from an IDE, Rust was fairly simple to work in, at least for getting the basics down.
The compiler also provided great help when I made some simple to fix mistake, and usually guided me towards a working solution.
For the remaining of 2023, I would continue working on and off in Rust for some projects.

### Analytical April
For April, I did the Julia track.
I mostly chose this since I already had some experience in Python and R, and wanted to compare Julia to it.
For the exercises, I mostly came at it from a Python based approach, which might bais my solutions towards that.
However, with that, I found Julia to be a more compact Python.
I especially liked the way lambdas are done in Julia over Python, since it is just an arrow `->` away instead of the word `lambda`.
Also, I found the idea of Unicode identifier to be really cool to add executable math equations, although I didn't use that as much.

### Mindshift May
For the month of May, I worked on the Prolog track.
I made this choice mostly to learn about a different declarative paradigm from functional programming.
Working with Prolog was fairly strange at first, as I had to unlearn my imperative way of thinking.
For example, learning not to think in assigning variables directly, but having atoms bound to a value based on the logical relations.
Once I got my head around the concepts, things clicked into place form me.
The biggest hurdle for me was learning how to use the cut operator correctly.
However, this month was one of the most rewarding.
It was the only time in this challenge where I physically woke up to write a solution that popped into my mind.

### Summer of Sexp (June)
For June, I focused on the Clojure track.
I hadn't worked with a LISP dialect before and none of them particularly called me, 
so I just chose the one that had I had heard about the most.
For the most part, nothing seem to be too out of the ordinary for me besides the prefix function/operator application.

For this month, I also worked on implementing [Make-A-Lisp] in Rust, which is Clojure inspired Lisp.
The synergy with this month gave me a better grasp of how Lisps work and some concepts related to language development.

### Jurassic July
For July, I worked through the C track exercises.
I only knew C from what I remember from the introduction to C++, so I initially was hesitant with how the coding would go.
However, surprisingly, I found working in C to be really simple once you get the major concepts down.
It was also interesting to note that: 1) `bool` type is not built-in, but 2) C actually pointed out that I should include `<stdbool.h>`.
I guess I expected C to be more obtuse (though it could just be that I haven't programmed in a large C project).
Although the language itself doesn't actually provide the safeguards of modern languages, I didn't really run into issues with out-of-bound or off by 1 errors (thanks Rust for baking the borrow checker into my brain, I guess). 

This month, I also finished a bunch more Haskell exercises.

### App August
For August, I worked through the Elm track.
This was mostly down to trying out something I could use for web app development.
Considering what I learned from Elixir and Haskell, Elm's functional programming features felt really simple to grasp.
Just like Rust, I liked the helpful compiler guiding towards a working solution.
While I did like working in Elm, I actually settled on Svelte for web development.

I also finished a few more exercises for Elixir this month.

### Slim-line September
For September, I worked on the Raku track.
I was going between Perl and Raku, and what decided it for me was Raku's parsing tools.
The hardest part for Raku was figuring out how to work with strings.
Surprisingly, the regex wasn't that bad, though that could just be how my brain works.
I would have wanted to spend more time going through the Grammar tool (and especially the named regex for use in a language I am considering).

### Object-Oriented October
For October, I focused on Ruby.
I had already had some experience with Java and C#.

### Nibby November
For November, I worked on the WASM track.
The main reason I chose this was to get an idea for working with Wasm from other languages like Rust or TypeScript.

I felt it much like C, simple once you got the major concepts down.

Also, this was the first time I posted in the Exercism Discord.


### December
For December, I worked through the Wren track.
The big reason I wanted to try Wren was to compare it to Lua.


### What I learned from this challenge
Gave a good focus for what to do.

2023 is the year of practicing Rust.

## What's next

Here's what's next for me after the challenge.

### Comp Lang Comp
In late 2022, I started work on a website to compare a bunch of programming/computer languages syntaxes against each other.
The key think I wanted to do for this was provide links to resources for learning about the syntaxes from primary sources.
Originally, I had planned to work on this during the challenge, but time slipped away from me, and I didn't really have time to contribute.

However, with what I learned and some resources I have found, I still plan to continue working on this.
The main thing I want to do before opening it up to public contributions is updating the website to make it easier to work with.
Currently, it is a Slate website, which I mostly chose because it had the side by side code view.
The only issue was that I didn't really have a good grasp of Ruby to make changes on my own. 

### #48-in-24

Every Tuesday starting on January 16, 2024, new exercise will release.
Don't know the full plan, but I will be doing some blog post for (some of) these exercises.
