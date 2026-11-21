<!--no-pdf-->
# Why Each Prediction Reads the Way It Does

Write one entry per case, before you run anything. Name the rule behind each
predicted field. One to three sentences is enough. The grader checks the
structure of this file only. It does not judge how good the reasoning is, and
it sets no word minimum.

For every case you predicted wrongly, add a short correction after you run
it: what you predicted, what the language printed, and which rule you
misread. Corrections belong in the correction commits, not the prediction
commit.

Commit this file together with `predictions.tsv`, before any case runs:

```bash
git add predictions.tsv REASONING.md
```

The example below uses a query that is not one of the 16 cases, so it gives
no answer away. `parent(pat, X)` reads `parent(pat, jim)` because the `parent`
facts are tried top to bottom and only that one has `pat` as its first
argument:

```text
## parent(pat, X)

The goal scans the parent facts in source order. Only parent(pat, jim) has
pat first, so X binds to jim. No second fact matches, so the first ; finds
nothing and the search ends after one answer.
```

## parent(tom, X)

The goal scans the parent facts in source order and takes the first whose first argument is tom. The first match it sees is parent(tom, liz), so X binds to liz. Aside from that, parent(tom, bob) also exists below, so the first ; finds it and X binds to bob, giving the count 2.

## parent(tom, bob)

The query has no variables so it has nothing to bind, and parent(tom, bob) exists, so the program returns true. No other fact that satisfies the goal exist, so the first ; finds nothing and the count is 1.

## parent(X,tom)

There are no facts in the knowledge base to know who tom's parent are, so the query has no answer and returns false. Moreover, following this, the first ; finds nothing and the count is 0.

## X = f(Y, a), Y = b

X = f(Y, a) doesn't compute anything. It unifies X with the term f(Y, a), leaving Y inside it unbound, and a later Y = b fills that in, giving X=f(b,a),Y=b with no more following answers after for the first ; and has count 1. 

## f(X, b) = f(a, Y)

f(X, b) unifies with f(a, Y) by binding X to a and Y to b, so X=a,Y=b with no more following answers after for the first ; and has count 1. 

## a = b

Both a and b are atoms and not variables, so they are fixed values and therefore cannot be equal to each other, producing false with no more following answers for the first ; and has count 1. 

## grandparent(tom, X)

The rule grandparent(tom, X) has two goals, parent(tom, P) and parent(P, X). The first goal parent(tom, P) binds P with liz since this is the first fact it comes across, but since parent(liz, X) is false, parent(tom, P) backtracks and rebinds P to bob instead. This gives us grandparent's second goal parent(bob, X) which binds X with ann since it is the first fact it comes across in the knowledge base. Aside from that, parent(bob, pat) also exists below, so the first ; finds it and X binds to pat, giving the count 2.

## ancestor(tom, X)

In the knowledge base, the base clause is written first so the rule ancestor(tom, X) -: parent(tom, X) is what the query uses, binding X with liz. The second query given by the first ; also uses this rule, rebinding X with bob. However, further backtracking after bob gives several more answers, namely: X=ann, X=pat, X=jim, and X=sue, totaling the count to 6.

## ancestor_r(tom, X)

Because ancestor_r(tom, X) evaluates the recursive rule ancestor_r(A, D) :- parent(A, P), ancestor_r(P, D) first, it binds P = liz, fails ancestor_r(liz, D) completely (since liz has no children), and backtracks to P = bob to dive down bob's branches and return X = sue first. On subsequent ; backtracks, Prolog continues unwinding through bob's descendants to yield X = jim, X = ann, and X = pat before exhausting the recursive clause and finally falling back to the top-level base case parent(tom, D) to return X = liz and X = bob. Ultimately, changing the rule order still produces all 6 descendants, merely reversing the discovery order from deepest descendants to direct children.

## ancestor(X, jim)

Prolog resolves ancestor(X, jim) using top-to-bottom depth-first search, first matching the base case rule (parent(X, jim)) to return X = pat. Upon backtracking during the first ;, it switches to the recursive rule and explores every parent branch from top to bottom—failing down tom -> liz and bob -> ann -> sue before finding X = tom (via tom -> bob -> pat -> jim). Moreover, another ; gives X = bob (via bob -> pat -> jim) before it runs out of choices and terminates with false, giving count = 3.

## reach(a, d)

The rule is left-recursive, causeing Prolog to loop, because in order to satisfy the query reach(a,d) Prolog uses the first rule reach(X, Y) :- reach(X, Z), edge(Z, Y). This means that its next goal will be to satisfy the query reach(a,Z1) for some new variable Z, but to satisfy this new goal, Prolog again has to use the first rule, and this means that its next goal is going to be reach(a, Z2) for some new variable Z2. This gives rise to non-terminating computations, producing both unbounded results and count.

## reach_r(a, d)

This rule executes the base clause first before the one with the recursive call, so reach(a,d) first checks the rule reach_r(X, Y) :- edge(X, Y) and fails since there is no edge(a, d) fact that exists then proceeds to the second one, which is reach_r(X, Y) :- edge(X, Z), reach_r(Z, Y). Prolog first handles edge(a,Z), binding Z to b since we have the fact edge(a, b) and then starts the recursion with reach_r(b, d). This again fails the base clause since no edge(b,d) exist so it goes on to the second rule to have edge(b, Z) bind Z to c and call reach_r(c,d), which in turn satisfies the base clause because we have the edge(c,d). Therefore, Prolog returns true on the first query. However, for the first ;, Prolog backtracks to call reach_r(c,d) but use the recursive definition instead of the base clause, seeing edge(c, a) so it gets thrown in an infinite loop that answers true each time. Since it never hits false, its count is unbounded. 

## len([a, b, c], N)

The query matches the recursive rule of the predicate len, which is len([_|T], N) :- len(T, M), N is M + 1. len([a, b, c], N) fails the base clause and satisfies the second rule, recursing to len([b, c], M1), where N = M1 + 1, then again fails the base clause and recurses to len([c], M2), where M1 = M2 + 1. Finally, len([], M3), which satisfies the base clause, gives M3 = 0. Unwinding gives us M3 = 0 -> M2 = M3 + 1 = 0 + 1 = 1, M1 = M2 + 1 = 1 + 1 = 2, N = M1 + 1 = 2 + 1 = 3, no more answers for the first ;, giving count 1.

## join(X, Y, [1, 2])

The query matches the base clause join([], L, L), binding X to [] and Y to [1, 2] since the second and third terms are the same. For the first ;, Prolog proceeds to the recursive clause join([H|T], L, [H|R]) :- join(T, L, R) where join(X, Y, [1, 2]) calls join(X, Y, [2]) and it satisfying the base clause again, binding X to [] and Y to [2], since 2 is the tail of [1,2]. This can recurse another more time before it hits false, totaling count to 3.

## X > 2, elem(X, [1, 2, 3])

The goal order inside the query body puts X > 2 first, and it can only be checked when X already holds a number, which in this case it does not have one yet. Therefore, Prolog throws an (instantiation) error.

## elem(X, [1, 2, 3]), X > 2

The compound query takes each element in the list and binds it to X (goal 1) and checks if X > 2 (goal 2). It first binds X to 1 since 1 is the head and matches the base clause, but 1 is not greater than 2, so Prolog unbinds X, backtracks to the recursive clause, and calls elem(X, [2, 3]). There, it matches the base clause to bind X to 2, but again 2 is not greater than 2, so it unbinds X and recurses one last time as elem(X, [3])—binding X to 3 via the base clause, verifying that 3 > 2, and returning X=3. The first ; recurses into elem(X, []) which fails, so the count remains at 1.

