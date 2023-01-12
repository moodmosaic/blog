---
layout: basic
title: The rhyme of the altered Parser
---

Fritz Ruehr's rhyme gives us the essence of parsers:

>*A Parser for Things
is a function from Strings
to Lists of Pairs
of Things and Strings!*

This rhyme was made popular, I believe, in [Programming in Haskell](https://www.amazon.com/dp/1316626229/) book by Graham Hutton.

## Altered parsers

I may be coming back later on to add examples and resources on parsers, but now I'd like to mention [Vaibhav Sagar](https://vaibhavsagar.com/)'s post on [Revisiting 'Monadic Parsing in Haskell'](https://vaibhavsagar.com/blog/2018/02/04/revisiting-monadic-parsing-haskell/).

## Altered rhymes

Once you read Vaibhav Sagar's post, followed by the [HN discussion](https://news.ycombinator.com/item?id=16302014), you'll notice two new rhymes coming out: the rhyme of the *altered* parser, and the rhyme of the *yoctoparsec*.

The rhyme of the altered Parser:
>*A parser for things is a function from strings to potentially a pair of that thing and its string.*

The rhyme of the yoctoparsec:
>*A parser just means A function from streams To maybe a pair, One in which there Is the stream, and the thing that it means.*

I might be trying out [yoctoparsec](https://hackage.haskell.org/package/yoctoparsec). The only place I've seen it being used is in [Eric Mertens](https://github.com/glguy)'s [Day 8 solution](https://github.com/glguy/advent2018/blob/c298bceec795733d4ebc0cbeb782e0c4b1bd4c88/execs/Day08.hs) of a past Advent of Code.
