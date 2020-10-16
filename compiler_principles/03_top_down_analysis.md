# Top-down analysis

cf. bottom-up analysis.

## Introduction

### Two causes of non-determinism

... in every derivation step...

- which nonterminal symbol to derive from
- which production rule to use

#### Determine nonterminal symbol

Use a left-most derivation.

### Deterministic top-down analysis

lookahead method: 
- scan once from left to right
- allowed to look at most N alphabets ahead while scanning

There are CFGs that cannot be analyzed by this method.

- left recursions
  > analyze `ba^n` using CFG G:
  > `S -> Sa | b`
  > must look ahead n+2 alphabets to determine a derivation sequence
  - there are also "indirect left recursions":
  > `S -> Aa`
  > `A -> Sb`
  - can remove left recursion on certain occasions

- left common factor
  > `S -> aAb | aAc`
  > `A -> a | aA`
  - can remove left common factors on certain occasions
  
## LL(1) analysis

