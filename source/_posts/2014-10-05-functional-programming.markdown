---
layout: post
title: "Functional Programming"
date: 2014-10-05 18:21
comments: true
categories: functional programming mind
---

From time to time I try to look on direction which are still now very clear to me. Trying to find interesting topics and discover something new. At the moment it's functional programming.

So far, I would not say functional programming is somehow hard. It's more likely, it's very unusual for ones who spend a lot time with object oriented programming. My knowledge is still very low, but I'm trying to figure out, do I like it or not? Would I have some real gains if switch to functional language?

<!-- MORE -->

I would like to formulate what I think the functional programming is, because I see many interpretation of the topic. For me, functional programming is style which embraces data immutability and purity of function calls (one which is know as absence of side effects). Sounds not so clear, right? But, let's try to think about.

## Immutability of data

Thats both the feauture of language and fundamental concept that all data application operates is immutable. For OOP guys, as myself, it means - once object is instantiated and initialized, it could not be changed any longer. But how would you deal with variables? Well, the variables still exists but not in the way we get used to. Also, the changes of state is still possible, but original object is not destoroyed. That means, that application is able to keep all it's state changes for a time of being.

That's very interesting concept, could be explained with a little metaphor: images funtional program as a stack of papers, each paper you draw a character. As you go throught the papers very quickly, you observe a kind of "movie" - character moves. In object-oriented program what you have is: one piece of paper, pencil and erase tool. To change the state, you destory previous and create new one.

This is very related to event-sourcing paradigm, which represents current state of system as function of it's all previous states, or `s -> f (e)` (state change is the function of event).

## Pure functional calls

Functional programming deeps it's roots to math. As function there is abstraction that takes some inputs and produces the output. The important thing - nothing else as output production happens. Moreover, what every many calls you do function with same arguments, as many times it produces just the same results.

They call function calls "pure", because it's only the call and nothing else happens. That "else" - is what know as side-effects. Side effect is the implicit change of state, which is typically source of problems in object-oriented programs.

## Code as data

TDB.

## Benefits

## First impressions

## Learning


