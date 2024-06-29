<!--yml

分类：未分类

日期：2024-05-27 14:46:54

-->

# RC4 with cards (was: Requesting feedback ...)

> 来源：[https://www.mail-archive.com/cryptography@wasabisystems.com/msg00065.html](https://www.mail-archive.com/cryptography@wasabisystems.com/msg00065.html)

```
>  > PS I tried it today.. plain RC4 can excellently be executed manually
>  > (base 52 or 26, instead of 256) using a deck of cards :-)
>
>i must have missed the original message about that.  could you share
>the details with the list?

There was no previous message.. it was more like a random comment

Using RC4 with a deck of cards:

Number of cards: (keysizes rounded down to a power of 2)
* 26 - Roughly equivalent to 64-bit encryption
* 52 - Roughly equivalent to 128-bit encryption

Other values are possible too, 26 are 52 are simply convenient. 
Different ranges might however allow for use of digits and 
punctuation in messages, so don't feel too constrained to these two 
values.

You also need two distinct "markers", for which you might use 
joker-cards, dice, or random objects.

All calculation is done modulo n, where n is the number of cards used.

(now and then I might use the word "byte", but in this context it 
doesn't mean a value in range 0..255 but a value in range 0..(n-1), 
obviously)

Enumeration of the cards:
What I usually do is give the Ace value 1, cards 2..10 values 2..10, 
J/Q/K values 11..13\. For spades, add nothing, for hearts add 13, and 
optionally for diamonds add 26 and for clubs add 39\. The last card 
(king of hearts (26) or clubs (52)) gets value 0 because of the 
modulo.

When putting them on the table, they can be arranged in rows of 13, 
or rows of 10, which every you find easiest to locate a card by index 
in. (I recommend putting card 0 at the topleft)

Encoding of text:
Well, use anything you like! A possible encoding is: 0 = space, 1 = 
A, 2 = B, .... 16 = P, 17 = Q / X, 18 = R, .. 23 = W, 24 = Y, 25 = Z. 
When using n=52, mod-26 the output of the keystream. When using other 
values for n, design your own encoding.

Initial order of the cards / key scheduling:

Method 1\. Give the cards a random permutation and duplicate the 
configuration of the deck. One deck is now the encryption deck and 
one the decryption deck. By keeping track of the byte-offset within 
the RC4 key stream, you can use the deck for quite a while before you 
risk deterioration of the security. The advantages are ease of use 
and high security, the disadvantages are the requirement for both 
parties to meet (or exchange a long n-value key) in advance and the 
cards must stay in order, so you can't play a game with them :-)

Method 2\. Agree on some way to determine a key (which consists of 
values in range 0..(n-1), usually an encoded string of text) and then 
use the standard RC4 key scheduling system. Advantages: character 
string can be used as key, key can be of any length, deck doesn't 
need to be pre-arranged. Disadvantages: RC4 key scheduling takes 
about as much time as encrypting n characters.

Method 3\. Use your own. Security through obscurity.

The two markers, X and Y are usually initially placed on card 0, but 
I suppose different values work too. Perhaps you can even vary them, 
use them as a kind of initialization vector. (Maybe this will

En-/decryption: Standard RC4, except all calculation is done modulo n 
instead of modulo 256\. Also, to encrypt a byte add the keystream byte 
to it, to decrypt use subtract the keystream byte from it. (If you 
pick a power of 2 for n, instead of 26 or 52, you can use 
alternatively use xor like normal RC4).

RC4 keystream generation using cards, per byte:
1\. Move the X marker to the next card
2\. Move the Y marker i cards forward where i is the value of the card 
marked by X
3\. Swap the cards marked by X and Y
4\. The keystream byte is the value of the ith card on table, where i 
is the sum of the value of the card marked by X and the card marked 
by Y

RC4 key scheduling (for Method 2), per byte: (loop i from 0 to n-1)
1\. Put the X marker on the ith card
2\. Move the Y marker i cards forward where i is the sum of the value 
of the card marked by X, and the ith character of the key (0-based, 
repeat if needed)
3\. Swap the cards marked by X and Y

That's all I think.. if there are any questions I'll hear 'em

  -xmath

Matthijs van Duin
- PGP Key: 0xB6205CCB   <finger:[EMAIL PROTECTED]> -
- FP: D73C 9EE3 5F6B E5D5 8E19  2CBE 4648 8C3E B620 5CCB -

---------------------------------------------------------------------
The Cryptography Mailing List
Unsubscribe by sending "unsubscribe cryptography" to [EMAIL PROTECTED]

```
