# pluggs
*Programming Language for Unified Generic Game Service*

Based on the [Game Description Language Specification] and its extensions
[GDL-II] and [GDL-III], with a few adjustments (case-match patterns and added
semantics to structured types).  Includes:

* a parser & interpreter (`pluggo`) for evaluating actions on a game state
* a code generator (`pluggen`) to build a basic service from a game definition
* a library (`pluggi`) for securely connecting typescript clients to the service

In order to maintain the utility of already existing GDL game definitions,
the [plugg grammar] tries to remain backwards-compatible with the GDL syntax
such that a plugg service would be able to run the rules while nothing would be
persisted.  It also leans heavily on the flatbuffer schema definition language
and tries to remain backwards-compatible with [.fbs schema language] deprecating
file ID and extension but still parsing them, and adding keywords & attributes
that provide semantics for persisting).  An existing flatbuffer schema could be
parsed and its structures & rpc methods inferred, but it would not persist any
data or indices without a bit of work on the code generator's output.

((TODO: image representing the components and how they interact))

The goal is to have one definition of a game in GDL and be able to generate the
backend (for persisting games and game history), the service interface (with
automatically generated RPC endpoints, extended by explicitly defined endpoints)
and the client interface (for joining, updating and reviewing games).  While GDL
and datalog-like languages aren't the most approachable, it is succinct and very
declarative, well suited for defining game rules.  While .fbs isn't widely used,
it is very broadly compatible with many programming languages and well suited to
precision in data layouts and index relations.



**Note:** The above is almost entirely aspirational at this point, but these are
projects that I have either accomplished in other contexts (interpreter and code
generators) or have an itch I want to scratch (a [wits-game] turn-based strategy
game with Fog of War that I have millions of real playthroughs for, in need of a
service and AI to really launch what I want it to become).  Perhaps through some
real-world examples and creative presentations of the definitions and conditions
of familiar games the task of building 


<!-- citations -->

[Game Description Language Specification]: https://www.ijcai.org/Proceedings/11/Papers/189.pdf
[GDL-II]: https://ojs.aaai.org/index.php/AAAI/article/view/7647/7508
[GDL-III]: https://www.ijcai.org/proceedings/2017/0177.pdf
[.fbs definitions]
