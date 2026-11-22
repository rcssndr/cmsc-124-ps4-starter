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

It checks the parent facts from top to bottom. It first finds parent(tom, liz), so X is bound to liz. When asked for another answer using a semicolon, it continues searching and finds parent(tom, bob), making X = bob. Since there are no more matching facts, the search stops after 2 answers.

## P02

Since there are no variables in this query, it only needs to check if parent(tom, bob) exists in the knowledge base. It finds an exact match, so the query succeeds and returns true. There are no other possible answers, leaving a total of 1 answer.

## P03

The query checks if there is someone who has tom as their child. It searches the parent facts from top to bottom, but none of them have tom as their second argument. Since there is no matching fact in the knowledge base, the query fails and returns false, with 0 answers.

## P04

It evaluates the goals from left to right while keeping the variable bindings between them. First, X = f(Y, a) creates a structure where Y is still unbound. Then, Y = b assigns b to Y, which also updates the value inside X. As a result, X becomes f(b, a), giving 1 successful answer.

## P05

It checks whether the two compound terms can be unified. Since both terms have the same functor, f/2, it compares their arguments one by one. This binds X = a and Y = b. Since both sides match completely in one step without any conflicts, it succeeds once with a count of 1.

## P06

The atoms a and b are different constant values, so they cannot be unified. Since neither value is a variable that can be assigned to make the two equal, the query immediately fails. It returns false with 0 answers.

## P07

The grandparent rule checks two parent relationships: first, it finds someone who is a child of tom, then checks if that person has a child. It first tries P = liz, but she has no children, so it goes back and tries P = bob. From there, it finds X = ann first, followed by X = pat when searching for another answer. This gives a total of 2 answers.

## P08

The base rule parent(A, D) comes before the recursive rule, so it checks direct parent relationships first before going deeper into the family tree. It first finds tom's direct children, X = liz and X = bob. After that, it continues through the recursive rule to find the remaining descendants. The search produces 6 answers in total.

## P09

Unlike P08, the recursive rule is placed before the base rule. Because of this, it explores deeper relationships first before checking the direct parent relationships. It follows each branch of the family tree before moving to the next one, finding X = sue first and X = jim second. Although the search order is different, it still finds 6 answers in total.

## P10

The query searches for the ancestors of jim. It first uses the base rule and finds X = pat, who is jim's direct parent. When searching for another answer, it uses the recursive rule and checks the parent relationships from top to bottom. This leads to X = tom being found before X = bob, giving 3 answers in total.

## P11

The first clause of reach calls itself again before checking any actual graph edges. Since it keeps repeating the same call with the starting point a, it gets stuck in an endless recursive loop. It never reaches a point where it can produce an answer, leaving the remaining fields unbounded.

## P12

Unlike P11, this rule checks an actual graph edge first before calling itself recursively. This allows it to follow the path a -> b -> c -> d and successfully return true for the first answer. However, when searching for another answer, it can follow the cycle a -> b -> c -> a repeatedly. Because there is no condition to stop the cycle, the search continues indefinitely, making the second answer and total count unbounded.

## P13

The len rule counts the elements by removing one element from the head of the list at a time. It continues doing this until the list becomes empty, which matches the base case len([], 0). As the recursive calls return, it adds 1 for every element counted (0 + 1 + 1 + 1), resulting in N = 3. Since there are no alternative branches, the query produces 1 answer.

## P14

The join rule can also be used to split a list into two parts. It first uses the base rule, giving X = [] and Y = [1, 2]. When it goes back to look for other answers, the recursive rule moves one element at a time from Y to X, producing [1] and [2], followed by [1, 2] and []. This results in 3 valid answers.

## P15

It evaluates the goals from left to right, so it tries to evaluate X > 2 before checking elem. However, X does not have a value yet, and arithmetic comparisons require an assigned numeric value. Because of this, it encounters an instantiation error before it can check the elements of the list. The query therefore results in an error for all fields.

## P16

Unlike P15, the goals are arranged so that elem runs before the comparison. This allows it to generate values for X from the list first. It tries X = 1 and X = 2, but both fail the condition X > 2. It then tries X = 3, which satisfies the condition and becomes the only successful answer.
