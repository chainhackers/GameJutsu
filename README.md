# GameJutsu

ETHOnline 2022 entry by ChainHackers  
https://gamejutsu.app/

[Contracts](https://github.com/ChainHackers/gamejutsu-contracts)  
[Demo games](https://github.com/ChainHackers/gamejutsu-demos)  
[Subgraph](https://github.com/ChainHackers/gamejutsu-subgraph)  
[ETHOnline project page](https://ethglobal.com/showcase/gamejutsu-0u4en)  

### Who we are?

We are ChainHackers - adventurous developers and creatives who hack everything, i.e. try hands-on every technology
we like. One could call us blockchain generalists, though at the moment we're mostly exploring the Ethereum ecosystem.

### What we built

We built a framework to make on-chain game creation a lot easier. It makes use of

* state channels and highly formalized game rules definitions
* EIP-712 signatures for off-chain game state updates
* session keys to allow users skip sigining every move with their wallets

GameJutsu is partially based on the Magmo's [ForceMove whitepaper](https://magmo.com/force-move-games.pdf),
and it's aiming at being more developer-friendly and easier to use. ForceMove is a great protocol,
it's all science-like and strict - i.e. it's hard to make a mistake and use it in a wrong way, but also it is harder
to start using ForceMove for a less prepared programmer. GameJutsu is a bit more flexible and allows for more freedom in
client-to-arbiter communication, and presumably requires less sophisticated code in game definition.
Main differences between GameJutsu and ForceMove are:

* move order is determined by the game rules, not by the turn number
* game rules are defined by 2 main functions instead of the single `validTransition`
    * [isValidMove](https://github.com/ChainHackers/gamejutsu-contracts/blob/main/interfaces/IGameJutsuRules.sol#L26)
    * [transition](https://github.com/ChainHackers/gamejutsu-contracts/blob/main/interfaces/IGameJutsuRules.sol#L28)

In case there's only a `validTransition` function in the game rules interface,
dealing with ambiguity in game state transitions might be challenging.
Imagine a red king on a checkers board jumping a few times in sequence, and the progam
has to answer the question - is this transition valid? That requires answering the question
"how exactly did it get to its new position? - did it jump over just two piece, or more?"
Surely you don't want to run a BFS on-chain, even in checkers.

```
                  1       2       3       4
      1  01 │███│   │███│   │███│   │███│   │ 04 4
      5  05 │   │███│   │███│   │███│   │███│ 08 8
      9  09 │███│   │███│ o │███│ o │███│ o │ 0C 12
      13 0D │   │███│   │███│   │███│   │███│ 10 16
      17 11 │███│ o │███│ o │███│ o │███│   │ 14 20
      21 15 │   │███│   │███│   │███│ x │███│ 18 24
      25 19 │███│ o │███│   │███│ o │███│ x │ 1C 28
      29 1D │ x │███│ ♛ │███│ x │███│ x │███│ 20 32
             1D      1E      1F      20

How did we get here?
                  1       2       3       4
      1  01 │███│ . │███│   │███│   │███│   │ 04 4
      5  05 │   │███│   │███│   │███│   │███│ 08 8
      9  09 │███│   │███│   │███│   │███│ o │ 0C 12
      13 0D │   │███│ ♛ │███│   │███│   │███│ 10 16
      17 11 │███│   │███│   │███│   │███│   │ 14 20
      21 15 │   │███│   │███│   │███│ x │███│ 18 24
      25 19 │███│   │███│   │███│ o │███│ x │ 1C 28
      29 1D │ x │███│   │███│ x │███│ x │███│ 20 32
             1D      1E      1F      20

                  1       2       3       4
      1  01 │███│ . │███│   │███│   │███│   │ 04 4
      5  05 │   │███│   │███│ ↘ │███│   │███│ 08 8
      9  09 │███│   │███│ . │███│ . │███│ o │ 0C 12
      13 0D │   │███│ ♛ │███│   │███│ ↙ │███│ 10 16
      17 11 │███│ . │███│   │███│ . │███│   │ 14 20
      21 15 │ ↗ │███│   │███│ ↖ │███│ x │███│ 18 24
      25 19 │███│ . │███│   │███│ o │███│ x │ 1C 28
      29 1D │ x │███│ ↖ │███│ x │███│ x │███│ 20 32
             1D      1E      1F      20
```

The answer is - game state must include the move that led to it, so the game rules can follow the king's path
and reconstruct the details of the transition. GameJutsu team decided to make it explicit and include into the game
rules

#### Do we really need a state channel based protocol for on-chain games and why

Ethereum's community has been promised decentralized games since the year 2015, and is still expecting them to come.
CryptoKitties were great for their time and, for good or bad, most of today's on-chain games are not-that-remotely
similar to the legendary first widely-known NFT game. Some games are all about money -
either having a non-transparent yield-farming-style incentives at their core, others are outright centralized,
many do both.

So why aren't there many more games making use of all the amazing blockchain tech?

* Frankly speaking, coding on-chain game logic is hard - ChainHackers team just completed coding checkers on Solidity,
  and it was no trivial matter
* players don't feel like clicking buttons on a browser wallet pop-up every now and then, which is quite disruptive to
  game UX all by itself, paying for every click is a show-stopper for most
* the problem with sending blockchain transactions can be partially solved by collecting multiple player signatures and
  verifying them combinations in contracts later, but still that's a pop-up and a button click per move is far from
  smooth user experience, even when you don't have to pay for each one

GameJutsu skillfully attacks all 3 problems at once using state channels and session keys - just create a state channel
and publish your session key address in a single transaction and then you don't have neither to send a transaction per
move nor sign every single move with your wallet of choice. The game can progress as fast as game clients can exchange
signed messages between themselves and process those. With message size under 1KB and today's internet that's pretty
fast. Can you make a truly decentralized Dota-2 with that? Who knows, check out the 2 games ChainHackers made for you to
try during ETHOnline2022 - pure old-fashioned game experience with almost no extra clicks!
