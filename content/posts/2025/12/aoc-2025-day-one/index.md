---
title: "Advent of Code 2025 - Day One"
description: "Clocks (or locks... whatever). It's been a hot minute. Bold of me to assume that any of you are still here - or that this time will be different and I'll be able to do both AoC and blog."
summary: "Clocks (or locks... whatever). It's been a hot minute. Bold of me to assume that any of you are still here - or that this time will be different and I'll be able to do both AoC and blog." # For the post in lists.
date: '2025-12-01'
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
## Clocks (or locks... whatever)

Day one was a fairly simple problem - keep track of where the dial is pointing. You can imagine this as a
clock, or a combination lock, or a circle with numbers on it. Whatever flys your plane. The dial ranges 
from 0 - 99 and you're given a list of instructions for how far to turn the dial, and in which direction.
Nevermind that the elves have left and right backward...

My solutions here: [https://github.com/theznerd/AdventOfCode/tree/main/2025/01](https://github.com/theznerd/AdventOfCode/tree/main/2025/01)

## Part 1 - Stopping on Zero

Part one of the problem involves counting how many times we've stopped on the number zero. The
list of instructions is formatted as `DN` where `D` is the direction (L or R) and `N` is an integer
representing the number of ticks to move the dial. Quick and dirty way to separate this is just to
switch on the first character of the instruction, and then add or subtract from the dial position using
the remaining characters (`.Substring(1)`).

Since we know that the boundary for the dial is 100, we can simply just move the dial positive or
negative, and then take the modulus of the current value. If the result is 0 (0 % 100, 100 % 100,
200 % 100, -100 % 100) then we know we've landed on zero and we can increment the counter. If it's
anything else, ignore and move on to the next instruction. In the case of using the modulus,
-100, 0, 100, 200, etc. are the same position on the dial so we don't need to reset the dial position
to be within the bounds of 0-99 - we get the same result regardless:

```Text
Dial Position: 50
Instructions: L283, R384, R28, L79

Absolute:
  50 -> -233 (L283)
-233 ->  151 (R384)
 151 ->  179 (R28)
 179 ->  100 (L79)

Relative (0-99 boundary):
  50 -> 67 (L283)
  67 -> 51 (R384)
  51 -> 79 (R28)
  79 ->  0 (L79)
```

Doing this in PowerShell is pretty straightforward. Give ourselves a variable to store the dial
position, a variable to count how many times we landed on zero, and then a loop to parse through
the instructions.

```Powershell
$dialposition = 50
$counter = 0
foreach($instruction in $puzzleInput)
{
    switch($instruction[0]){
        "L" { $dialposition -= $instruction.Substring(1) }
        "R" { $dialposition += $instruction.Substring(1) }
    }
    if($dialposition % 100 -eq 0){$counter++}
}
```

## Part 2 - Touching Zero

Surprise! Now we need to keep track of every time we touch zero when moving the dial. This
will not include starting on zero, but will include ending on zero. Basically - if turning
the dial causes zero to be touched, then we increment the counter. It becomes easier if we
reset the value of the dial position variable to be within the boundary - if anything
at least for debugging to account for edge cases.

A smart way to get two birds with one stone here would be to subtract or add 100 to the
current dial position until you are back within the bounds of the dial (for example, if
you end at 384, you subtract 300 and therefore know that you've crossed 0 three times
and then your dial is normalized back to 84). I like convoluted solutions... so I did
something similar, but I used math instead of a loop to get there.

Basically, we take the floor of dividing the absolute value of the dial position by 100.
So `-328` gives a value of 3 (`|-328| / 100 = 3.28` - floor is 3). `84` gives a value of
0 (`|84| / 100 = 0.8` - floor is 0). If the starting position was not zero, and we ended
negative, we need to add one more to the counter (since we crossed zero to get there, but
it will not be counted in the floor equation). Similarly, if we land directly on zero
(e.g. starting at 51 and instruction is L51), we also need to add one to the counter.

After we calculate this, we need to reset the dial position variable to the boundary of
the dial. We can use modulus again, and if we're negative then add 100. Easy peasy,
overcomplicated lemonade squeezy.

```PowerShell
$dialposition = 50
$counter = 0
foreach($instruction in $puzzleInput)
{
    switch($instruction[0]){
        "L" { $dialposition -= $instruction.Substring(1) }
        "R" { $dialposition += $instruction.Substring(1) }
    }
    $counter += [Math]::Floor([Math]::Abs($dialposition) / 100) # count full rotations (above or below zero)
    if($dialposition -lt 0 -and -not $dialStartedAtZero){ $counter++ } # when we cross over to the negative we need to add one as the above floor will not count that
    if($dialposition -eq 0){ $counter++ } # when we land on exactly 0, we need to increment the counter, as the floor will not count that

    # reset dial position within bounds (0-99)
    $dialposition = $dialposition % 100
    if($dialposition -eq 0){ $dialStartedAtZero = $true } else { $dialStartedAtZero = $false } # for the crossover check
    if($dialposition -lt 0){ $dialposition += 100 }
}
```

Now don't ask me how long it took me to troubleshoot my logic at 11pm - because I did not
immediately account for crossing zero when swinging negative and landing on zero exactly.
There is a reason sleep is good for your brain. With that said... I need sleep... so...

Until next time, Happy Scripting!
