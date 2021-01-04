---
layout: post
title:  "How FM-index works"
date:   2020-08-30 23:30:07 +0800
category: bio
author: m0ssc0de
tags: [bio, cs, array, algorithm]
summary:
---
# FM-index

FM-index is a method of index text. And the index could help to locate text fastly in a huge mass text.
I learn it from Wikipedia. Below is my restate. In order to be consistent with Wikipedia, the serial number of array count from `1`.

## Burrows-Wheeler transform (BWT)

Burrows-Wheeler transform (BWT) is the first thing to clear. It's used to compress data Originally. But

Let's get in detail.

Its name has the word "transform". So the raw data will be transformed into another form. Then it will become back.

Now we have a data "ABABC$". And list all its suffix.

- 1 "ABABC$"
- 2 "BABC$"
- 3 "ABC$"
- 4 "BC$"
- 5 "C$"
- 6 "$"

The serial number represents the suffix's location in origin text. We use `k` to representation this number.

Every character will have a weight. we assume "$" is smallest. "A" smaller than "B".
"B" smaller than "A". The relation of them are similar with dictionary, Lexicographic order.
By this kind order, we can have a new array.

"$" is the smallest one.

"ABABC$" and "ABC$" is smaller than others exclude "$". The third character of these two is different.
And "A" is smaller than "C". So "ABABC$" is smaller than "ABC$". And so on. There will be a suffix array.

- 1 "$"         6
- 2 "ABABC$"    1
- 3 "ABC$"      3
- 4 "BABC$"     2
- 5 "BC$"       4
- 6 "C$"        5

The first number is the serial number in the suffix array. We use `i` to represent it. The second number is the serial number in the original text, the `k`.
A real suffix array in computer memory will look like this `[6, 1, 3, 2, 4, 5]`.

In fact, the suffix could be a part of the rotation of origin text. "BABC$" is a part of "BABC$A". "BC$" is a part of "BC$ABA". We could complete the text for every suffix. Then there will be a new matrix.

- "$ABABC"
- "ABABC$"
- "ABC$AB"
- "BABC$A"
- "BC$ABA"
- "C$ABAB"

The last character of every suffix array item comes from the original text. The array of them is the result of the Burrows-Wheeler transform. So we get a bwt, `['C', '$', 'B', 'A', 'A', 'B']`.

Now there are two array, `sa [6, 1, 3, 2, 4, 5]`, `bwt ['C', '$', 'B', 'A', 'A', 'B']`. We could use them to get the origin text.


First, create a table. Count number of characters that are small than the current characters in the `bwt`.

| `c`    | $   | A   | B   | C   |
| ------ | --- | --- | --- | --- |
| `C[c]` | 0   | 1   | 3   | 5   |

There is nothing which is small than `$`. So it's 0(`C[$]=0`). There is a `$` which is small than `A`. So it's 1(`C[A]=1`).
There is 2 `A` and 1 `$`. So it's 5(`C[B]=3`).

Second, create another table. It's different from the last table. The order of the bwt will be considered.

|     | C, 1 | $, 2 | B, 3 | A, 4 | A, 5 | B, 6 |
| --- | ---- | ---- | ---- | ---- | ---- | ---- |
| $   | 0    | 1    | 1    | 1    | 1    | 1    |
| A   | 0    | 0    | 0    | 1    | 2    | 2    |
| B   | 0    | 0    | 1    | 1    | 1    | 2    |

The first `0` means there is no one `$` from start to current position, "1". (`Occ($, 1)=0`). The next `1` means there is one `$` from start to current position, "2".(`Occ($, 2)=1`).

The `2` at second line's last means there are two `A` from start to current position, "6". (`Occ(A, 6)=2`).

At the third line, `1` means there is one `B` from start to current position. `Occ(B, 1)=0`, `Occ(B, 3)=1`

Let's back to the matrix again.

- 1 "$ABABC"
- 2 "ABABC$"
- 3 "ABC$AB"
- 4 "BABC$A"
- 5 "BC$ABA"
- 6 "C$ABAB"

The first character of every line comes from the original test, so do the last one.
So a character of the last characters must could be located to the first character.
With the help of `sa` and `bwt`, it will be complete.

Let's choose the first `B` in `bwt`. `bwt(3)=B`.

>`bwt ['C', '$', 'B', 'A', 'A', 'B']`

With the help of the first table, we could get a number. `C[B]=3`.

With the help of the second table, we could get `Occ(B, 3)=1`.

`C[B] + Occ(B, 3)` is `4`. So the first `B` in `bwt` is the `B` located in the fourth line's beginning.

-    1 "$ABABC"
-    2 "ABABC$"
-    3 "ABC$AB"👈
- 👉 4 "BABC$A"
-    5 "BC$ABA"
-    6 "C$ABAB"

Use the way comes from Wikipedia to calculate:

```
LF(3) = C[L[3]] + Occ(L[3],3)
L[3] = B
LF(3) = C[B] + Occ(B, 3)
LF(3) = 3 + 1 = 4
```

Don't forget the `sa`. With the help of it, we know the 4th item of `sa` is the 2nd item of origin text. `sa[4]=2`

> `sa [6, 1, 3, 2, 4, 5]`

With the counting of every item in `bwt`, we could get the whole origin text.

So this is how the Burrows-Wheeler transform (BWT) works.

## Search Pattern

Assume we want to search pattern "AB" in "ABABC$". First setp, search "B" by a function. And then search "A" by another function with the result of last step. Let's try.

The first step. `[C["B"] + 1 .. C["B"+1]]`. `[C["B"]]` will get 3 with the help of first table below. `"B"+1` is the next character in Lexicographic order, "C". So step by step, `[C["B"] + 1..C["C"]]`, `[3 + 1, 5]`, `[4, 5]`.
"4" is the start, and "5" is the end.

The second step, `[C["A"] + Occ("A", start-1) + 1 .. C["A"] + Occ("A", end)]`. The `start` and `end` come from the last step. The result of step by step.

- `[C["A"] + Occ("A", start-1) + 1 .. C["A"] + Occ("A", end)]`
- `[C["A"] + Occ("A", 4-1) + 1 .. C["A"] + Occ("A", 5)]`
- `[C["A"] + Occ("A", 3) + 1 .. C["A"] + Occ("A", 5)]`
- `[C["A"] + Occ("A", 3) + 1 .. C["A"] + Occ("A", 5)]`
- `[1 + Occ("A", 3) + 1 .. 1 + Occ("A", 5)]`
- `[1 + 0 + 1 .. 1 + 2]`
- `[2 .. 3]`

`[2..3]` means from 2nd to 3rd suffix start with the "AB". With the help of `sa`, `sa [6, 1, 3, 2, 4, 5]`, they correspond to `1, 3`. It means "AB" are located in first and third in origin text.

We saw the number of search steps only relate to the long of pattern. This mean the search complete in linear time in the length of the pattern: O(p) time. So this is why it's suitable for search pattern in long text.