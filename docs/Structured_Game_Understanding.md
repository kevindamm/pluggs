Structured Game Understanding
===

Building on prior research from Stanford's GDL and Ai Ai's MCTS, plugg's model
of games is based on a collection of rules and operations, goals and legal game
states.  This is sufficient for a generic game player to develop a strategy for
playing the game and it offers a handle (and naming) for attaching UI/creative
assets, presenting the game in a consistent way and persisting metrics for
global scoreboards and gaemplay analytics.

Both an AI player and code generation framework benefit from a Game Model as

   1. generating allowed moves from a valid game state
   2. how to apply 1..n moves to get subsequent game states
   3. identifying if the current game state is terminal
   4. evaluating the "score" or game's result at a game state

### Modularity

The improvement being proposed is a type system for value objects in GDL and a
few additional reserved words for plugging data sources and data sinks into the
game model.  The result is called *[pluggs](github.com/kevindamm/pluggs)* and
aims to provide a suite of tools based on interpreting a plugg GameModel.

### Uniformity

While these are usually explicitly coded into the game logic of the cient and
server, it is often tediously repetitious and can lead to errors if the logic
in the client does not accurately match what is on the server or if the
server overlooks an edge case of the rules in its input validation.  It also
distracts from the development of the game rules and inhibits the exploration
of game rules when changes may easily break things.  The confidence in having
a client and server (and other) components be immediately compatible encourages
more frequent updates to the rules and evaluating their effects.

### Accessibility

By having an interface developed on the basis of the inputs and actions for
the game, there is a lot of potential in making the game itself accessible,
via screen readers and inserting of game rule explanation into the control
elements themselves.  Big TODO to research this more.

This applies also to the presentation of the rule definitions for those who
are learning it for the first time or want to develop their own variant.  The
syntax in this document is approachable for those familiar with datalog or GDL
but it is unusual even for those used to more procedural

### Adaptability

By building game rule and game model understanding into the game generator, it
would also be possible to run that game model understanding within the game
itself.  Without overburdening the developers on handling which game rules
have been modified at each point where they may come into play, the game model
itself is updated to change the balance of the game.  This could lead to an
interesting new style of game, meta-strategies and demanding more agility on
the roles of computer players.  We may soon need an upper hand against these
adversaries.

# Syntax

A brief example of a complete definition of tic-tac-toe in plugg:

```
// A two player game.
role xplayer, oplayer

data Marker = blank | x | o

Cell :: int -> int -> Marker

init {
	// Initialize a play grid with rows and columns,
	// define a 2-dimensional cell type.
	board.cell = Cell
	for i in int[1..3]:
		for j in int[1..3]:
			board.cell(i, j) = blank

	// Start play with xplayer and switch control each turn.
	board.control = xplayer
	board.open :: bool
}

next(board.control = oplayer)) :- board.control == xplayer
next(board.control = xplayer)) :- board.control == oplayer

// Marking a cell is the primary action each player has.
Mark :: int -> int -> Action
NOOP :: Action

// A player can mark a cell if it is blank and the player is in control
legal{?player: Mark(?i, ?j)} :-
    board.cell(?i, ?j) == blank and control == ?player

// The only legal move for a player not in control is a no-op.
legal{?player: NOOP} :- distinct(?player, board.control)

// A legal marking results in the cell being correctly updated next turn.
next{board.cell(?i, ?j) = x} :-
  xplayer.does(Mark(?i, ?j)) and board.cell(?i, ?j) == blank)
next{board.cell(?i, ?j) = o} :-
	oplayer.does(Mark(?i, ?j)) and board.cell(?i, ?j) == blank)

// A cell that has been marked will remain marked.
next{board.cell(?i, ?j) = ?mark} :-
	board.cell(?i, ?j) == ?mark and distinct(?mark, blank)

// A cell will remain blank if it is not the one being marked.
next{board.cell(?i, ?j) = blank} :- (
	board.cell(?i, ?j) == blank and
	board.control.does(Mark(?m, ?n)) and
	(distinct(?i, ?m) or distinct(?j, ?n))
)


// Evaluating the winning conditions

row(?i, ?mark) :- (
	board.cell(?i, 1) == ?mark and
  board.cell(?i, 2) == ?mark and
  board.cell(?i, 3) == ?mark
)

column(?j, ?mark) :- (
	board.cell(1, ?j) == ?mark and
  board.cell(2, ?j) == ?mark and
  board.cell(3, ?j) == ?mark
)

diagonal(?mark) :- (
  board.cell(1, 1) == ?mark and
	board.cell(2, 2) == ?mark and
  board.cell(3, 3) == ?mark
)

diagonal(?mark) :- (
	board.cell(1, 3) == ?mark and
	board.cell(2, 2) == ?mark and
	board.cell(3, 1) == ?mark
)

line(?p) :- row(?k, ?p)
line(?p) :- column(?k, ?p)
line(?p) :- diagonal(?p)

board.open :- board.cell(?i, ?j) == blank

terminal :- line(x)
terminal :- line(o)
terminal :- ~ board.open

// Goal evaluation

xplayer.goal(100) :- line(x)
xplayer.goal(50) :- ~ line(x) and ~ line(o) and ~ board.open
xplayer.goal(0) :- line(o)

oplayer.goal(100) :- line(o)
oplayer.goal(50) :- ~ line(o) and ~ line(x) and ~ board.open
oplayer.goal(0) :- line(x)
```

You can see that it retains the symbolic and declarative aspects of the language
that inspired it while also aiming to be more readable than KIF, without overly
complicating the syntax and operational semantics of the language.

Some translation from GDL can be automatic, and for the sake of including many
already developed game definitions it would be wise to aim for as direct a
translation as possible while designing the plugg language, or at least have
the direct translation be compatible, even if additional structure and the
substitution of basic arithmetic and logic operators could make the code more
readable than a direct translation would.

## Reserved keywords and built-in operations

These keywords were already reserved in the original GDL design
and continue to retain their meaning:

#### `role`

Defines the symbol(s) representing the players of the game.  There must be
at least one, and gameplay expects a move from every player each turn (though
in some games the only legal move is a no-op if play transfers control between
players).

#### `init`

Defines an initialization statement for facts in play.

#### `legal`

As a head of an inference it asserts that a (Player, Action) pair is a legal move
if the prerequisites are satisfied in the current game state.

#### `next`

As a head of an inference it asserts the values in the next state of the game,
if the prerequisites are satisfied in the current game state.

#### `true` [deprecated]

In GDL it evaluates the following expression.  This is redundant when plugg
already has a type-consistent notion of boolean values and operations.  May
still be used as a boolean constant.

#### `distinct`

Compares two symbols for being independent or (value) distinct from each other.

#### `does`

A boolean-compatible value for whether a player takes the indicated action this
turn, now a property of Player.

#### `goal`

Scoring specifier, now a property of Player.

#### `terminal`

Indicates a terminating state of the game, the game-end conditions.


## Assertions of fact and inference


## Structured types and properties


## Template matching random variables

As with the datalog-inspired GDL, statements in the rules definition can have
random variables, as defined in the domain of Probability.  These will match
against any available assertions in the current game state.

## Asymmetric knowledge and indeterminacy

In some games the players do not have perfect knowledge or there are elements
of chance that also lead to imperfect knowledge.  The language adopts the
extensions from GDL-II with the keywords `seen` and `random` (distinct from
the random variable mapping provided by the '?' annotation mentioned in the
previous section).

## Similarities to GDL and advantages

The syntax follows that of GDL in rule semantics but differs by using infix
notation rather than the usual prefix notation found in KIF-style rules.

Rather than the less familiar style of code like KIF:

```        
  (role red)
  (role blue)
  (<= (legal blue noop) (true (control red)))
  (<= (legal red noop) (true (control blue)))
```

the plugg style uses a syntax closer to that of datalog:

```
  role red, blue
  legal(blue, noop) :- red has control
  legal(red, noop) :- blue has control
```

The clarity is more apparent in larger rules like the following GDL

```
  (<= (legal ?player (move ?x1 ?y1 ?x2 ?y2))
      (true (control ?player))
      (true (location ?x1 ?y1 ?piece))
      (owns ?piece ?player)
      (validmove ?piece ?x1 ?y1 ?x2 ?y2)
      (not (celloccupiedby ?x2 ?y2 ?player)))
```

which says that a move from position (x1, y1) to (x2, y2) is legal when the
player has control, the piece at (x1, y1) is owned by the player, and the
player doesn't currently occupy (x2, y2) with a different piece.  (Note, this
is like chess rules but would be different in games where you must jump over
or avoid opponent pieces, or are restricted to only moving into unoccupied
positions, but these could be adjusted with just the one clause here).

In plugg it would look more like:

```
  legal(?player, board.translate((?i ?j), (?i' ?j'))) :- all {
      ?player has control;
      ?player has ?piece;
      ?piece.location == (?i, ?j);
      ?piece.valid_move((?i, ?j), (?i', ?j'));
      distinct(board.at(?i', ?j').player, ?player);
  }
```

Negative statements are still restricted to appearing in rules with a positive
head that include all the random variables in the statement.  The reserved words
are the same and retain the same arity as they had in GDL.  It becomes clearer
the semantics of expressions when they appear at the head of an inference, such
as legal(player, move) and next(expr).


