---
layout: post
title:  "The Set Function"
date:   2017-04-08 12:53:14
categories: update games math
---

The card game Set is a favorite of my family's. For those unfamiliar it is a card game that has 81 cards in the deck, each with a unique combination of attributes. Each card has one of three shapes on it. The shape is one of three colors, and has one of three fill types, and there will be one to three copies of the shape. So each card as four attributes (count, fill, color, shape) each will three possible values. The deck contains all combinations, which is where the number of a cards in the deck comes from. 3^4 = 81.

The game is played by initially laying out 12 cards. All players (of which there can be any number) then try to be the first to find a "set" of three cards. A valid set is one where for each of the four attributes on the three cards, the values are all the same or all different. So the three cards must all contain the same number of shapes or all different number of shapes, as well as all the same fill type or all different fill types, as well as all the same color or all different color, as well as all the same shape or all different shape. When a player identifies a set they first say "set" in order to claim it. The cards are then taken by the player and new ones from the deck are laid out in their place. If all players are unable to find a set then another three cards may be laid out, though when sets are found you don't deal more if there are already more than 12 cards out. The game ends when there are no more cards in the deck and no more sets can be found. The player with the most sets is the winner.

Because of a project I was working, which is briefly discussed near the end, I desired to have a purely mathematical method of identifying valid sets. The final goal was to have a single expression that takes two integers, each representing a card, and returns the number that represents the third card needed to create a valid set with the first two. So I needed a function f(A, B) that returns C, where A, B, C are three cards that form a valid set.

If you want a decent challenge stop here and try to come up with the function on your own. For reference, it took me about 3 days.

From here I'll explain the solution and how I arrived at it.

Step 1
======

The first step is to number the cards. In my mind there was really only one way to do this. I don't even remember trying other methods. You sort them and number them starting from 0. Starting from 0 may seem odd to non-programmers, but its important. By "sort" I mean by each of the card's four attributes. It doesn't matter which order you go through the attributes and their values as long you stay consistent throughout the whole process. I chose the one that sounded most natural when spoken: count, fill, color, shape. The values for each attribute go in this order:

<table class="tableh">
	<tr>
		<th>count:</th> <th>one</th> <th>two</th> <th>three</th>
	</tr>
	<tr>
		<th>fill:</th> <th>empty</th> <th>line</th> <th>solid</th>
	</tr>
	<tr>
    <th>color:</th> <th>red</th> <th>green</th> <th>purple</th>
	</tr>
	<tr>
		<th>shape:</th> <th>oval</th> <th>diamond</th> <th>squiggle</th>
	</tr>
</table>

So the oder and numbering of all the cards looks like this:

<table class="tableh">
	<tr>
		<th>0:</th> <th>one</th> <th>empty</th> <th>red</th> <th>oval</th>
	</tr>
	<tr>
		<th>1:</th> <th>one</th> <th>empty</th> <th>red</th> <th>diamond</th>
	</tr>
	<tr>
    <th>2:</th> <th>one</th> <th>empty</th> <th>red</th> <th>squiggle</th>
	</tr>
	<tr>
		<th>3:</th> <th>one</th> <th>empty</th> <th>green</th> <th>oval</th>
	</tr>
	<tr>
		<th>...</th>
	</tr>
	<tr>
		<th>80:</th> <th>three</th> <th>solid</th> <th>purple</th> <th>oval</th>
	</tr>
</table>

There happens to be something very nice about this ordering that connects each card to its index and each index to its card. Lets assign a number to the values of each attribute. In other words one = 0, two = 1, three = 2, empty = 0, line = 1, solid = 2, etc.

<table>
	<tr>
		<th></th> <th>count</th> <th>fill</th> <th>color</th> <th>shape</th>
	</tr>
	<tr>
		<th>0</th> <th>one</th> <th>empty</th> <th>red</th> <th>oval</th>
	</tr>
	<tr>
    <th>1</th> <th>two</th> <th>line</th> <th>green</th> <th>diamond</th>
	</tr>
	<tr>
		<th>2</th> <th>three</th> <th>solid</th> <th>purple</th> <th>squiggle</th>
	</tr>
</table>

Now lets list the cards again, using these numbers as well.

<table class="tableh">
	<tr>
		<th>0:</th> <th>one</th> <th>empty</th> <th>red</th> <th>oval</th> <th>0000</th>
	</tr>
	<tr>
		<th>1:</th> <th>one</th> <th>empty</th> <th>red</th> <th>diamond</th> <th>0001</th>
	</tr>
	<tr>
    <th>2:</th> <th>one</th> <th>empty</th> <th>red</th> <th>squiggle</th> <th>0002</th>
	</tr>
	<tr>
		<th>3:</th> <th>one</th> <th>empty</th> <th>green</th> <th>oval</th> <th>0010</th>
	</tr>
	<tr>
		<th>...</th>
	</tr>
	<tr>
		<th>80:</th> <th>three</th> <th>solid</th> <th>purple</th> <th>oval</th> <th>2222</th>
	</tr>
</table>

It turns out that if you treat those 4 digits as a base 3 number it exactly lines up with the index of the card. For example, if we treat 2222 as base 3 and convert it into base 10 we get `2*27 + 2*9 + 2*3 + 2 = 80`, which is the index. Importantly, we can also go the other way. Index 10 is 0101 in base 3, which from the table is "one line red diamond". So if we know the card we know the index, and if we know the index we know the card. They are now one and the same.

So now, with the perfect numbering scheme for our cards, we can easily use them in any mathematical expression. This is where the real fun begins. If you haven't found the solution on your own yet, and haven't tried with this numbering scheme, feel free to give it another shot before continuing.

Step 2
======

The first thing I tried was to see if there was some sort of pattern. So I made a list of all inputs and their outputs. Eventually even including imaginary sets where all three cards are the same, and going back and forth between deciding whether to include all the different permutations identical sets can appear in.

<table class="tablev">
	<tr>
		<th>A</th> <th>B</th> <th>C</th>
	</tr>
	<tr>
		<th>0</th> <th>0</th> <th>0</th>
	</tr>
	<tr>
		<th>0</th> <th>1</th> <th>2</th>
	</tr>
	<tr>
		<th>0</th> <th>2</th> <th>1</th>
	</tr>
	<tr>
		<th>0</th> <th>3</th> <th>6</th>
	</tr>
	<tr>
		<th>...</th> <th>...</th> <th>...</th>
	</tr>
	<tr>
		<th>80</th> <th>78</th> <th>79</th>
	</tr>
	<tr>
		<th>80</th> <th>79</th> <th>78</th>
	</tr>
	<tr>
		<th>80</th> <th>80</th> <th>80</th>
	</tr>
</table>

I made all sorts of plots and eventually got some very interesting patterns that were somewhat fractal in appearance. But it didn't look like there was any obvious way to express these patterns I was finding mathematically.

Another approach I explored was to pull apart the index numbers themselves, even briefly attempting looking at things in binary and trying some truth tables and boolean logic. But ultimately the solution never needs to go further than base 3.

Lets examine a set from base 3.

    A = 0121	(16)
    B = 0221	(25)
    C = 0021	(7)

Here we see the rules of a valid set applied to each digit. If the corresponding digits in A and B are the same then it is also the same C, but if they are different then it is also different in C. Lets look at things from the perspective of a single base 3 digit.

<table class="tablev">
	<tr>
		<th>a</th> <th>b</th> <th>c</th>
	</tr>
	<tr>
		<th>0</th> <th>0</th> <th>0</th>
	</tr>
	<tr>
		<th>0</th> <th>1</th> <th>2</th>
	</tr>
	<tr>
		<th>0</th> <th>2</th> <th>1</th>
	</tr>
	<tr>
		<th>1</th> <th>0</th> <th>2</th>
	</tr>
	<tr>
		<th>1</th> <th>1</th> <th>1</th>
	</tr>
	<tr>
		<th>1</th> <th>2</th> <th>0</th>
	</tr>
	<tr>
		<th>2</th> <th>0</th> <th>1</th>
	</tr>
	<tr>
		<th>2</th> <th>1</th> <th>0</th>
	</tr>
	<tr>
		<th>2</th> <th>2</th> <th>2</th>
	</tr>
</table>

The question is how do we get c if only given a and b? This was one of those times where I tried boolean logic, but that didn't work out like I'd hoped.

Unfortunately I have to say I don't know of any specific process to arrive at the solution from this point. The solution came to me one night shortly after getting in bed. I can't even remember much about the train of thought that led to it. I've heard about people coming up with solutions to problems over night, apparently my brain was too eager to even wait for sleep. The solution was simple enough that I didn't even bother getting out of bed to write it down. Anyway, what I came up with is (3 - a - b) mod 3 = c, which is simplified to

    c = (-a - b) mod 3

It's important to use the proper modulus operator here, not the remainder operator that many programming languages use (python's % is proper modulus, which is nice). This is so that negatives are handled correctly.

    x       -5 -4 -3 -2 -1 0 1 2 3 4 5
    x mod 3  1  2  0  1  2 0 1 2 0 1 2

The equation does exactly what is needed. It returns the same number when a and b are the same, but a different number when a and b are different (assuming a, b, and c are base 3 of course).

    a = 1, b = 2
    c = (-1 - 2) mod 3
    c = -3 mod 3
    c = 0

    a = 1,  b = 1
    c = (-1 - 1) mod 3
    c = -2 mod 3
    c = 1

We now have everything we need to write a single, relatively simple, mathematical expression for identifying valid sets. Lets put it all together so we can operate on actual cards.

Step 3
======

The equation needs to be applied to each pair of corresponding digits in A and B to get the digits of C.

    A = 0121	(16)
    B = 0221	(25)
    C = 0021	(7)

    c3 = (-0 - 0) mod 3 = 0
    c2 = (-1 - 2) mod 3 = 0
    c1 = (-2 - 2) mod 3 = 2
    c0 = (-1 - 1) mod 3 = 1

The overall process that needs to happen is to extract each base 3 digit from A and B, apply the equation, then recombine each digit back to base 10 to get C.

Each digit is found by first dividing by the power of 3 corresponding to the digit, then applying modulus 3. It's important to use integer division or the floor function, which is represented by the symbols ⎣ and ⎦.

    A = 0121	(16)
    a3 = ⎣16 / 27⎦ = 0
    a2 = ⎣16 / 9⎦ = 1
    a1 = ⎣16 / 3⎦ = 5, 5 mod 3 = 2
    a0 = ⎣16 / 1⎦ = 16, 16 mod 3 = 1

Getting back to base 10 is done by multiplying each base 3 digit by the corresponding power of 3 and adding the results together.

    C = 0021	(7)
    0*27 + 0*9 + 2*3 + 2*1 = 7

Now it's just a matter of substitution to get the final desired function.

    C = f(A, B) = (-a3 - b3) mod 3 * 27 +
                  (-a2 - b2) mod 3 * 9 +
                  (-a1 - b1) mod 3 * 3 +
                  (-a0 - b0) mod 3

    C = f(A, B) = (-⎣A / 27⎦ mod 3  - ⎣B / 27⎦ mod 3) mod 3 * 27 +
                  (-⎣A / 9⎦ mod 3  - ⎣B / 9⎦ mod 3) mod 3 * 9 +
                  (-⎣A / 3⎦ mod 3  - ⎣B / 3⎦ mod 3) mod 3 * 3 +
                  (-A mod 3  - B mod 3) mod 3

    C = f(A, B) = (-(⎣A / 27⎦ + ⎣B / 27⎦) mod 3) * 27 +
                  (-(⎣A / 9⎦ + ⎣B / 9⎦) mod 3) * 9 +
                  (-(⎣A / 3⎦ + ⎣B / 3⎦) mod 3) * 3 +
                  (-(A + B) mod 3)

And we're done. I single expression to identify valid sets.

So, what was the motivation for this anyway?
============================================

First of all it just seemed like an interesting problem, even though I wasn't sure I was going to be able to find a solution, or if there even was one. But the thing that got me started on it was that I was trying to make a video game version of the game (see the project page [here](/projects/Set)). I was using a program called Game Develop which had it's own way of scripting events that didn't allow for the object oriented approach that one might typically use to identify cards and test for sets. Having a single mathematical expression made it so much easier. However I eventually abandoned Game Develop for Unity, and since I already had my solution I kept the same approach of using a single integer for representing a card since the solution is a bit simpler that way. There's not a single reference to any of the card attributes in any Unity script, because at this point it's redundant. Just a sorted array of card images indexed by the card's number.

Now, lets have some fun
=======================

I'm not aware of any way to simply the function any further, but it can be generalized to take any number of inputs and a base other than 3, then written as a python one-liner.

    def setFn(base, nums):
       return sum( sum(-(d / (base**i)) for d in nums) % base * (base**i) for i in range(base + 1))

The function f(A, B) we found above is the same as calling `setFn(3, [A, B])`. This function has a couple interesting properties. With any inputs, if you take the return value and replace one of the items in the nums list with it, then the new return value is equal to the item replaced. If instead you take the return value and just insert it into the list, then the new return value is always 0. This function identifies certain seemingly arbitrary sets of numbers, though it happens to include those of the card game. And if you plot this function where it's output equals 0 you get some fractal-like patterns. It's pretty nicely visualized with a shader, which can be seen [here](https://www.shadertoy.com/view/llVXDy).

I'm sure there's a lot more fun to be had with this type of function, such as using different operations other than `(-a - b) mod 3` on the digits of the new base. But that's an experiment for another day.
