---
title: "Advent of Code 2025 - Day Four"
description: "Killed my streak last week. Forgot to blog while curling. Dang."
summary: "Killed my streak last week. Forgot to blog while curling. Dang." # For the post in lists.
date: '2026-03-27'
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
## Oops... killed the streak.

Welcome back to my weekly (current streak: 1) PowerShell post. 

![0 week blogging streak](image.png)

I'm in Seattle this week - close to the MVP Summit, but sadly did not book
in person tickets soon enough. Next year (assuming I'm renewed). I was out
in a curling tournament last week and completely forgot. Oops! Going back
to talking about Advent of Code. This week the elves need help optimizing
their forklift use. 

My solutions here: [https://github.com/theznerd/AdventOfCode/tree/main/2025/04](https://github.com/theznerd/AdventOfCode/tree/main/2025/04)

## Single Iteration

We've got these rolls of paper used to make decorations. Apparently they
have some odd magnentic field or something associated with them because
the forklifts can only access the roll of paper if there are no more than
3 adjacent rolls to the roll you're trying to remove. This is in 8 directions
(N, NE, E, SE, S, SW, W, NW). Our puzzle input is an XY grid of the storage
room floor - empty points represnted by `.` and rolls represented by `@`.
We do not have to worry about the path to the roll being clear - so no need
to validate a path from outside of the map to the roll in question.

For a problem like this I like to create the map as a hashtable using
the XY coordinates as the key "(x,y)". We can use the following block
to convert the script input into this hashtable map:

```PowerShell
$rollMap = @{}
for($y = 0; $y -lt $puzzleInput.Count; $y++)
{
    for($x = 0; $x -lt $puzzleInput[$y].Length; $x++)
    {
        $rollMap["$x,$y"] = $puzzleInput[$y][$x]
    }
}
```

Now to determine if we can gain access to the roll, we need to determine
if there are less than four rolls next to it in any of the eight
directions (like an old fashioned controller input). The way I like to
do this is by creating a mask and then iterating through that mask to
get the updated coordinate. Why I mean by this is, if my coordinate is
`8,8` it is easier for me to build a mask table that says to add or
subtract one from one or both coordinates depending on the direction 
(for example, N would be "0" (add zero to y) and "-1" (subtract one
from x)). My mask looks like this:

```PowerShell
$mask = @{
    "N" = @("0","-1")
    "NE" = @("1","-1")
    "E" = @("1","0")
    "SE" = @("1","1")
    "S" = @("0","1")
    "SW" = @("-1","1")
    "W" = @("-1","0")
    "NW" = @("-1","-1")
}
```

Now we grab all the rolls, and then iterate through the mask directions
and count whether or not that masked position has a roll as well (and
keep track of how many rolls we got).

```PowerShell
$accessibleRolls = 0
foreach($roll in $rollMap.GetEnumerator().Where({$_.Value -eq "@"}))
{
    $x,$y = $roll.Key -split ","
    $countRolls = 0
    foreach($direction in $mask.Keys)
    {
        $dx, $dy = $mask[$direction]
        $nx = [int]$x + [int]$dx
        $ny = [int]$y + [int]$dy
        if($rollMap["$nx,$ny"] -eq "@")
        {
            $countRolls++
        }
    }
    if($countRolls -lt 4){$accessibleRolls++}
}
$accessibleRolls
```

Two things to point out here:

1) Going back to what I mentioned about using a hashtable from earlier
   instead of a multidimensional array, is that it's stupid simple to
   just grab the value from the hashtable using the string points (x,y).
   `$rollMap["$nx,$ny"]`
2) I could have broken out of the loop if as soon as I found 4 rolls and
   used that as a condition to increment the `$accessibleRolls` counter.
   I was just lazy, and frankly it was already pretty fast.

We run this once algorithm once against all the rolls and then output our
count. Bingo we know how many rolls can be accessed.

## Of course they can access more

What the elves failed to realize (surprise, surprise) is that once you
remove an accessible roll, you might then be able to access more. So now
we need to do some state tracking. This is easily accomplished by adding
a few items:

1) A state of which rolls we've removed (for the count)
2) Removing rolls that are accessible (change them from `@` to `.`)
3) Whether or not we removed any rolls in the last execution

I overcomplicated it and kept an initial roll count instead of just
counting rolls I removed as I removed them (combining 1 and 2 from
above). I also could have removed them as I went rather than removing
them enmasse after I determined which ones were accessible. I'm not
sure the latter would have made it significantly faster, but depending
on the size of the dataset it could have made a difference. However,
the example DID show removing them all at once, and sometimes when the
puzzle master does that it means there is an edge case where it's
important to do it at once instead of as you go.

So now our algorithm looks something like this:

```PowerShell
$initialRollCount = $rollMap.GetEnumerator().Where({$_.Value -eq "@"}).Count
do{
    $rollsRemoved = $false
    $rollsToRemove = @()
    foreach($roll in $rollMap.GetEnumerator().Where({$_.Value -eq "@"}))
    {
        $x,$y = $roll.Key -split ","
        $countRolls = 0
        foreach($direction in $mask.Keys)
        {
            $dx, $dy = $mask[$direction]
            $nx = [int]$x + [int]$dx
            $ny = [int]$y + [int]$dy
            if($rollMap["$nx,$ny"] -eq "@")
            {
                $countRolls++
            }
        }
        if($countRolls -lt 4){
            $rollsToRemove += $roll.Key
            $rollsRemoved = $true
        }
    }
    foreach($rollKey in $rollsToRemove)
    {
        $rollMap[$rollKey] = "."
    }
}while($rollsRemoved)
$finalRollCount = $rollMap.GetEnumerator().Where({$_.Value -eq "@"}).Count
$initialRollCount - $finalRollCount
```

As I said, it's overcomplicated (especially with the `$initialRollCount` and
`$finalRollCount` mess) but we got it done and put day 4 out of 12 in the
books. So far this is feeling much easier than year's past (espeically
considering this year is half of the puzzle count from last year).

Any questions about what I did, feel free to dump into the comments. Otherwise,
as always, Happy Scripting!