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

## P01

The goal scans the parent facts in source order and takes the first whose first argument is tom. The first match it sees is parent(tom, liz), so X binds to liz. Aside from that, parent(tom, bob) also exists below, so the first ; finds it and X binds to bob, giving the count 2.
The goal scans the parent facts in source order (top to bottom) and first finds parent(tom, liz), so X is bound to liz. When asked for another answer using a semicolon, it continues searching and finds parent(tom, bob), making X = bob. Since there are no more matching facts after this, the search stops after 2 answers.

## P02

The query has no variables, so it has nothing to bind. It only needs to check if parent(tom, bob) exists in the knowledge base and finds an exact match, so the query succeeds and returns true. There are no other possible answers, leaving a total of 1 answer.

## P03

The query checks if there is someone who has tom as their child. It searches the parent facts from top to bottom, but none of them have tom as their second argument. Since there is no matching fact in the knowledge base, the query fails and returns false, with no (0) answers.

## P04

X = f(Y, a) doesn't compute anything; it just evaluates the goals from left to right while keeping the variable bindings between them. First, X = f(Y, a) creates a structure where Y is still unbound. Then, Y = b assigns b to Y, which also updates the value inside X. As a result, X becomes f(b, a), giving 1 successful answer.

## P05

It checks whether the two compound terms can be unified. Since both terms have the same functor, f/2, it compares their arguments one by one and unifies f(X, b) with f(a, Y) by binding X = a and Y = b. Since both sides match completely in one step without any conflicts, it succeeds once with a count of 1.

## P06

Both a and b are atoms and not variables, so they are different constant values and cannot be unified. Since neither value can be assigned to make the two equal, the query immediately fails. It returns false with 0 answers.

## P07

The grandparent rule checks two parent relationships: first, it finds someone who is a child of tom, then checks if that person has a child. It first tries P = liz, but she has no children, so it backtracks and tries P = bob. From there, Prolog finds bob’s children with X = ann first, followed by X = pat when searching for another answer. This gives a total of 2 answers.

## P08

In the knowledge base, the base rule parent(A, D) comes before the recursive rule, so it checks direct parent relationships first before going deeper into the family tree. It first finds tom's direct children, binding X to  liz for the first answer and X to bob for the next query. After that, it continues through the recursive rule to find tom’s remaining descendants, producing 6 answers in total.

## P09

Write one to three sentences here. Name the rule that fixes each field.

## P10

Write one to three sentences here. Name the rule that fixes each field.

## P11

Write one to three sentences here. Name the rule that fixes each field.

## P12

Write one to three sentences here. Name the rule that fixes each field.

## P13

Write one to three sentences here. Name the rule that fixes each field.

## P14

Write one to three sentences here. Name the rule that fixes each field.

## P15

Write one to three sentences here. Name the rule that fixes each field.

## P16

Write one to three sentences here. Name the rule that fixes each field.

