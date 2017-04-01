based on the following research: http://zhangroup.aporc.org/images/files/Paper_3485.pdf

# Generating Puzzles

## Requirements of Puzzle Generation Strategy
- 1. ability to take a varying difficulty as an input
- 2. generate unique solution
- 3. minimize complexity (be fast & computationally efficient)


## Dig-hole Strategy
- Use a Las Vegas algorithm to generate a valid sudoku board
- generate puzzles by removing cells using the following five operators:
  - 1. determine sequence according to difficulty level
  - 2. set two restrictions to guide cell distribution
  - 3. use depth-first solver to determine whether puzzle has a unique solution
  - 4. add pruning techniques to avoid digging invalid cells
  - 5. perform propagation on the puzzle to raise diversity of output puzzle

Requires developing *3* algorithms:
- generating puzzle (las vegas + dig hole)
- grading difficulty of a puzzle
- solving puzzle (checks for 1 unique solution)


# Difficulty Algorithm

## Difficulty Metrics
- 1. amount of given cells
  - more empty cells === more difficult, direct correletion
- 2. the lower bound (?) of given cells per row/col
  - the positioning of empty cells within a row/col
  - clusters are harder, scattered are easier
- 3. applicable techniques by human logic thinking
  - various techniques used by humans to solve sudoku puzzles are rated by
    difficulty which correspond puzzle difficulty
  - if a puzzle is solvable by technique X, you can grade the puzzle with the difficulty score of technique X
- 4. enumerating search time by computer
  - basically measure brute force time of computer
  - time generally correlates to difficulty, higher time === harder difficulty


## Calculate Difficulty
- choose a scale (say 1-5)
- weight each metric that will be utilized
  - example uses (.4, .2, .2, .2) for above metrics
- calculate each individual score then combine into weighted average


# Solving Algorithm
- Used to judge whether there is a single unique solution to a given puzzle
- Using given values + following game rules, you can use enumeration search to generate all solutions

## Depth-first Algorithm
- search empty cells from left to right, top to bottom
- attempts to enter value 1-9 in empty cell while satisfying the three constraints (row, col, block)
- if no valid digit can be put in a cell (any value results in a collision of sorts), go back to previous cell and try the next value
- repeat process until all options are tried
- mark filled game boards as valid solutions
- once two solutions are found, the algorithm can stop? if all its doing is verifying unique solutions


# Generating Algorithm
Basic steps:
- generate terminal pattern
- dig holes to generate puzzle

## Las Vegas Algorithm
- used to generate random terminal pattern
- steps:
  - 1. start with empty grid
  - 2. randomly locate n cells in the grid
  - 3. fill cells with random values that satisfy game constraints
  - 4. attempt to solve puzzle with our solving algorithm
  - 5. while the puzzle is NOT solvable within a given time constraint (say 0.1s), go back to step 1
- paper suggests starting with 11 (n) givens to optimize computational time and puzzle diversity

## Digging Holes
- basic idea is erase numbers from terminal pattern until the following two conditions are met:
  - puzzle is solvable and has a unique solution
  - puzzle scores desired difficulty level
- different mechanisms of digging holes lead to diversity of puzzles in terms of difficulty and and puzzle pattern
- greedy mechanism
  - once a hole is dug, it stays dug
  - faster strategy

### Process of Digging Holes
- 1. determine difficulty
- 2. determine hole digging sequence
  - strategy for deciding what cell to dig next)
  - different strategies correspond to different difficulty levels
  - strategies:
    - left to right, top to bottom (5)
    - wandering S (4)
      - same as above, EXCEPT every other row goes right to left
    - jumping one cell ? (3)
    - randomizing globally ? (1 - 2)
- 3. determine restrictions
  - a trial of digging holes is said to be illegal if it violates any of the restrictions
  - adhering to these restrictions guarantee sufficient information is implied in puzzle for human logical deduction
  - restrictions:
    - 1. "randomize a bound value within the range of the total givens, and the remained cells must be more than that bound value" ??
    - 2. "the remained cells in each row and column must be more than the lower bound of givens in rows and columns" ??
- 4. determine what cells if any can be dug out
  - 4.1 get next cell to be dug out according to the sequence
  - 4.2 skip cell if it violates a restriction
  - 4.3 skip cell if it does not have a unique solution
    - utilize pruning optimization (see below)
  - 4.4 otherwise dig cell and repeat process
- 5. when no more cells can be dug, move on to propagation
- 6. output puzzle?


### Checking Solutions by Reduction to Absurdity
- enhancement to checking for solution uniqueness while digging
- summary: If I make this cell empty, and then I try enter a possible value other than the solution value in it, and it can result in a valid terminal state other than our solution, then emptying this cell results in multiple solutions for our puzzle
- while attempting to dig a particular cell..
  - 1. try every other valid value in that cell other than the given
  - 2. run the solver on this new board state
  - 3. once the solver finds a solution, determine that the dig is illegal because emptying this cell allows for multiple solutions
  - 4. if all 8 other numbers are tried and the solver finds no solution, then digging this cell does not create multiple solutions and is a feasible legal dig


### Pruning Optimization
- reduce running time of hole digging process by avoiding "digging paths" that are guaranteed to have multiple solutions
- basic idea: if digging out cell X results in multiple solutions, then digging out any more cells then X is also going to result in in multiple solutions, so leave X in place and move to next hole
- this means each cell will only be tried to be dug once (it's either dug, or left in place)
  - this further means that the "dig" loop executes a maximum of 81 times
- differs from backtrack idea
  - pruning makes it so we iterate over each cell once
  - backtracking would iterate over every possible diggable cell again after each dig?


### Propagation
- four types of propagation can be applied to a valid terminal state to change the puzzle pattern while obeying game rules
- 1. mutual exchange of two digits
- 2. mutual exchange of two columns in the same block column (cols 0 and 2 can be exchanged, but not 0 and 3 or 2 and 3)
  - same as mutual exchange of two rows in the same block row
- 3. mutual exchange of two columns of blocks (basically exchange cols 0,1,2 with 6,7,8)
  - same as mutual exchange of two rows of blocks
- 4. grid rolling
  - rotate or flip the grid any number of times


# Analysis

## Strengths
- extensible algorithm for generating puzzle according to difficulty
- utilizes different sequences of digging holes to adapt to the different characteristics of difficulty
- pruning technique minimize time spent digging holes
- las vegas algorithm + propagation techniques guarantee diversity in puzzles

## Weaknesses
- does not generate puzzles able simulate all existing techniques for solving sudoku puzzles by human logic
- can only generate "evil" difficulty puzzles with 22 givens (as opposed to the minimal 17)
