---
title: "Advent of Code 2023 - Day Two"
description: "Day two is the level I expected for day one"
summary: "Day two is the level I expected for day one" # For the post in lists.
date: '2023-12-02'
aliases:
keywords: ["PowerShell","posh","pwsh"]
author: 'Nathan Ziehnert'
usePageBundles: true
toc: true

featureImage: 'https://images.unsplash.com/photo-1606482512676-255bf02be7cf?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=350&q=80' # Top image on post.
featureImageAlt: 'Advent Calendar.' # Alternative text for featured image.
featureImageCap: 'Image by Elena Mozhvilo on Unsplash' # Caption (optional).
thumbnail: 'https://images.unsplash.com/photo-1606482512676-255bf02be7cf?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # Image in lists of posts.
shareImage: 'https://images.unsplash.com/photo-1606482512676-255bf02be7cf?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # For SEO and social media snippets.

categories:
  - PowerShell
tags:
  - PowerShell
---
## Round Two, Fight!
Day Two was a cakewalk for the most part (aside from a few minutes scratching my head
as I got my minimum/maximum backward). 

![I always do that. I always mess up some mundane detail](mess-up-mundane-detail.gif " ")

Here's my solution - we'll break down parts one and two in this post. 
[GitHub](https://github.com/theznerd/AdventOfCode/tree/main/2023/02)

## Part 1 - A Little Data Cleanup and a Hashtable
So the basics of the problem were you played a game - the game involved a bag of
colored cubes (red, green, blue) and an elf pulls out a handful of cubes three
times during the game. The elf asks you "If there were only supposed to be a
maximum of 12 red, 13 green, and 14 blue cubes in the bag, which of the games
am I dirty rotten filthy cheater" (some embellishment on my part maybe).

You've got a list of the games played and the cubes drawn... some games maybe
the elf drew 14 red in one draw which would mean he was a filthy cheater. We
know how many cubes should be in the bag. The elf wants to know the games that
they didn't cheat on, so add up the game number if the game was fair. This is 
fairly simple with a fun little PowerShell trick:

```PowerShell
$maxValue = @{
    red = 12
    green = 13
    blue = 14
}

$sum = 0
:outer for($i = 1; $i -le $puzzleInput.Count; $i++)
{
    $gameId = $i
    $pulls = ($puzzleInput[($i-1)])[(7 + ([string]$i).Length)..($puzzleInput[($i-1)].Length)] -join '' -split '; '
    foreach($pull in $pulls)
    {
        $colors = $pull -split ", "
        foreach($color in $colors)
        {
            $number, $colorid = $color -split ' '
            if($maxValue[$colorid] -lt [int]$number){continue outer}
        }
    }
    $sum += $gameId
}
$sum
```

The basic premise is - we'll create a hashtable with the maximum values for
each color, then loop through each game (and each draw within the game - 
there are three per game). If we ever encounter a condition where the drawn 
number is greater than the max value, we throw the cubes back at them and
continue on to the next game without checking the remaining draws (if any).

Since we're nesting loops, and we're checking inside the nest, it's best to
just continue the outer loop (skip the remainder of that loop iteration and
start with the next time through the loop). In PowerShell we can accomplish
this by labeling our loop - I conveniently called the outer/parent loop
`:outer`. Then when we reach a condition we want to continue on, we just
call `continue outer`. Because we're adding the game id to the `$sum` var
at the end of the `:outer` loop, it conveniently skips that step unless
we never reach the condition of the elf cheating.

The other little "trick" being used here, is instead of doing some sort of
switch statement on the color, we're referencing the max value in the 
hashtable for that color: `$maxValue[$colorid]` to keep our code clean.

Easy peasy, even if the elf is a scoundrel.

## Part 2 - Where We Assume The Elf Isn't Cheating
Okay, so maybe the elf was wrong about the number of cubes in the bag,
but he forgot how many were in there. Now he wants to know for each game
what is the minimum number of cubes that had to be in the bag for that
game to be possible. I think he wants to know because he wants me to
take the knife away from his throat after swindling me out of a thousand
bucks.

We'll use a hashtable again, but this time set it up for each game. Since
he wants us to do multiplication of the colors for some odd reason, we'll
give each color a default minimum of 1 (so that if a one color is never
pulled during the game it doesn't throw off our multiplication table -
1 times anything is still just the anything).

```PowerShell
$sum = 0
for($i = 1; $i -le $puzzleInput.Count; $i++)
{
    $pulls = ($puzzleInput[($i-1)])[(7 + ([string]$i).Length)..($puzzleInput[($i-1)].Length)] -join '' -split '; '
    $lowestValue = @{
        red = 1
        green = 1
        blue = 1
    }
    foreach($pull in $pulls)
    {
        $colors = $pull -split ", "
        foreach($color in $colors)
        {
            $number, $colorid = $color -split ' '
            if($lowestValue[$colorid] -lt [int]$number){$lowestValue[$colorid] = [int]$number}
        }
    }
    $gamePower = $lowestValue['red']*$lowestValue['blue']*$lowestValue['green']
    $sum += $gamePower
}
$sum
```

We do basically the same thing here except this time instead of
referencing the value for lookup, we'll also update the value with
a new minimum if he drew more than the current known minimum. For
example - if he drew 4 red cubes, but so far I've only seen him draw
2 red cubes, the new minimum value for the red cubes this game is 4.
We don't have to continue on the loop this time because we do want
to evaluate every draw/pull. After we evaluate each draw for the game
then we multiply the minimum number of cubes for each color together
and add it to the sum. I'm told this number will help me get to some
destination unknown, but I'm convinced he just is asking for gibberish
now.

## Closing Thoughts
Closing Thought 1: _Why didn't I use Regex to manipulate the data?_
<br>Meh... I'm lazy and doing some good old fashioned string manipulation
seemed easy to me for today. Regex probably would have been cleaner than
the nightmare that this is: `($puzzleInput[($i-1)])[(7 + ([string]$i).Length)..($puzzleInput[($i-1)].Length)] -join '' -split '; '`

Closing Thought 2: _You mentioned making a mundane mistake... what was it?_
<br>When I was doing part two, I had originally set the hashtable to
`[int]::MaxValue` and when I found a lower value I updated it. All this
accomplished was telling me the lowest number pulled/grabbed for each
color during a game. I needed the max value pulled during a game. Twas
a silly mistake.

Happy Scripting!