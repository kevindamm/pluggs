# pluggs
*Programming Language for Unified Generic Game Service*

Based on the [Game Description Language Specification] and its extensions
[GDL-II] and [GDL-III], with a few adjustments (case-match patterns and added
semantics to structured types).  Includes:

* a parser (`pluggo`) for evaluating actions on a game state
* a code generator (`pluggen`) for building a basic service from
* a library (`pluggi`) for integrating typescript clients

In order to maintain the utility of already existing GDL game definitions,
the [plugg grammar] tries to remain backwards-compatible with the GDL syntax
such that a plugg service would be able to run the rules but nothing would be
persisted.  It also leans heavily on the flatbuffer schema definition language
and tries to remain backwards-compatible with .fbs definitions (deprecating
file-ID and file-ext keywords but parsing them, and adding keywords & attributes
that provide semantics for persisting).  An existing flatbuffer schema could be
parsed and its structures & rpc methods inferred, but it would not persist any
data or indices without a bit of work on the code generator's output.


**Note:** The above is almost entirely aspirational at this point, but these are
projects that I have either accomplished in other contexts (interpreter and code
generators) or have an itch I want to scratch (`wits-game` turn-based strategy
game with Fog of War that I have millions of real playthroughs for, in need of a
service and AI to really launch what I want it to be).


<!-- citations -->

[Game Description Language Specification]: https://www.ijcai.org/Proceedings/11/Papers/189.pdf
[GDL-II]: https://ojs.aaai.org/index.php/AAAI/article/view/7647/7508
[GDL-III]: https://www.ijcai.org/proceedings/2017/0177.pdf
