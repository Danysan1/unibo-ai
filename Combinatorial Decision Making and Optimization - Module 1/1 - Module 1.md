# Constraint Programming

## Constraint Satisfaction Problems

CSP = $<X,D,C>$

Variables $X = \{X_1, X_2, \dots, X_n\}$

Domains $D = \{D_1, D_2, \dots, D_n\}$ (binary, integer, continuous, finite set, structured types)

Constraints $C = \{C_1, C_2, \dots, C_m\}$

### Constraints

Extensional: cite all allowed combinations

Intensional: declarative relation among objects (preferred if possible)
- Algebraic
- Logical 
- Global (should be preferred to manual constraints)
    - alldistinct$([X_i, X_j, \dots, X_k])$
    - Global Cardinality: gcc$([X_1, X_2, \dots, X_k], [v_1, v_2, \dots, v_m], [count_1, count_2, \dots, count_m])$
    - among$([X_i, X_j, \dots, X_k], \{\dots\}, minCount, maxCount)$
    - disjunctive / noOverlap$([start_1, start_2, \dots, start_k], [dur_i, dur_j, \dots, dur_k])$
    - cumulative$([start_1, start_2, \dots, start_k], [dur_1, dur_2, \dots, dur_k], [R_1, R_2, \dots, R_k], capacity)$
    - Lexicographic ordering: lex$\le([X_1, X_2, \dots, X_k], [Y_1, Y_2, \dots, Y_k])$
    - table$([X_1, X_2, \dots, X_k], dictionary)$
    - Many more available in MiniZinc
- Meta-constraints (ex. $\sum_i (X_i > t_i) > 5$ )

#### Implied vs redundant constraints
- Implied constraint: already implied in other constraints but REDUCES computation
  - Variable symmetry breaking constraints
  - Value symmetry breaking constraints
- Redundant constraint: already implied in other constraints, INCREASES computation

#### Constraint propagation

Local consistency
- Generalized Arc Consistency (GAC, very expensive, finds inconsistencies earlier)
- Bounds Consistency (BC, less expensive, misses some inconsistencies, sometimes worth it)
- Hybrids and variations of GAC and BC
- Dedicated propagation algorithms

Backtracking Tree Search combines depth-first tree search with constraint propagation to quickly solve CSPs.

#### Improving performances

Variable Ordering Heuristics (VOH) further optimize search prioritizing values most likely to fail as to maximize propagation (Fail-First principle).
An example is **domWdeg**, which prioritizes variables with the minimum domain size and the most number of constraints weighted by the number of constraint fails.

In addition we can use other techniques to exit of big useless decision subtrees, like **randomization** and **restarting** learning from accumulated experience:
* constant restart (after a certain number $L$ of resources is used)
* geometric restart (after $L$, $\alpha L$, $\alpha^2 L$, ...)
* Luby restart (after $s[i]L$ where $s[i]$ is the $i$nth number in the Luby sequence)

These are ineffective on simple minimum domain size but are very effective with domWdeg because it carries constraint weights from one round to the next.

### Constraint Optimization Problems

COP = $<X,D,C,f>$

Optimization criterion / objective function $f$

Goal: satisfy the CSP $<X,D,C>$ and minimize $f$

#### Branch & Bound algorithm

The CSP is run multiple times, each time a "bounding constraint" is added/updated that ensures the objective function output is better than the previous best.
