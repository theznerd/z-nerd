---
title: "Keeping The Streak: Just Under The Wire. Something, Something, Error Handling."
description: "We spend a bit of time this week talking about error handling in PowerShell. All because I spent the day staring at roller coaster procedures and electronics instead of the animals at Seaworld."
summary: "We spend a bit of time this week talking about error handling in PowerShell. All because I spent the day staring at roller coaster procedures and electronics instead of the animals at Seaworld." # For the post in lists.
date: '2026-03-13'
aliases:
keywords: ["PowerShell","posh","pwsh"]
author: 'Nathan Ziehnert'
usePageBundles: true
toc: true

featureImage: 'https://images.unsplash.com/photo-1574352787491-d968efc29428?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=350&q=80' # Top image on post.
featureImageAlt: 'The Bird is on the Wire' # Alternative text for featured image.
featureImageCap: 'Image by Streetwindy on Unsplash' # Caption (optional).
thumbnail: 'https://images.unsplash.com/photo-1574352787491-d968efc29428?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # Image in lists of posts.
shareImage: 'https://images.unsplash.com/photo-1574352787491-d968efc29428?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # For SEO and social media snippets.

categories:
  - PowerShell
tags:
  - PowerShell
---
## I'm On Vacation... But I Can't Let The Streak Die

Welcome back to my weekly (current streak: 4) PowerShell post. I'm on vacation
this week, but I couldn't let my infant streak die. So here we are, just under
the wire on Friday night. We spent the day at Seaworld, and of course, rather
than being fascinated by the animals, I was really fascinated by all the
technology and engineering that goes into the roller coasters and rides. I mean
the animals were cool too, I guess. Here's what staring at roller coaster
electronics all day ~~taught me about B2B sales~~ gets you for a blog post this
week.

## Error Handling

Almost all of the systems I saw today had at least one layer of redundancy built
in, and many of them had multiple (or at least checks to make sure that the
process would end with me in one piece at the end of the ride). Also the
procedures for the ride operators were very specific and detailed, including
what to do in the case of an error condition. Most coding languages have some
sort of error handling built-in, and PowerShell is no exception. In PowerShell,
we have [`try/catch/finally` blocks](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_try_catch_finally)
which allow us to handle errors gracefully. However, I'm going to share three
ways that I have seen people use these blocks that could be improved.

I'm going to make the assumption that most people are at least basically
familiar with how `try/catch/finally` blocks work. If not, please read the
documentation linked above. The most basic explanation if you can't be bothered
to RTFM, is that you put code that might throw an error in the `try` block, and
then you can either catch the error (either specifically or generally) in the
`catch` block, and then you'd run any code that needs to run regardless of
whether an error was thrown or not in the `finally` block. You can have
`try/catch/finally`, `try/catch`, or `try/finally` blocks, but not `try` alone.
Why would you even want to do that?

![straight to jail](image.png)

## Error Number One: Wrapping Everything In A Try/Catch Block With A Generic Error Message

I get it - you've got this fancy tool that you want to use, and you want to use
it for everything. So you wrap your entire script in a `try/catch` block, and
then you come up with the most amazing error message that you can think of when
something inevitably goes wrong: "An error occurred. Please try again." Wow,
thanks for that super helpful error message. Here's the problem - I now have no
idea what went wrong or what I can do to fix it other than give it a minute and
try again. Going back to roller coasters, this would be like wrapping the entire
ride procedure from start to finish in one big procedure block, and if anything
goes wrong, the ride just stops and the operator says "wanna try again?"

Hell naw.
![hell naw](image-1.png)

If you're going to use `try/catch` blocks, use them for specific sections of
code that you know might throw an error, and then leave the rest of your code
to the professionals (the built-in error handling of the language). This serves
two purposes:

1) It allows you to provide more specific error messages that can help you or
the user understand what went wrong and how to fix it.
2) The built-in error handling of PowerShell gives you a lot of information
about errors that you didn't capture/anticipate, which will help you debug
easier.

There are certainly cases where you might want to wrap the whole script in a
`try` block and then pair it with a `finally` block (to always log a terminating
error, for example). I'm mostly complaining here about `try/catch` with a basic
ass error message in the `catch` block that doesn't help anyone.

## Error Number Two: Not Cleaning Up After Yourself

There is a reason that we have `finally` blocks in PowerShell - they allow us to
clean up after ourselves regardless of whether an error was thrown or not. This
is one of the few times where wrapping a large portion of your script in a try
block can make sense. For example, assume that you're writing a script which
creates a temporary web server bound to some port, and that web server is
used through the whole lifetime of the script. If a terminating error is thrown
before you dispose of your web server, you might end up leaving a server running
and still bound to that port. If you then try to run your script again, it will
throw an error because the port is already in use. Annoying.

Maybe you're writing a script that creates a bunch of temporary files. Maybe
you're writing a script that changes some system settings temporarily that you
need to change back. Maybe you want to make sure that you always log something
to a file to indicate the script (or that block) ran, even if it threw an error.
Maybe you want to compare the state of something before and after the block,
even if it threw an error. These are all examples of things that you could use
the `finally` block for. Ask your favorite generative AI to come up with other
use cases if these don't tickle your fancy.

## Error Number Three: Not Actually Handling The Error

Look, sometimes you just want to catch the error and rethrow it in a more
user-friendly way. That's fine. I know the type of people who need that
handholding ("OMG, it said 'Access Denied', am I getting fired?"). Also, I
recognize there is value in context - sometimes it's not just about seeing the
error, but knowing what the implications of the error are. However, one beauty
of `try/catch` blocks is that if you can recover from the error and keep going,
you can do that. One ride in particular I rode today had a procedure I saw
executed multiple times. If the lap bar didn't quite set into place - one of
the ride operators would request the lap bars be released and then reset them
for the whole row. If the lap bar still didn't set into place, they would
remove the rider from the row and then try again. (I assume that if the lap bar
still didn't set into place after that, they would have to pull the train or
possibly just mark that seat out of order, but for the sake of this example,
we'll leave it at two checks).

Now, by the time we've reached this part of the ride procedure, the other riders
on the train are already strapped in and ready to go. Would be kind of a pain in
the ass to have to start the whole process over again just because one rider's
lap bar had a minor hiccup. I assume you've encountered a script that has a
similar problem before - you've just spent 120 seconds (yeah, our brains are
full on rot now) waiting to get to this section of your script, only for it to
fail because the network cable came a bit loose (I don't know, come up with your
own reason for why the script failed at this specific moment). You know that
this is a pretty common thing that happens, so you catch your error and give the
script operator a chance to correct it (or maybe you're able to handle it
without involving them at all). Bingo bango you've just saved that user the pain
of running the script for two minutes again.

Again, unless we're doing some handholding, I don't think there is a need for
you to rephrase every red block of text that PowerShell outputs - unless you're
adding context to the error (e.g. "Failed to create the VM: `$VMName`"). If I
see try/catch routines peppered throughout your scripts without context, it's a
[code smell](https://en.wikipedia.org/wiki/Code_smell).

## Cleaning Up After Myself

Don't be afraid to use `try/catch/finally` blocks in your scripts, nobody is
really judging you. I always say the best code is the code that works for you.
However, if you want to write code that is easier to debug and maintain, then
maybe consider using `try/catch/finally` blocks more strategically. With that,
I am off to bed to be present for my vacation tomorrow.

As always, Happy Scripting!

## One More Thing...

Don't forget that "catch" only catches **terminating errors**. If you want to
catch non-terminating errors, you need to use the `-ErrorAction` parameter when
available, or set the `$ErrorActionPreference` variable to make them
terminating. For example, if you want to catch a non-terminating error from
`Get-ChildItem`, you could do something like this:

```PowerShell
try {
    Get-ChildItem -Path "C:\NonExistentPath" -ErrorAction Stop
}
catch {
    Write-Host "An error occurred. Please try again. :)"
}
```
