# Top-down parsing

cf. bottom-up parsing.

## Introduction

### Two causes of non-determinism

... in every derivation step...

- which nonterminal symbol to derive from
- which production rule to use

#### Determine nonterminal symbol

Use a left-most derivation.

### Deterministic top-down parsing

lookahead method: 
- scan once from left to right
- allowed to read ahead at most N tokens ahead while scanning, where N is a fixed value

There are CFGs that cannot be parsed by this method.

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
  
## LL(1) parser

An LL (left-to-right, leftmost derivation) parser is called an LL(k) parser if it uses k tokens of lookahead.

A grammer is called an LL(k) grammar if an LL(k) parser can be constructed from it. ([LL parser - Wikipedia](https://en.wikipedia.org/wiki/LL_parser))

### First-set (*Fi*) and Follow-set (*Fo*)

#### *Fi*

for CFG G=(V<sub>T</sub>, V<sub>N</sub>, P, S), the first-set of any &alpha; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup> is defined as follows:
- **Fi**(&alpha;) = { a | &alpha; =>\* a &beta;, a &isin; V<sub>T</sub>, &beta; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup>, or &alpha; =>\* a where a=	&epsilon; }

explanation: **Fi**(&alpha;) contains the first token of any possible derivation of &alpha;, or &epsilon; for &epsilon; production rule

##### Calculation of *Fi*

```python
def get_Fi(V_T, V_N, P, S):
# setup
    for x in (V_T.join(V_N).join({epsilon}).join({any suffix of any production rule})).closure():
        if x in V_T or x == epsilon:
            Fi(x) = {x}
        else:
            Fi(x) = {}

    while Fi_changed(): # do until Fi does not change anymore
        for p in P:
            (A, beta) = p  # p = A->beta
            if beta == epsilon:  
                Fi(A) = Fi(A).join(epsilon)
            for Ys in u.suffix():  # generates [Y1, Y2, ..., Yk] where Yj in V_n.join(V_T)
                
        
```


- 
