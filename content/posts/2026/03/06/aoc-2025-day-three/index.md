---
title: "Advent of Code 2025 - Day Three"
description: "Electrical surges. Should have bought a bigger surge protector."
summary: "Electrical surges. Should have bought a bigger surge protector." # For the post in lists.
date: '2026-03-06'
aliases:
keywords: ["PowerShell","posh","pwsh"]
author: 'Nathan Ziehnert'
usePageBundles: true
toc: true

featureImage: 'https://images.unsplash.com/photo-1606482714043-600dc0b89ae0?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=350&q=80' # Top image on post.
featureImageAlt: 'Advent Calendar.' # Alternative text for featured image.
featureImageCap: 'Image by Elena Mozhvilo on Unsplash' # Caption (optional).
thumbnail: 'https://images.unsplash.com/photo-1606482714043-600dc0b89ae0?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # Image in lists of posts.
shareImage: 'https://images.unsplash.com/photo-1606482714043-600dc0b89ae0?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # For SEO and social media snippets.

categories:
  - PowerShell
tags:
  - PowerShell
---
## Batteries... the bane of every parent during the holidays...

Welcome back to my weekly (current streak: 3) PowerShell post. Still covering the
[Advent of Code](https://adventofcode.com/) for now.

Something, something, we've got to figure out which batteries are best to use based
on their avaialble power rating (joltage). But they work on powers of 10 based on
their position in the order of "on" from left to right. Silly.

My solutions here: [https://github.com/theznerd/AdventOfCode/tree/main/2025/03](https://github.com/theznerd/AdventOfCode/tree/main/2025/03)

## HE GOT TWO

For the first part of the puzzle, we get to turn on [(͡~ ͜ʖ ͡°) hehe] two batteries in
each bank of batteries. The goal is the largest possible amount of power. As is typical
with these puzzles, we're summing up the value from each row (bank).

Since we're turning on two batteries per bank, we just need to locate the largest number
from the left side (leaving at least one digit to the right - for the second battery)
and then find the largest number remaining in the bank to the right of the first number.

A simple for loop is going to do the trick here, and we can just keep a running total
of the joltage as we go. Locating the first value is easy - we iterate through each
value from lowest index to highest index minus one (remember we need at least two
batteries) and record the value, and the index.

```PowerShell
$maxFirstValue = 0
$maxFirstIndex = 0
for($i = 0; $i -lt ($bank.Length - 1); $i++)
{
  if([Int]::Parse($bank[$i]) -gt $maxFirstValue)
  {
    $maxFirstValue = [Int]::Parse($bank[$i])
    $maxFirstIndex = $i
  }
}
```

To get the second battery, we start our index at the `$maxFirstIndex + 1` and do the
same thing:

```PowerShell
$maxSecondValue = 0
for($i = $maxFirstIndex + 1; $i -lt $bank.Length; $i++)
{
  if([Int]::Parse($bank[$i]) -gt $maxSecondValue)
  {
    $maxSecondValue = [Int]::Parse($bank[$i])
  }
}
```

Now we've got two battery values `$maxFirstValue` and `$maxSecondValue`. You could
do something with powers to generate your voltage value (first * 10, second * 1)
or we can just let `[int]` try to parse it for us - I like that:

```PowerShell
$outputJoltage += [Int]::Parse("$maxValue$maxSecondValue")
```

Easy peasy lemon battery squeezy.

## Twelve ~~days of christmas~~ batteries

Okay, now we have to do the same thing, but this time with twelve batteries
instead of two. At this point the concept is the same, but the index delta
is a little different (have to leave enough space at the end to sure there
are a proper number of batteries).

Now it becomes a bit easier to run our for loop down instead of up. "Wait
a minute," you say, "you can count down intead of up in a for loop?" Why
yes, yes you can. The syntax in this case looks just a little different:

`for($i = 10; $i -gt 0; $i--)`

Theoretically you could count up by two, or down by 7. It doesn't matter.
The syntax of the for loop is effectively:

- Here's some initialization code
- Here's the condition you should check at each loop
- Here's what you do on loop repeats

You can even include multiple things in each:

`for(($i = 0), ($j = 10); $i -lt 10 -and $j -gt 4; $i++, $j--)`

Mind blown.

So back to our loop... agian we're doing the same thing 

```PowerShell
for($i = 12; $i -gt 0; $i--) 
{
  $maxValue = 0

  # search from the right minus $i (leaves enough batteries on the right side to
  # make a bank of $i batteries), but stop before the last value found...
  # basically we want to search between the furthest possible right battery 
  # and the last battery we found in the previous iteration
  for($j = $bank.Length - $i; $j -gt $lastMaxIndex; $j--)
  {
    # we want the largest number searching right to left, but 
    # we want the first occurrence of it from left to right to
    # maximize the remaining batteries for the next searches
    if([Int]::Parse($bank[$j]) -ge $maxValue)
    {
      $maxValue = [Int]::Parse($bank[$j])
      $currentMaxIndex = $j
    }
  }
  $lastMaxIndex = $currentMaxIndex # this is the "leftmost" index for the next search
  $maxValues[$i] = $maxValue # store the max value found for this position
}
$outputJoltage += [BigInt]::Parse($maxValues.Values -join '')
```

Again we're making use of `[Int]::Parse` (or in this case `[BigInt]`) and just
joining all the maxvalues (the batteries selected from the bank) into one big
number and then summing them up.

This one felt pretty easy, and it's always fun to use little tricks that you
may not have been taught when learning loops. If there's anything in the scripts
you want some more info on, I'm happy to share - just let me know in the comments
 below. Otherwise, until next time, Happy Scripting!