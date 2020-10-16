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

- left recursion: $S -> Sa$