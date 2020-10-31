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

#### First-set

for CFG G=(V<sub>T</sub>, V<sub>N</sub>, P, S), the first-set of any &alpha; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup> is defined as follows:
- **Fi**(&alpha;) = \{ a | &alpha; =>\* a &beta;, a &isin; V<sub>T</sub>, &beta; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup>, or &alpha; =>\* a where a=	&epsilon; \}

explanation: **Fi**(&alpha;) contains the first token of any possible derivation of &alpha;, or &epsilon; for &epsilon; production rule

##### Calculation of *Fi*

```python
def get_Fi(V_T, V_N, P, S):
# setup
    for x in (V_T + V_N + {epsilon} + {any suffix of any production rule}).closure():
        if x in V_T or x == epsilon:
            Fi[x] = {x}
        else:
            Fi[x] = {}

    while Fi_changed(): # do until Fi does not change anymore
        for p in P:
            (A, beta) = p  # p = A->beta
            if beta == epsilon:  
                Fi[A] = Fi[A]+{epsilon}
            for Ys in u.suffix():  # generates [Y1, Y2, ..., Yk] where Yj in V_n.join(V_T)
                
        
```


#### Follow-set

for CFG G=(V<sub>T</sub>, V<sub>N</sub>, P, S), the first-set of any A &isin; V<sub>N</sub> is defined as follows:

- **Fo**(A) = \{a | S# =><sup>\*</sup> &alpha;A&beta;# and a &isin; Fi(&beta;#), where &alpha;, &beta; &isin; (V<sub>T</sub> &cup; V<sub>N</sub>)<sup>\*</sup>}, where # is the end-of-input token

##### Calculation of *Fo*
```python
def get_Fo(V_T, V_N, P, S):
    for x in V_N:
        Fo[x] = {} 
    Fo[S] = {EOI}
    while Fo_changed():
        for p in P:
            (A, w) = p  # A -> w
            for B in V_N:
                if B in w:  # A -> xBy
                    x, B, y = w
                    Fo[B] += Fi[y] - {epsilon}
                    if epsilon in Fi(y):
                        Fo[B] += Fo[A]
```

### Predictive set

PS(A -> &alpha;) = 
- Fi(&alpha;) if &epsilon; &notin; Fi(&alpha;)
- Fi(&alpha;) - \{&epsilon;\}) &cup; Fo(A)

### LL(1) grammar

G is LL(1) *iff* PS(A -> &alpha;) &cap; PS(A -> &beta;) = &empty; for any A -> &alpha; | &beta;.

### Recursive descent LL(1) parser

Each nonterminal symbol corresponds to a subroutine which determines the production rule by reading the next input token, *a*.

When scanning through the production rules, scan their right-hand side:

- for terminal symbol *t*, judge if *a* == *t*:
  - if *a* == *t*: get the next token
  - else: error
- for nonterminal symbol *A*, call the corresponding subroutine.

