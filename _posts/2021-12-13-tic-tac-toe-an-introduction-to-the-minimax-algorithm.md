---
layout: post
title:  "Tic-Tac-Toe: An introduction to the minimac algorithm"
date:   2021-12-13 14:33:21 +0530
categories: jekyll update game-theory
---

Tic-Tac-Toe, though a simple game to play between two friends, can offer a lot of insight into adversial search, and the widely used minimax algorithm.

# Parts:
Today we will build a complete Tic-Tac-Toe game complete with a bot to play against users. This game will contain several interconnected components:
 - Game framework
 - Game UI
 - Minimax based bot

# Game framewok and UI
Tic-Tac-Toe is a simple game to implement. We start of with an empty 3×3 game board. Each turn we allow the user (or bot) to make a move on an empty square. After completing the move, we check for any consecutive rows, columns or diagonals with the same token. Finally, we update the set of empty cells in the board, and flip the turn state.

This is what the code for the game looks like:
```python
```

To ensure our game doesn't look like it came straight from the 80s, we can create a basic user interface for the game. The pygame package makes it very simple for us to create an interface that allows users to interact with the game.

# The minimax algorithm
The minimax algorithm is one of the simplest adversial search decision. Adversial search algorithms involve optimizing a certain heuristic score against an "adversary", the human player in this case. The minimax algorithm is used to minimize the potential loss in the worst case. The basic psuedocode for the minimax is extremely simple:
```
function minimax(ganeState, maximizingPlayer):
	if terminal node:
		return heuristic(gameState)
	
	if maximizingPlayer:
		res = -INF
	else:
		res = INF
	
	for nextState in possibleMoves(gameState):
		if maximizingPlayer:
			res = max(minimax(nextState), false)
		else:
			res = min(minimax(nextState), false)
	
	return res
```

The first thing we notice about the minimax algorithm is that it is a recursive algorithm. We start at the current game state, and recursively traverse down the game tree to the terminal nodes, which mark the end of the game. Each terminal node has a heuristic score, which the bot (the maximizing player) tries to increase, and the human opponent tries to decrease. 

In our game, the heuristic is very basic. Since we are only calculating the heuristic score at a terminal node, we can give 1 for a winning board, 0 for a tie, and -1 for a losing board.

For the sake of comparison, let us add 2 more bots, one that randomly selects an empty cell, and another that plays the immediate best move (greedy bot). The greedy bot 
