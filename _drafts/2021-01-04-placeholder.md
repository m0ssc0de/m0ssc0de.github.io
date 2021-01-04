---
layout: post
title: How search in big
date: 2021-01-04 12:00
author: m0ssc0de
tags: [rust, bio, alg]
summary:
---

```rust
//[dependencies]
//bio = "*"

use bio::alphabets;
use bio::data_structures::bwt::{bwt, less, Occ};
use bio::data_structures::fmindex::{FMIndex, FMIndexable};
use bio::data_structures::suffix_array::suffix_array;

fn main() {
    let text = b"ACAGCTCGATCGGTA$";
    let pattern = b"ATCG";
    let alphabet = alphabets::dna::iupac_alphabet();
    // calculate a suffix array
    let sa = suffix_array(text);
    // calculate the Burrows-Wheeler-transform
    let bwt = bwt(text, &sa);
    // calculate the vectors less and Occ (occurrences)
    let less = less(&bwt, &alphabet);
    let occ = Occ::new(&bwt, 3, &alphabet);
    // set up FMIndex
    let fmindex = FMIndex::new(&bwt, &less, &occ);
    // do a backwards search for the pattern
    let interval = fmindex.backward_search(pattern.iter());
    let positions = interval.occ(&sa);
    println!("{:?}", positions);
}
```