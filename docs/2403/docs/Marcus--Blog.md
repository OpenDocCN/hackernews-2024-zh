<!--yml
category: 未分类
date: 2024-05-27 14:59:02
-->

# Marcus' Blog

> 来源：[https://mbuffett.com/posts/compressing-chess-moves/](https://mbuffett.com/posts/compressing-chess-moves/)

UPDATE: I’ve written a [new post](/posts/compressing-chess-moves-even-further/) with a new technique which is about 2.5x more efficient than this approach

Chess notation has come a long way since [descriptive notation](https://en.wikipedia.org/wiki/Descriptive_notation), now we have nice and decipherable Standard Algebraic Notation, like `Qxf7` (queen takes on f7) or `Nf3` (knight takes on f3).

This is a great text format, but a massive waste of space if you’re trying to store a lot of these. `Qxf7` takes 4 bytes, or 32 bits. Let’s do some rough back-of-the-envelope math of how much information is actually being transmitted though. This move affects one of 6 pieces (3 bits), this move is also a capture (1 bit), and it specifies a destination squuare (64 possibilities == 6 bits). Add those up, you get 10 bits. Far from the 32 bits that the textual representation needs.

Why do I care? I run a site that stores a *ton* of chess lines, something like 100 million in total. Assume an average of 6 moves for each line, that’s 600 million moves. The database is growing large enough that querying it is IO-constrained. I want to speed up the reads from this database when I’m fetching thousands of lines.

## A first pass

First some general numbers:

*   Encoding a file or rank (a-h or 1-8) takes 3 bits (8 possibilities)
*   Encoding a piece (k, q, r, b, n, p) takes 3 bits (6 possibilities)
*   Encoding a square takes 6 bits (64 possibilities)

So let’s see the pieces we’re working with for encoding a SAN.

So let’s go with the most naive approach.

*   Which piece was it? 3 bits
*   Is it a capture? 1 bit
*   Do we have to disabiguate it (ie `Ngf3`)? Maximum of 2 bits + 6 bits (this is explained more further down)
*   Where did it go? 6 bits
*   Is it a promotion, and to which piece? 7 bits
*   Is it a check? 1 bit
*   Is it a checkmate? 1 bit
*   Is it a castle? Short or long? 2 bits

This gives us a total of `3+1+2+6+6+7+1+1+2 = 29` bits, or about 3.5 bytes per move. That’s not great though. A lot of moves actually take up less bits than that in text format.

## Getting smarter

### The first few bits

In the first pass, we just encoded the piece that moved using 3 bits, but that leaves 2 unused permutations available, since there are only 6 pieces in chess. Luckily, there are two very common moves that fit neatly into this hole, those being short castles (`O-O`) and long castles (`O-O-O`).

So we can encode the first 3 bits like so:

```
match first_bytes {  FirstBytes::Pawn => bits.extend(&[false, false, false]), FirstBytes::Knight => bits.extend(&[false, false, true]), FirstBytes::Bishop => bits.extend(&[false, true, false]), FirstBytes::Rook => bits.extend(&[false, true, true]), FirstBytes::Queen => bits.extend(&[true, false, false]), FirstBytes::King => bits.extend(&[true, false, true]), FirstBytes::ShortCastling => bits.extend(&[true, true, false]), FirstBytes::LongCastling => bits.extend(&[true, true, true]), } 
```

### The destination square

In all cases except castling, you need to know the square that the piece is moving to, to reproduce the SAN. So we’ll skip this entirely when castling, but otherwise always include 6 bits for the destination square.

#### Just kidding, pawn moves!

There’s another exception to the destination square rule that might not seem so obvious. Let’s take the move `exf6`. We know it’s a pawn move from the e-file, so we don’t really need to encode the file it’s capturing using 6 bits. After all, you’ll never see `exa6`. So in these cases instead of 6 bits for the destination square, we only need 4 (one for the direction, and one for the rank).

But we can get even more clever here. Take `hxg6` for example. You know as soon as you see `hx`, that the file is going to be the g-file. So we don’t even need the extra bit to encode direction, we can just encode the file once, and the rank once.

So here’s the setup:

*   Pawn capture from b-g files: 3 bits for the file you’re capturing from, 1 bit for the direction of the capture, and 3 bits for the rank
    *   Total: 10 bits for movement
*   Pawn capture from a and h files: 3 bits for the file you’re capturing from, 3 bits for rank
    *   Total: 6 bits for movement

### “Special” moves

It’s a bit of a waste to encode for each move, whether it’s a promotion, check, checkmate, capture, etc. After all, the vast majority of chess moves in a game are not any of these.

So we’ll devote one bit to determining whether a move is “special”. It means we have to use an extra bit for moves that are promotions/checks/captures, but it also means that we save a whole lot of bits for “regular” moves.

Promotions are nice, because even though there are 6 chess pieces, there are only 4 valid pieces that you can promote to (after all, you can’t promote to a King or Pawn). So we only need 2 bits instead of 3, to encode the promotion piece.

```
if let Some(promotion) = promotion {  bits.push(true); bits.extend(match promotion { PromotionPiece::Queen => &[false, false], PromotionPiece::Rook => &[false, true], PromotionPiece::Bishop => &[true, false], PromotionPiece::Knight => &[true, true], }); } else {  bits.push(false); } 
```

### Disambiguation

Disambiguation is a bit thorny. You have to encode a surprising amount of information, to be able to decode the SAN exactly as you received it.

Take `Ngf3` for example. Besides the usual stuff you also need to encode the disambiguation (file=g). There are 3 different disambiguation possibilities (rank, file, or whole square), so we need 2 bits. One goes to waste but disambiguated moves aren’t that common and I can’t think of a nice way to take advantage of that last permutation.

So we always use 2 bits, then either 3 bits for the rank, 3 bits for the file, or 6 bits for a whole square (very rare).

```
if let Some(disambiguate) = disambiguate {  bits.push(true); match disambiguate { Disambiguation::File(file) => { bits.extend(&[false, true]); bits.extend(square_component_to_bits(file)); } Disambiguation::Rank(rank) => { bits.extend(&[true, false]); bits.extend(square_component_to_bits(rank)); } Disambiguation::Square(square) => { bits.extend(&[true, true]); bits.extend(square_component_to_bits(square.0)); bits.extend(square_component_to_bits(square.1)); } } } else {  bits.push(false); } 
```

## How does this do?

| Move | Original bits | Encoded bits | Savings |
| --- | --- | --- | --- |
| e4 | 16 | 10 | 37.5% |
| exd5 | 32 | 12 | 62.5% |
| Nf3+ | 24 | 10 | 58.33% |
| Qxa5+ | 40 | 16 | 60% |
| cxd8=Q# | 56 | 16 | 71.43% |

Not bad! We’re saving anywhere from 37.5% to 71.43% of the bits.

## PGNs

You may be thinking something like this: “Isn’t is sorta cheating to measure these in bits? Since you can only address one byte at a time, needing 10 bits for a move is virtually the same as 16 bits”

Well yes, that’s true and means that we get less savings when storing individual SANs. But they don’t account for the majority of what I’m storing, which are PGNs.

PGNs are a way to store lines or games, and they look something like this:

```
1.e4 d5 2.exd5 Nf6 3.d4 Bg4 4.Be2 Bxe2 5.Nxe2 Qxd5 6.O-O Nc6 7.c3 O-O-O 8.Qb3 Qh5 9.Nf4 Qh4 10.Qxf7 Ng4 11.h3 Nf6 12.Be3 e6 13.Qxe6+ Kb8 14.Nd2 Bd6 15.g3 Qh6 16.Qf5 Ne7 17.Qa5 b6 18.Qa6 Bxf4 19.Bxf4 Qxh3 20.Nf3 Ned5 21.Be5 Qh5 22.Kg2 Qg6 23.Rae1 Nh5 24.Nh4 Nhf4+ 25.Bxf4 Nxf4+ 26.Kh2 b5 27.Qxg6 hxg6 28.gxf4 Rxh4+ 29.Kg3 Rdh8 30.Re5 Rh3+ 31.Kg4 R8h4+ 32.Kg5 Rh5+ 33.Kxg6 Rh6+ 34.Kf7 Rh7 35.Rxb5+ Kc8 36.Rg5 g6+ 37.Kxg6 R3h6+ 38.Kf5 Rf7+ 39.Kg4 Rhf6 40.f5 Kd7 41.Re1 Kd6 42.Re8 Kd7 43.Ra8 Rh6 44.Rxa7 Rfh7 45.Kf4 Rh4+ 46.Rg4 Rh2 47.f3 Rxb2 48.f6 Rf7 49.Rg7 Ke6 50.Rxf7 Kxf7 51.Rxc7+ Kxf6 52.a4 Ra2 53.Rc4 Ra3 54.d5 Ke7 55.Ke5 Kd7 56.f4 Ra2 57.Kd4 Rd2+ 58.Kc5 Ra2 59.Kb5 Kd6 60.Rd4 Rb2+ 61.Ka6 Ra2 62.a5 Kc5 63.d6 Ra3 64.d7 Kc6 65.d8=Q Rxa5+ 66.Kxa5 Kc5 67.Qc7# 
```

This PGN is 759 bytes. There’s a ton of wasted space though. One byte between each move (the space). Then **at least 3 bytes** between full moves ( `2.`). This is sort of crazy, and if we combine our SAN encoding, we can compress this to be way smaller.

If we encode the whole PGN using our SAN encoding, with no space between moves because we know once we’ve reached the end of a move, we can compress this specific 759-byte PGN down to **195 bytes**, for a savings of **74%**.

## Impact

This hasn’t been deployed yet, I’m working on a ton of other performance improvements. But I anticipate this along with EPD compression (which I may write another at article on) will reduce the size of the database by about 70%. We’re almost entierly read-constrained, which should mean a 3x speedup for the most expensive queries we run.

## Speed

Another consideration here is the speed of doing this encoding/decoding; will it just cancel out the gains from having a much smaller database? Turns out, computers are really fast at this stuff, and conversion to and from this encoding is a rounding error.

I’m using Rust + the `bitvec` library. Encoding and then decoding 1000 moves takes about 600,000ns, or 0.6ms. I haven’t taken a performance pass at all either, and there’s a few places I know are very inefficient. I’m guessing I’m not even within 10x of optimal, but it should be good enough.