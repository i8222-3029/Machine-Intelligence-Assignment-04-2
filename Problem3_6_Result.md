# Problem 3.2: Propositional vs. First-Order Expressiveness

## Part A. Propositional Encoding Cost

In the propositional encoding (as in Q9 and Q17), adjacency is not encoded as a separate relation—it is embedded in the structure of each biconditional. We count the sentences that explicitly reference which squares neighbor which.

1. **Adjacency**: In propositional logic there is no binary predicate, so adjacency is implicit. If we wanted to state it explicitly (e.g., using propositions \(A_{(i,j),(i',j')}\)), we would need one assertion for every ordered pair of locations: \((n^2)^2 = n^4\) sentences (true for adjacent pairs, false otherwise). For \(n = 100\): \(10^8\) sentences.

2. **Creaking rule**: One biconditional per square, each listing adjacent \(D\) variables in the disjunction. There are \(n^2\) squares, so \(n^2\) sentences. For \(n = 100\): 10,000 sentences.

3. **Safety rule**: One biconditional per square: \(S_{i,j} \leftrightarrow \neg D_{i,j} \land \neg F_{i,j}\). Again \(n^2\) sentences. For \(n = 100\): 10,000 sentences.

Including the rumbling rule (another \(n^2\)), the physics encoding requires \(3n^2\) biconditionals. For \(n = 100\): 30,000 propositional sentences for physics alone.

## Part B. FOL Comparison

In first-order logic, the three physics rules each require a single quantified sentence:

1. \(\forall L\; Creaking(L) \leftrightarrow \exists L'\; Adjacent(L,L') \land Damaged(L')\)
2. \(\forall L\; Rumbling(L) \leftrightarrow \exists L'\; Adjacent(L,L') \land Forklift(L')\)
3. \(\forall L\; Safe(L) \leftrightarrow \neg Damaged(L) \land \neg Forklift(L)\)

3 sentences encode the physics for any grid size. Structural facts still scale: closed-world adjacency requires \(O(n^4)\) ground assertions (one per ordered pair of locations, stating adjacent or not), and domain closure adds \(O(n^2)\) terms. But the physics rules themselves—the part that grows as \(3n^2\) in the propositional encoding—are constant in FOL.

## Part C. Translation Challenge

FOL: The statement “there is exactly one damaged floor section” is concise:

\[
\exists L\; Damaged(L) \land \forall L'\; (Damaged(L') \rightarrow L' = L)
\]

Propositional logic for \(3 \times 3\): With 9 damage variables \(D_{1,1}, D_{2,1}, \ldots, D_{3,3}\), we need:

- **At-least-one**: \(D_{1,1} \lor D_{2,1} \lor D_{3,1} \lor D_{1,2} \lor D_{2,2} \lor D_{3,2} \lor D_{1,3} \lor D_{2,3} \lor D_{3,3}\) (1 clause)
- **At-most-one**: For every pair of distinct variables, assert that they are not both true: \(\neg D_{i,j} \lor \neg D_{k,\ell}\). There are \(\binom{9}{2} = 36\) such clauses

Total: \(1 + 36 = 37\) clauses for \(3 \times 3\).

Scaling: For an \(n \times n\) grid with \(n^2\) damage variables, the at-most-one constraint requires

\[
\binom{n^2}{2} = \frac{n^2(n^2-1)}{2}
\]

pairwise exclusion clauses, which is \(O(n^4)\). For \(n = 100\): \(\binom{10,000}{2} = 49,995,000\) clauses. The FOL version remains 1 sentence regardless of \(n\).

So yes, the statement can be expressed in propositional logic, but the encoding cost grows as \(O(n^4)\)—a dramatic illustration of the expressiveness gap.

## Part D. Practical Implications

1. **Propositional at scale**: The basic physics encoding (\(3n^2 = 30,000\) biconditionals for \(n = 100\)) is well within the capacity of modern SAT solvers, which routinely handle millions of clauses. However, richer constraints like “exactly one damaged floor” add \(O(n^4) \approx 50\) million clauses, which is feasible but pushing practical limits. Adding multiple such cardinality constraints rapidly becomes unwieldy.

2. **FOL restrictions for tractability**: Unrestricted FOL is semi-decidable (and undecidable for satisfiability in general). Tractable restrictions include: (a) Horn clauses, which enable polynomial-time forward and backward chaining; (b) function-free FOL over finite domains, which is decidable; (c) description logics, which restrict quantifier patterns to guarantee decidability while retaining useful expressiveness; (d) bounded-variable fragments, which limit the number of variables.

3. **Hybrid approach**: Use FOL for the general physics rules (3 quantified sentences, grid-size independent), and propositionalize only the local neighborhood around the agent’s current position for fast inference. For example, the agent could maintain a FOL KB for global constraints but ground the creaking/rumbling rules only for the squares within \(k\) steps of its current location. This combines FOL’s compact representation with propositional logic’s efficient (and decidable) inference.
