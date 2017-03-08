# Schemers - Exercise 1
<div class="subtitle">Published November 9th, 2016</div>

This post will be covering the exercises given in the first article.
I'll be going over the answers and providing more idiomatic ways to do
them. We'll evolve the answers to the one in the repo and cover some new
concepts along the way.

## What we'll be covering
- The `trim` function from the `String` standard library module
- Rust `use` statements
- More `match` patterns
- Function Parameters
- Touching on `&str`
- Touching on `enum`

## Exercises
Here's what I asked you to do in the prior post:

1. Modify the `Err` line so that the program exits gracefully on an
   `EOF` or `Interrupted`, but prints an error out like before otherwise.
   (Hint: You'll need to modify `done` here and check for the error
   somehow)
2. What happens when you modify `>> ` to be something else? If you
   understand what is happening when you change it, modify it to be
   something you like!
3. What happens when I put in something like "      (exit)      " to the
   interpreter? What method in the [standard
   library](https://doc.rust-lang.org/std/string/struct.String.html) would get rid of
   the whitespace? Find the method in the linked documentation and use it
   in the interpreter so that "    (exit)", "(exit)    ", and "(exit)"
   all cause the program to exit.

We'll start with item two, then we'll do three, and finish off with one
the most complicated of the three.

## What's the deal with ">> "?
If you played around with the interpreter you might have noticed that
`>> ` was printed out at the beginning of each line. If you changed it
to something like `|-->` then `|-->` would have been printed out on each
line. What we did there was pass a parameter to a function. In Rust
a function can take zero or more parameters as input. In this case
`readline` was expecting a `&str` as input. This input would be printed
out on each line of the prompt. If a function has more than one
parameter we separate each input with a comma. It would look something
like this when calling a function.

```rust
let x = foo(bar, baz);
```

It might be confusing that you passed in what looks like a `String`
using `"` and now I'm saying that it's type is not `String` but `&str`
For now you can think of `&str` as an immutable version of `String`.
It can't be grown or mutated in any way. What you need to know is that `str` is a
primitive type in Rust like `bool` or `i32` while `String` is not and
exists in the Standard Library. We'll be diving more in depth on this
topic in the beginning of the parser but I wanted to touch on it
briefly so that you can get a feel for the differences between `str`
and `String` which can be confusing for newcomers.

Alright let's figure out how to get number three working.

## Trim the fat
You might have noticed that typing in " (exit)" or "(exit) " didn't work
at all. In these cases we've added whitespace to the beginning and end
of this input. When we check for `String` equality though we're checking
that both strings are *exactly* the same. The problem is now that one
string has more characters than the other because of the whitespace. The
interpreter will just print it out now! How do we fix it?

Well I linked you to the documentation on all the methods you can use
for `Strings` in Rust. The Rust documentation is great and we'll
actually generate our own in the next tutorial! I asked you to find
a method that would remove the whitespace from the beginning and end of
the line. If you went through it all you would have found a method
`trim` that could be used. If you got rid of the whitespace in the
string then " (exit)", "(exit) " and " (exit) " would work in our
interpreter to close it. Where would we put this function? How would we
call it?

Let's look at the `Ok` block in our code:

```rust
Ok(line) =>
    if line == "(exit)" {
        done = true;
    } else {
        println!("{}",line)
    },
```

We want to trim it when we check for equality. Your code will look like
this when you call it.

```rust
Ok(line) =>
    if line.trim() == "(exit)" {
        done = true;
    } else {
        println!("{}",line)
    },
```

Simple huh? The Standard Library contains many great and nifty methods
and data structures you can use. I encourage you to read through it and
the examples they provide. You might not use it now but it's good to
know what's available. I've even found that I've reimplemented stuff by
accident before and there was already a method to do it. If only I had
just looked!

Last one, closing the interpreter on a `EOF` or `Interrupted`.

## It's all about the Ctrl
Remember our code from before? It looked like this:

```rust
Err(e) => println!("Couldn't readline. Error was: {}", e),
```

Anytime we triggered an error it would print out what it was. Well
Ctrl-c and Ctrl-d are common ways to exit applications like this and
accounting for them would be a good idea. With the `readline` function
they are considered one of the `Error` types in `rustyline`.
Specifically it's called a `ReadlineError`. You can read the
documentation on it
[here](https://kkawakam.github.io/rustyline/rustyline/error/enum.ReadlineError.html). `ReadlineError` is what's known as an
`enum` in Rust. An `enum` is an enumerated type that can be a few
different values. In fact the `Result` type we used before is an `enum`!
It has two possible values, `Ok` or `Err`. In this case we want to match
on the possible values for `ReadlineError`! Let's whip out our match
statement then

```rust
Err(e) => {
    match e {
        rustyline::error::ReadlineError::Eof => done = true,
        rustyline::error::ReadlineError::Interrupted => done = true,
        _ => println!("Couldn't readline. Error was: {}", e),
    }
}
```
This compiles and runs. Try it! Run it then hit Ctrl-c or Ctrl-d and watch
it exit without a fuss. This is really ugly though am I right? Having to
match on the full file path every single time just to use stuff from
another crate? That's crazy talk. Who would do that? Not us! That's why
we'll be using a nice little thing called the `use` statement. If you've
used C++ before it's basically a `namespace`, or `import` in other
languages.

```rust
Err(e) => {
    use rustyline::error::ReadlineError::*;
    match e {
        Eof => done = true,
        Interrupted => done = true,
        _ => println!("Couldn't readline. Error was: {}", e),
    }
}
```

Try it out! It'll again work like before. So what's this `use` statement
doing? Well it's importing all of the `enum` variants in
`ReadlineError`! Weird? Here's a version where we don't use the `*`.

```rust
Err(e) => {
    use rustyline::error::ReadlineError;
    match e {
        ReadlineError::Eof => done = true,
        ReadlineError::Interrupted => done = true,
        _ => println!("Couldn't readline. Error was: {}", e),
    }
}
```

We're just removing how much of that path we need to put in to make sure
Rust knows what we're talking about. I prefer the former. Now you can do
a top level import actually. If I had put `use` at the top then it would
have actually imported that for everywhere in the file! Unlike many
languages we can do imports in the scopes that we want. In this case
I only wanted it for this `match` statement so I wrapped the whole thing
in `{}`. You might have noticed that extra `{}` after the `=>`. This
basically wraps things in it's own scope. It's also what's let us
execute multiple statements after a `match`.

Alright now I've been ignoring that `_` for a while now and you might be
dying to know what it is. It's a catch all pattern in a `match` block.
It's saying, "Alright if anything above me doesn't match then execute
whatever is after my `=>`"

That means in this case if you put:

```rust
Err(e) => {
    use rustyline::error::ReadlineError::*;
    match e {
        _ => println!("Couldn't readline. Error was: {}", e),
        Eof => done = true,
        Interrupted => done = true,
    }
}
```

It will never reach any of the other matches since it comes first! In
fact you'll get an error saying that it's an unreachable pattern. The
compiler saves us again from an egregious mistake!

This is nifty but doesn't it seem a bit redundant that we have the same
code execute on `Eof` or `Interrupted`? Is there a way we could pattern
match on either or and have it execute the same code? Yes there is!

```rust
Err(e) => {
    use rustyline::error::ReadlineError::*;
    match e {
        Eof | Interrupted => done = true,
        _ => println!("Couldn't readline. Error was: {}", e),
    }
}
```

That little `|` says, "If anything on my left or right matches execute
the code of the match." You can chain a bunch of them so that if any of
them match they do the same thing. Cool huh? This looks way cleaner then
our original implementation.

## Conclusion
We've covered all of the code needed to answer the previous article's
questions. We've even learned some new concepts along the way! Next up
we'll begin writing a parser for our language. We'll start off by
checking that all of the parentheses in a statement have a matching one
and if not throw an error. We'll be covering a lot of new material like
writing our own `enum` and implementing the `Error` trait for it so we
can write errors ourselves. It'll be more code then before but don't
worry we'll go through it step by step. I encourage you, in the
meantime, to make sure you understand what we've done so far!

You can find the code from this exercise [here](https://github.com/mgattozzi/schemers/tree/Exercise_1).
The previous article can be found [here](/scheme-input).
The next article can be found [here](/scheme-parser).

