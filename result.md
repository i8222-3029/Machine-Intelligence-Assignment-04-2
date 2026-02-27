# Problem 3.6 — Propositional vs. First-Order Expressiveness

## Part A. Propositional Encoding Cost

Let the warehouse be an \(n \times n\) grid, and let \(N=n^2\) be the number of locations.

### 1) Adjacency relation
Using 4-neighborhood adjacency (up, down, left, right), the number of undirected adjacent pairs is
\[
E = 2n(n-1)
\]
because there are \(n(n-1)\) horizontal pairs and \(n(n-1)\) vertical pairs.

So, if adjacency facts are listed propositionally, the number of sentences is
\[
2n(n-1)=O(n^2).
\]

### 2) Creaking rule
For each location \(l\):
\[
C_l \leftrightarrow \bigvee_{k \in Adj(l)} D_k
\]
One sentence per location, so total:
\[
n^2=O(n^2).
\]

### 3) Safety rule
For each location \(l\):
\[
S_l \leftrightarrow (\neg D_l \land \neg F_l)
\]
Again one sentence per location, so total:
\[
n^2=O(n^2).
\]

### Total number of propositional sentences
\[
2n(n-1)+n^2+n^2 = 4n^2-2n.
\]

For \(n=100\):
- Adjacency: \(2\cdot100\cdot99=19{,}800\)
- Creaking: \(10{,}000\)
- Safety: \(10{,}000\)
- Total: \(39{,}800\) sentences

Note: If one also explicitly encodes all non-adjacent pairs as false, complexity can rise toward \(O(n^4)\).

---

## Part B. FOL Comparison

Use predicates:
- \(Adj(x,y)\): \(x\) and \(y\) are adjacent
- \(Damaged(x)\), \(Creak(x)\), \(Forklift(x)\), \(Safe(x)\)

FOL rules:

1. Adjacency definition (coordinate-based schema)
\[
\forall x\forall y\big(Adj(x,y) \leftrightarrow \text{(x and y differ by one grid step)}\big)
\]

2. Creaking rule
\[
\forall l\big(Creak(l) \leftrightarrow \exists m(Adj(m,l)\land Damaged(m))\big)
\]

3. Safety rule
\[
\forall l\big(Safe(l) \leftrightarrow (\neg Damaged(l)\land \neg Forklift(l))\big)
\]

The key point: the number of FOL sentences is constant with respect to \(n\) (roughly 3 core rules, plus background axioms if needed).

---

## Part C. Translation Challenge

Statement: “There is exactly one damaged floor section in the entire warehouse.”

### FOL form
\[
\exists x\Big(Damaged(x)\land \forall y(Damaged(y)\rightarrow y=x)\Big)
\]
(or equivalently \(\exists!x\,Damaged(x)\)).

### Can this be expressed in propositional logic?
- **Yes**, for a fixed finite grid size (e.g., \(3\times3\)).
- **No**, as one size-independent schema over arbitrary domains, because propositional logic has no quantifiers.

### Propositional encoding for a \(3\times3\) grid
Let \(D_1,\dots,D_9\) denote “cell \(i\) is damaged.” Then:
\[
(D_1\vee\cdots\vee D_9)\land\bigwedge_{i<j}\neg(D_i\land D_j)
\]
- At least one damaged cell: 1 clause
- At most one damaged cell: \(\binom{9}{2}=36\) clauses
- Total: 37 clauses/sentences

For general \(n\times n\), \(N=n^2\):
\[
1+\binom{N}{2}=1+\frac{N(N-1)}{2}=1+\frac{n^2(n^2-1)}{2},
\]
which scales as \(O(n^4)\).

---

## Part D. Practical Implications (100×100, 10,000 locations)

1. **Is propositional encoding practical at this scale?**
   - For local rules (adjacency, creaking, safety), it can still be practical.
   - For global cardinality constraints (e.g., exactly one), naive encodings become very large and costly.

2. **What FOL restrictions improve tractability?**
   - Function-free fragments
   - Horn clauses / Datalog-style rules
   - Finite domains with bounded or guarded quantification
   - Stratified negation

3. **How can a hybrid approach work?**
   - Keep general domain knowledge in FOL as reusable templates.
   - Ground only the relevant local region for a specific query.
   - Solve the grounded part with SAT/SMT.
   - Use efficient cardinality encodings (counters/sorting networks) for global constraints.

In practice, this hybrid strategy gives both high expressiveness (from FOL) and scalable solving (from propositional methods).
