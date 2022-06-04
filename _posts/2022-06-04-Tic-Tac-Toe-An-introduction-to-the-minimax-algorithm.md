---
layout: post
title:  "Tic-Tac-Toe: An introduction to the minimax algorithm"
date:   2022-06-04 00:00:00 +0530
categories: jekyll update game-theory
---

Tic-Tac-Toe, though a simple game to play between two friends, can offer a lot of insight into adversial search and the widely used minimax algorithm.

# Parts:
Today we will build a complete Tic-Tac-Toe game complete with a bot to play against users. This game will contain several interconnected components:
 - Game framework
 - Game UI
 - Minimax based bot

# Game framewok and UI
Tic-Tac-Toe is a simple game to implement. We start of with an empty 3×3 game board. Each turn we allow the user (or bot) to make a move on an empty square. After completing the move, we check for any consecutive rows, columns or diagonals with the same token. Finally, we update the set of empty cells in the board, and flip the turn state.

This is what the code for the game looks like:
```python
# Import packages
from Bot import *
from json import load, dumps

class Game:
	'''This class describes the TicTacToe game

	Attributes
	----------
	botModes : Dict[str, func]
	width : int
		The board width
	height : int
		The board height
	board : List[List[bool]]
		The game board
	turn : bool
		Whether it is player1's turn
	gameOver : bool
		Whether the game is over
	remainingCells : Set[int]
		The set of cells that are still empty

	Methods
	-------
	__init__() -> None
		Initializes the Game instance
	makeMove(Tuple[int, int]) -> bool
		Makes a move on the game board and returns if it is successful
	checkWin(Tuple[int, int]) -> bool
		Checks if the game is over
	'''

	botModes = {
		'e': easyBot,
		'm': mediumBot,
		'h': hardBot
	}

	def __init__(self, width, height):
		'''Initializes the Game instance

		Parameters
		----------
		width : int
			The board width
		height : int
			The board height
		'''

		# Initialize values
		self.width = width
		self.height = height
		self.turn = True
		self.gameOver = False

		# Create game board and remainingCells set
		self.board = []
		self.remainingCells = set()
		for x in range(self.width):
			self.board.append([])
			for y in range(self.height):
				self.board[x].append(None)
				self.remainingCells.add((x, y))
	
	def makeMove(self, cell):
		'''Makes a move on the game board and returns if it is successful

		Parameters
		----------
		cell : Tuple[int, int]
			The cell that we want to make a move on
		
		Returns
		-------
		bool
			Whether the move was successful
		'''

		# Check if cell is empty
		if cell not in self.remainingCells:
			return False
		
		# Update the game board
		self.board[cell[1]][cell[0]] = self.turn
		self.remainingCells.remove(cell)

		# Check for a win
		self.gameOver = self.checkWin(cell)

		# Swap turn and return True
		self.turn = not self.turn
		
		return True

	def checkWin(self, lastMove):
		'''Check if the game is over

		Parameters
		----------
		lastMove : Tuple[int, int]
			The last move
		
		Returns
		-------
		bool
			Whether the game is over
		'''

		# Get individual coordinates
		x, y = lastMove

		# Check the row of the cell
		for i in range(1, self.width):
			if self.board[y][0] != self.board[y][i]:
				break
		else:
			return True
		
		# Check the column of the cell
		for i in range(1, self.height):
			if self.board[0][x] != self.board[i][x]:
				break
		else:
			return True
		
		# Check the diagonals if the cell is in them
		# Only applies to squares
		if self.width == self.height and x == y:
			for i in range(1, self.width):
				if self.board[0][0] != self.board[i][i]:
					break
			else:
				return True
		if self.width == self.height and (self.width - x - 1) == y:
			for i in range(1, self.width):
				if self.board[0][self.width - 1] != self.board[i][self.width - i - 1]:
					break
			else:
				return True

		# There are no matches
		return False

if __name__ == '__main__':
	# Runs if this file is run directly
	displayMap = {
		None: '_',
		False: 'O',
		True: 'X'
	}

	# Run as long as the user wants to play a game
	while True:
		# Take input parameters
		botEnabled = input('Single Player / Double Player - S/D: ').lower() == 's'
		if botEnabled:
			mode = input('Enter your difficulty mode - E/M/H: ')
			bot = Game.botModes[mode.lower()]
			names = [
				input('Enter the Player\'s name: '),
				'the computer'
			]
			player = int(input('Enter which player you want to be: '))
		else:
			names = [
				input('Enter Player 1\'s name: '),
				input('Enter Player 2\'s name: ')
			]
			player = 1
		width, height = map(int, input('Enter board dimensions: ').split())
		game = Game(width, height)
		print()
		
		# Run the game
		while not game.gameOver and len(game.remainingCells) > 0:
			print(f'It is {names[not game.turn]}\'s turn')

			if (not botEnabled) or 1 + (not game.turn) == player:
				# It is the players turn
				# Get move input
				x, y = map(int, input('Enter cell: ').split())
				if not game.makeMove((x - 1, y - 1)):
					print('Not a valid move. Try again')
					continue
				
				# Display the board
				print('Board:')
				for row in game.board:
					print(*map(lambda x: displayMap[x], row))
				print()
			else:
				# It is the bots turn
				move = bot(game)
				game.makeMove(move)

				# Display the board
				print('Board:')
				for row in game.board:
					print(*map(lambda x: displayMap[x], row))
				print()

		# Display the result
		if game.gameOver:
			if botEnabled and 1 + game.turn != player:
				print(f'The computer won!')
			else:
				print(f'{names[game.turn]} won!')

			# get player scores
			if True:
				scoresDict = {}
				try:
					with open('scores.json', 'r') as scoresFile:
						scoresDict = load(scoresFile)
				except:
					# File has not been created
					pass
	
				# add players to scoresDict if not already present
				if names[game.turn] not in scoresDict:
					scoresDict[names[game.turn]] = {
						'win': 0,
						'lose': 0,
						'draw': 0
					}
				if names[not game.turn] not in scoresDict:
					scoresDict[names[not game.turn]] = {
						'win': 0,
						'lose': 0,
						'draw': 0
					}

				# update scores in scores.json
				if game.gameOver:
					scoresDict[names[game.turn]]['win'] += 1
					scoresDict[names[not game.turn]]['lose'] += 1
				else:
					scoresDict[names[game.turn]]['draw'] += 1
					scoresDict[names[not game.turn]]['draw'] += 1

				scoresDict.pop('the computer', None)

				# save results to scores.json
				with open('scores.json', 'w') as scoresFile:
					scoresFile.write(dumps(scoresDict))
		else:
			print('It is a draw')
		print('Thank you for playing!')

		# Check if the user wants to play again		
		if not input('Press <Enter> to exit and any other key to play again: '):
			break
		print()
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

For the sake of comparison, let us add 2 more bots, one that randomly selects an empty cell, and another that selects the immediate best move (greedy bot). The greedy bot first checks if it can complete a row, column or diagonal. If no such move is possible, the bot tries to block the opponent from completing 3 in a row. Finally, if there are no such threats, if reverts to the standard strategy of selecting a random empty cell to make a move.

Here's what the combined code of the 3 bots look like:
```python
'''This file contains the functions for different bots:
 - Easy mode
 - Medium mode
 - Hard mode

Methods
-------
easyBot(board) -> Tuple[int, int]
	Returns the move for the easy mode bot
mediumBot(board) -> Tuple[int, int]
	Returns the move for the medium bot
hardBot(board) -> Tuple[int, int]
	Returns the move for the hard bot
'''

# Import required packages
from random import choice
from copy import deepcopy

def easyBot(game):
	'''Returns the move for the easy mode bot

	This method randomly chooses an empty cell

	Parameters
	----------
	game : Game
		The current game board
	
	Returns
	-------
	Tuple[int, int]
		The next move of the easy bot
	'''

	# Select a random cell from game.remainingCells
	return choice(list(game.remainingCells))

def mediumBot(game):
	'''Returns the move for the medium mode bot

	This bot selects any immediate possibility of a win.
	If this is not possible, it tries to block any immediate possibility of a loss.
	Otherwise, it selects a random cell

	Parameters
	----------
	game : Game
		The current game board
	
	Returns
	-------
	Tuple[int, int]
		The next move of the medium bot
	'''

	# Check if any cell will provide an immediate win
	emptyCells = list(game.remainingCells)
	for x, y in emptyCells:
		# Try selecting the cell
		game.makeMove((x, y))
		
		# Revert to previous turn
		game.board[y][x] = None
		game.remainingCells.add((x, y))
		game.turn = not game.turn

		# Check if this cell is an immediate win
		if game.gameOver:
			game.gameOver = False
			return (x, y)

	# Check if any cell will block an immediate opponent win
	if len(emptyCells) > 1:
		game.turn = not game.turn
		for x, y in emptyCells:
			# Try selecting the cell
			game.makeMove((x, y))
			
			# Revert to previous turn
			game.board[y][x] = None
			game.remainingCells.add((x, y))
			game.turn = not game.turn

			# Check if this cell blocks anything
			if game.gameOver:
				game.gameOver = False
				game.turn = not game.turn
				return (x, y)
		game.turn = not game.turn
	
	# Select a random cell from game.remainingCells
	return choice(emptyCells)


def hardBot(game):
	'''Returns the move for the hard mode bot

	This method implements the minimax algorithm

	Parameters
	----------
	game : Game
		The current game board
	
	Returns
	-------
	Tuple[int, int]
		The next move of the hard bot
	'''
	
	# Initialize variables
	emptyCells = list(game.remainingCells)
	player = game.turn
	res = -1
	bestMove = emptyCells[0]

	# Iterate throught moves and find the best one
	for x, y in emptyCells:
		# Try selecting the cell
		game.board[y][x] = player
		game.remainingCells.remove((x, y))

		# Get the best move
		curr = hardBotUtil(player, not player, game)
		if curr > res:
			res = curr
			bestMove = (x, y)
		
		# Revert to previous turn
		game.board[y][x] = None
		game.remainingCells.add((x, y))
	
	# Returns the best move
	return bestMove


def hardBotUtil(player, currPlayer, game):
	'''Implements the minimax algorithm

	Parameters
	----------
	player : bool
		The initial player
	game : Game
		The current game state
	
	Returns
	-------
	int
		The score
	'''

	# Calculate score
	score = evaluateBoard(player, game.board, game.width, game.height)

	# Check for whether the game is over
	if score != 0 or len(game.remainingCells) == 0:
		return score
	
	# Recursively travel through game tree
	res = -1 if currPlayer == player else 1
	emptyCells = list(game.remainingCells)
	for x, y in emptyCells:
		# Try selecting the cell
		game.board[y][x] = currPlayer
		game.remainingCells.remove((x, y))

		# Get the best move
		curr = hardBotUtil(player, not currPlayer, game)
		if currPlayer == player:
			res = max(res, curr)
		else:
			res = min(res, curr)
		
		# Revert to previous turn
		game.board[y][x] = None
		game.remainingCells.add((x, y))
	
	# Return the score
	return res

def evaluateBoard(player, board, width, height):
	'''Returns the score of the board

	-1: Losing state
	0: Draw state
	1: Winning state

	Parameters
	----------
	player : bool
		The initial player
	board : List[List[int]]
		The current game board
	width : int
		The width of the game board
	height : int
		The height of the game board
	
	Returns
	-------
	int
		The score
	'''
	
	# Check rows
	for x in range(width):
		if board[0][x] == None:
			continue

		for y in range(1, height):
			if board[y][x] != board[0][x]:
				break
		else:
			if board[0][x] == player:
				return 1
			else:
				return -1
	
	# Check columns
	for y in range(height):
		if board[y][0] == None:
			continue

		for x in range(1, width):
			if board[y][x] != board[y][0]:
				break
		else:
			if board[y][0] == player:
				return 1
			else:
				return -1
	
	# Check diagonals if applicable
	if width != height:
		return 0
	
	# Top-left to Bottom-right
	for j in range(1, width):
		if board[j][j] != board[0][0]:
			break
	else:
		if board[0][0] == player:
			return 1
		else:
			return -1
	
	# Top-right to Bottom-left
	for j in range(1, width):
		if board[j][width - j - 1] != board[0][width - 1]:
			break
	else:
		if board[0][width - 1] == player:
			return 1
		else:
			return -1
	
	# It is a draw situation
	return 0
```

# The finished product:
After integrating all the components, here's what the completed game looks like:
![]()

Here's a comparison of the 3 bots:

To find the time complexity of our minimax bot, we need to look at the total number of moves possible in the worst case, i.e. an empty board. In the first move there are 9 possible moves. In the second move there are 8 possiblities since one of cells has already been occupied. This continues till all the cells have been occupied. In total, there are 9! possible sets of moves that can take place in a Tic-Tac-Toe game (approximately 360,000).As a rule of thumb, the average computer can compute 1e8 O(1) operations in a second. Since 9! is comfortably within that limit, there is no need for any additional optimization.

The minimax  decision rule is a simple but powerful algorithm, that can be extended to  many two player games like chess. In order to use the minimax algorithm on such complex games, we can utilize additional optimization techniques like alpha-beta pruning, as well as develop more comprehensive heuristic functions.

Check out the project on [Github](https://github.com/SiddhantAttavar/Tic-Tac-Terminator/).
