---
title: "Advent of Code 2025 - Day Two"
description: "Kids are dumb. Adults who let kids play on production systems are dumber."
summary: "Kids are dumb. Adults who let kids play on production systems are dumber." # For the post in lists.
date: '2026-02-27'
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
## Darn Kids

Welcome back to my weekly (current streak: 2) PowerShell post. Since I did a bunch
of [Advent of Code](https://adventofcode.com/) this year but only blogged about one
of them, these are probably appropriate topics to cover.

Apparently the elves gave a younger elf access to a production cash register and now
there's a bunch of fake product ids on their system. The little sh*t was just doing
patterns though, and apparently we never create a product id with a pattern, so it's
safe to remove them.

My solutions here: [https://github.com/theznerd/AdventOfCode/tree/main/2025/02](https://github.com/theznerd/AdventOfCode/tree/main/2025/02)

## Once, Twice, two is a pattern?

Okay. To set the stage we've got a big list of number ranges. These number ranges can
be small (2-3 digit numbers) all the way to big (10+ digits - becomes important here
in a bit). We have to find all the numbers in the range that are a sequence of numbers
repeated twice (e.g. 55, 6464, 123123, etc.). So that makes this problem fairly easy to
brute force. Take half the length of the range (may have to do this twice if the range
breaches a digit barrier [e.g. 7-25]), generate all the possible patterns in the search
space, and check to see if the possible pattern is in the range.

Really not worth boring you with the details of how to input and split text anymore,
it's been covered ad naseum. However I do want to point out a fun little tidbit of
PowerShell syntax - [Multiple Assignment](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_assignment_operators?#assigning-multiple-variables):

```PowerShell
$rangeStart, $rangeEnd = $range -split "-"
```

If you didn't click the link - here's what it is. If you've got an array on the right
side of your equals operator, the array elements are assigned to the comma-separated
values on the left side of the equals operator - in one line of code. The link above
goes into more detail, but the short of it is

- n variables = n elements - each variable gets one element
- n variables = >n elements - the final variable gets the remaining elements
- n variables = <n elements - variables with no elements are assigned `$null`

[![thats pretty neat](thats-pretty-neat.png)](https://youtu.be/Hm3JodBR-vs)

The next overcomplicated bit of logic determines what our search space is. It'd be
easier to just dump out the entire list of patterns (there aren't *that* many), but
par for the Z-Nerd golf course is overcomplicating things.

```PowerShell
if($rangeStart.Length % 2 -eq 0)
{
  # if the starting range is an even number of digits we start here (with the first half of number)
  $startSearch = $rangeStart.Substring(0,($rangeStart.Length / 2))

  # if the ending range is + 1 digit, we only go up to the max for half the starting range digit length
  if($rangeLengthDelta -eq 1){$endSearch = "9" * ($rangeEnd.Length / 2)}
  else{$endSearch = $rangeEnd.Substring(0,($rangeEnd.Length / 2))}
}
elseif($rangeEnd.Length % 2 -eq 0)
{
  # if the starting range is an odd number of digits, we start at lowest number for the end range length (e.g. 100 for 3 digit start
  # and then go up to the half-length of the even end range)
  $startSearch = "1" + ("0" * ((($rangeEnd.Length) / 2) - 1))
  $endSearch = $rangeEnd.Substring(0,($rangeEnd.Length / 2))
}
```

Here's the lowdown. One key point is that the delta in digit length was never more
than 1 when I checked, so we didn't have to factor in going from say 4-283.

1. Start of range is an even number of digits
    1. We'll start with the first half of the start of the range (e.g. if range start is
       12384578, we start with 1238)
    2. If the end range length is > than the start range length, then we'll end our search
       at the 9*n of the half length of the start range (e.g. from above 12384578, we end
       with 9999). If the end of the range is the same number of digits as the beginning,
       just take the first half of the end of the range (e.g. if range end is 54323848,
       we end with 5432).
2. Start of range is an ODD number of digits, but end range is even
    1. We start at the beginning of the even range (e.g. if range end is 12384578, we 
       start with 1000)
    2. We end the search at the first half of the end of the range (e.g. if range end is
       54323848, we end with 5432)

From here we create the list of patterns based on our search space generated above.

```PowerShell
$possibleCandidates = for($i = [long]$startSearch; $i -le [long]$endSearch; $i++){"$i$i"}
```

Basically just pour me a double of whatever the number is and dump it into `$possibleCandidates`

Now that we've got a list of candidates, we'll work through them one by one and test whether
they are in the range or not (there's a chance that the search space generated numbers that
are not in the range - for example 12135843 start, would generate 12131213 as a potential 
candidate, but it's less than 12135843 so it's not in the range).

Then we have to sum up all of the invalids. This is where what I mentioned earlier about
the long numbers comes into play... the answer could literally be `[long]`. So when we create
our possible candidates, and when we build our ranges, we are casting to `[long]` just 
to be sure we don't hit a max limit on `[int]`.

Pretty simple - took longer to write this post than it did to write the script.

## Oops... all patterns...

Surprise surprise, we now have to account for different types of patterns where the
sequence of digits is repeated at least twice (e.g. 11, 111, 1212, 121212, etc.).
Don't know why we didn't think about this at first when we consider the little sh*t
was making patterns.

This time since we know we'll use both the even and odd lengths in the range, I decided
to split them to make it easier to reason about.

```PowerShell
$rangesToSearch = @()
if($rangeStart.Length -eq $rangeEnd.Length)
{
    $rangesToSearch += ,@($rangeStart, $rangeEnd)
}else{
    $firstRangeEnd = ("9" * $rangeStart.Length)
    $secondRangeStart = "1" + ("0" * ($rangeEnd.Length - 1))
    $rangesToSearch += ,@($rangeStart, $firstRangeEnd)
    $rangesToSearch += ,@($secondRangeStart, $rangeEnd)
}
```

Effectively what we're doing here is just taking the whole range if the lengths are
the same (e.g. 100-238), or creating two ranges to search (e.g. 182-999, 1000-1874 if
the range was 182-1874). We need to generate candidates again like we did before, but
this time we have to consider patterns of length 1, up to patterns of length 1/2 the
length of the end of the range. We take the floor of that 1/2 measurement to ensure we
don't create a pattern length 5 for a range length of 9 (5 + 5 digits is more than 9).

We'll just iterate in a for loop over the possible pattern lengths, and generate all
the matches that match the length of the range when repeated (e.g. a 3-digit pattern
will not work for a 7-digit number).

```PowerShell
$patternRepeatCount = [math]::Floor($subRangeStart.Length / $i) #i is pattern length
...
[long[]]$possibleCandidates += for($p = $rangePatternStart; $p -le $rangePatternEnd; $p++)
  {if (("$p" * $patternRepeatCount).Length -eq $subRangeStart.Length -and 
       ("$p" * $patternRepeatCount).Length -gt 1)
       {"$p" * $patternRepeatCount}
  }
```

It looks a little complicated, but it does the job. I probably could have made use of
a modulus to check to see if the pattern repeat count even made sense instead of checking
it each time I was adding candidates, but remember I do these puzzles at like 11pm.

After this - we do the same thing we did before... check all the patters to see if they're 
in range, and if so add them to a big 'ol sum.

If there's anything in the scripts you want some more info on, I'm happy to share - just
let me know in the comments below. Otherwise, until next time, Happy Scripting!