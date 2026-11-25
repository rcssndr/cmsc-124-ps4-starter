# The Search and the Stack

```text
== trace P07 ==
Call: grandparent(tom,X)
  Call: parent(tom,_)
  Exit: parent(tom,liz)
  Call: parent(liz,X)
  Fail: parent(liz,X)
  Redo: parent(tom,liz)
  Exit: parent(tom,bob)
  Call: parent(bob,X)
  Exit: parent(bob,ann)
Exit: grandparent(tom,ann)
answer 1: X=ann
Redo: grandparent(tom,ann)
  Redo: parent(bob,ann)
  Exit: parent(bob,pat)
Exit: grandparent(tom,pat)
answer 2: X=pat
Redo: grandparent(tom,pat)
  Redo: parent(bob,pat)
  Fail: parent(bob,X)
  Redo: parent(tom,bob)
  Fail: parent(tom,_)
Fail: grandparent(tom,X)
== 2 answers ==
```

## Which of its lines push a frame onto Activity 15's runtime stack? Which lines pop one?

Every Call: line pushes a new frame onto the runtime stack whenever Prolog encounters a new goal to prove. This is similar to a regular function call, where a new frame is added while the previous one waits. Each frame keeps track of the goal being attempted and its variable bindings. In our P07 trace, it starts with grandparent(tom, X), followed by parent(tom, _), parent(liz, X), and parent(bob, X).

On the other hand, Exit: and Fail: lines pop frames. An Exit: means Prolog successfully found an answer, while a Fail: means it has run out of possible answers for that particular goal. For example, Fail: parent(liz, X) shows that Prolog could not find a child for Liz. However, finding one answer does not immediately end the search because Prolog can still go back and look for more.


## What does a `Redo` line resume that a returned frame cannot?

With a normal call stack, once a function returns its result, its frame is gone. Prolog is different because it can revisit something it already checked and try another possibility. This is possible because it keeps track of alternative clauses through choice points, including the variable bindings before the previous attempt.

As seen in the trace, Prolog first finds ann as an answer. When another answer is requested, it goes back to parent(bob, X) and tries again, finding pat as the second answer. It does not have to start everything from the beginning because it still remembers the other possibilities it has not checked yet.


## Why does swapping the two clauses of `reach/2` change whether the search ever answers, while swapping the two clauses of `ancestor/2` only changes the order of the answers?

In ancestor/2, swapping the clauses only changes which valid answer branch Prolog explores first. Since the family tree eventually ends and the search reaches its base case, Prolog can finish checking each path and return the answers in a different order.

For reach/2, however, Prolog keeps calling the same rule before even checking if there is a connection between two places. This pushes more frames onto the stack without reaching an Exit: or Fail:. As a result, the stack keeps growing until its limits are reached. Swapping the clauses allows Prolog to check the connections first instead of getting stuck calling itself over and over again.
