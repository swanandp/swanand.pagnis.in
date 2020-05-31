---
layout: post
title: "First Principles: Tic Tac Toe"
date: 2020-03-28 16:39:51 +0530
comments: true
categories: 
---

In this post, we'll look at building a simple Tic Tac Toe game playable by two players on the terminal.  We will do so by using "first principles."  A nifty technique for solving hard problems, learning something new, or deriving new insights. It often results in a significantly better design than otherwise.   

<!-- more -->

We start by defining the requirements, intended outcomes, or user stories. It does not have to be a polished list; we can always come back and edit it, as our solution and thought process matures.

So let's start with tic tac toe. It's a game. Games typically have setup, rules, and sequence of play. 

The setup:

- Two players
- Turn-based game
- A 3x3 grid.

There are going to be variations of these, like larger grids, but we're not designing the game right now, we're implementing it.

The rules:

- The first player to place three of their marks in horizontal, vertical, or diagonal row wins the game.
- At a time, any player can place only one mark.
- A player cannot replace marks put by themselves or other players. They can only place a mark on available squares.
- And so on. In the first principles approach, we focus on "foundational rules" or "invariants." What doesn't change, no matter the state of the play?


The sequence of play:

- When the program starts, it waits for players to join.
- When all required players join, it starts the game. 
- A player makes a move, and they receive the result. Say:
    - Invalid: they need to make another move
    - Valid: they need to await their turn
    - Game over.
    - Etc. 


So how do we begin writing code for something like this? There are various approaches, like top-down, where you build the outermost layer first, assuming inner layers exist and slowly go on building inner layers.  Bottom-up is the reverse; you make smaller functional layers first and then tie them all together.

A top-down approach, in this case, would be to start at the "game execution" level, assuming:
You have a state management layer, whose responsibility is maintaining and advancing the state
You have a player that is capable of making moves
You have utilities like printing the current state 

You can't pull this list out of thin air. More often than not, you need a pen-and-paper or whiteboarding session to think through.

A bottom-up approach, in this case, would be to pick the core inner layers and start building them, e.g., the state management layer here. It's the core piece that implements the game.

It's often a matter of taste when picking between these approaches. Keep in mind that a bottom-up approach often yields instant gratification. While the top-down approach often results in a better overall design.  _It depends_ disclaimer is applicable.



