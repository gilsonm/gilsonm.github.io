---
layout: post
title: Formal Reasoning about Program Behavior. An Introduction.
---

# What it do? 

## Formal Reasoning about Program Behavior. An Introduction.

Michael Gilson *January, 2017*

---

## 1. Motivations

Say we wanted to return the last item in a **LinkedList**. Easy right?

First, let’s be clear on what is a **LinkedList**.  

> A *linked list* is a recursive data structure that is either empty (*null*) or a reference to a *node* having a generic item and a reference to a linked list.  From *(Sedgewick and Wayne)*

From that definition, let's suppose you see this implementation in a code review

```java
class LinkedList<T>
	class Node
		T item
		Node next

	Node FIRST
```

And to get the last element, is a function like the following

```java
T last()
	if FIRST == null
		return null

	node = FIRST

	while node.next != null
		node = node.next
	return node.item
```

Looks right. But *is it*?  More specifically, under what conditions is it correct? Is there some way to **formally prove** that the code behaves as required? We could trace the function by hand on several small inputs to convince ourselves. We could even write tests over a large number of inputs, for which we knew the correct answer, and thereby demonstrate "some correctness." Thinking to ourselves, 

> ”Well if it works on all these inputs that I tried, how could it possibly fail on the ones I didn't try -- they're basically the same!"

As we'll see later, tests are actually **Hoare Triples** with extremely strong (specific) pre-conditions. They verify program behavior on some number of specific inputs. But unless every possible input is tested, we cannot claim that the program is correct.

Under what circumstances might we be interested in this level of certainty?

* Mission- or life-critical code
	* This is increasingly true as we move forward with automation, e.g. 
		* self-steering vehicles
		* home and building sensor/control systems
		* medical and surgical devices
* You must convince someone else your code is free of bugs
* Distributed systems
	* See AWS success with TLA+

So if tests, in general, only establish correct behavior for the given input cases, how can we go about proving general claims about program correctness over all inputs?

We can consider the various fields that are required to justify claims, and try to learn something from them and apply it to software behavior. E.g.

* Legal
* Exact Sciences
* Mathematics
* Philosophy

In my experience, most software correctness arguments follow along the lines of the Scientific Method. That is, a hypothesis is put forth, and it stands until it is falsified. In software, this hypothesis is the claim that our program does what is expected. The claim may be partially motivated by a suite of passing tests. And the claim stands until it's falsified by a runtime bug!

Surely we can do better.. in the exact sciences, we make claims about the natural world, whose inner workings or rules are mostly concealed to us. So we put forth hypotheses, given what evidence or prejudice is at hand, in a state of general ignorance of the system under consideration, and devise experiments that can falsify these hypotheses or any of their consequences.

Software, however, is fundamentally different. It runs on man-made devices. We know the inner workings of the host system to exquisite detail. Really, the software situation smells more like mathematics than basic science. So how are claims verified in mathematics? Mathematical Logic! Is there a correlate for programs?

### Floyd-Hoare Triples
---

What tools have been created to reason about our programs in a formal way?

Well, there's already Mathematical Logic that allows us to express claims about algebraic systems and perform deductions from sets of claims, axioms, and logical truths. **Can this be applied to programs?**

The answer is yes, but with some twists. Thanks to Robert Floyd and Tony Hoare for coming up with a formal system.

Here's the basic idea. Programs are built up from commands, and we'd like to make claims of the form

> Given that the program is in state, P, and I execute command C, I then expect the program to be in state Q

or in the words of Wikipedia

> When the precondition is met, executing the command establishes the postcondition

This is usually written as `{P} C {Q}` or top-down as 

```
{P}
C
{Q}
```

The set of claims `{P}` is called the **preconditions** and the set of claims `{Q}` is called the **postconditions**.

Here's an example in which we reason about incrementing a number stored in a variable. In plain english

> When variable x equals 1 and y equals 2, adding x to y and assigning the result to z results in z equal to 3

Which looks like `{x=1, y=2} z = x + y {z = 3}` or

```
{x=1, y=2}
z = x + y
{z=3}
```

> Note: Actually demonstrate that the triple is valid? Or use a simpler example, possibly from Aspnes

**Note**: A simple decision tree can be useful for analyzing Hoare Triples that you and others write. First, is the Hoare Triple a well-formed formula? If not, then it's not true. If it is a well-formed formula, then we can start the business of establishing whether or not the Triple is provable or "True"

Fantastic! 

We can now model our knowledge about the program in a **formal language**, and use a **deductive calculus** to prove theorems about its behavior.

### Relationship to Testing
---

There is a connection between Hoare Logic and modern software testing.  Given some test like 

```java
void testAdd() {
	int x = 1
	int y = 2

	assertEquals(
		3,
		x + y
	)
}
```

If it passes, then the following Hoare triple is true

```java
{x=1, y=2}
z = x + y
{z=3}
```

> Note, we introduced an auxiliary variable, z, for the sake of writing a post-condition. Also, our precondition should probably include a statement that f is a new Foo instance that has not been accessed before the call to bar(int,int)

However, we haven’t arrived at this result by a formal deduction. Instead, we evaluated a program with very strong pre- and post-condition. Which is to say, we evaluated the program on a specific input value and asserted that the result was a specific value.

If we continued to write passing test cases, we would continue to establish Hoare triples over our sub program. However, unless we exhausted the input space, this collection of tests/triples would not guarantee correct behavior over all inputs. There is a problem of induction here that is identical to the problem of induction in Science, see Popper.

If we’re testing a function that takes two ints as input, we really want to establish the Hoare triple

```java
{int x, int y}
z = f.bar(x,y)
{z = x + y}
```

We can verify this triple via the deductive calculus, as long as we correctly specify our preconditions and post conditions. 

### Back to the thing

Now back to the LinkedList... We were interested in demonstrating that the following algorithm correctly returns the last element from a **LinkedList**:

```java
Object targetItem
Node node = FIRST

while node.next != null
	if node.item.equals(targetItem)
		return true
	node = node.next
return false	
```

One thing to note here is that we have more than a single command in the body of the **Hoare Triple**, indeed we have a while loop. This is common and doesn't change how we interpret the **Triple**. We could augment the definition from wikipedia by pluralizing `command` to be

>When the precondition is met, executing the **commands** establishes the postcondition

So how do we tackle this loop?

Let's take a first stab at re-writing as an Hoare Triple

```java
// Starting from the general schema for a triple:

{P}
C
{Q}

// Substituting in our program fragment:

{P}
while node.next != null
	node = node.next
{Q}

// And our preconditions and requirements (post conditions):

{node = FIRST}
while node.next != null
	node = node.next
{node = LAST}
```

Now let's break apart the loop and figure out how to reason about it.

There's at least two important parts

- The condition
- The loop body

We can model this like

```java
{P}
while (C) 
	S
{Q}
```

We desire that given preconditions, `{P}`, the while loop with condition `C` and body of statements `S` guarantees our goal, `{Q}`.

A time-tested method for situations like the present is to use **an Invariant**. This is some property (a predicate), let's call it `{i}` that holds true at the beginning and end of each pass through the body, and together with the termination criteria, will imply `{Q}`. E.g. with a schema like

```java
{P}
while (C) 
	{I}
	S
	{I}
{Q}
```

our reasoning is

```
{P}, {I}, and !C imply {Q}
```

The `!C` is there because the loop has ultimately terminated, so the loop condition, `C`, is false, viz. `!C`. (We also have to prove that the loop terminates, but let's not get ahead of ourselves)

> If you are not used to formal logic or invariants, this should feel unituitive. Only after grappling with many examples does this subtle method reveal its strength.

Invariants are strange beasts. At first an invariant claim seems "trivial" since it's true before and after applying some operations. Indeed, the value `True` is an invariant of every terminating sequence of operations. However, we can make little use of it. There's an *art* to *finding* the right invariant. And in fact, there have been several attempts in the literature to provide a more systematic approach to invariant discovery.

This may help. If our loop is making progress of some sort, by traversing an array for instance, our invariant may be some property about the part of the array we've already processed.

Indeed, if we were implementing a sorting algorithm like Insertion Sort, where each iteration of the loop assures that the sub-array processed so far is sorted, is one such invariant.

> Possibly include a schematic diagram ..

So how do we find an invariant for our LinkedList traversal? Given our current schema

```java
{P}
while (C) 
	{I}
	S
	{I}
{Q}
```

which applied to the LinkedList algorithm is

```java
{node = FIRST}
while node.next != null
	{I}
	node = node.next
	{I}
{node = LAST}
```

If we work backwards from the target condition, `{Q}`, which here is `{node = LAST}`, this nudges us in the direciton of involving some property of the `node` variable in the invariant, `{I}`.

What do we know about the variable `node`? Let's start with the obvious or trivial, and if we can't find our solution there, only then should we start getting clever. 

One trivial claim that certainly is true is `node is either the last node or is not the last node`. Although trivial, this is not a bad starting point. We just need to sharpen the claim a bit. To start, let's ask

>when is it the last node and when is not? 

Well, on the first iteration of the loop, it's the FIRST node. On the second iteration, it's the second node. And so forth. So on the kth iteration, it's the kth node in the list.

```java
// Iterations of the loop

Command						Iteration
-------						---------
while node.next != null		0
node = node.next
while node.next != null		1
node = node.next
while node.next != null		2
node = node.next
...
while node.next != null		k
node = node.next
...
while node.next != null		n
node = node.next
```

## Proof of traversal

Somehow demonstrate that we're traversing the list. Using pointers rather than array indexing. May have to establish some pre-condition claims about lists, such that the repeated calls to .next visit each node.

E.g. How do we know we didn't skip a node? *(Show proof)*

E.g. Prove that kth iteration is kth node in list

Add precondition that FIRST is not null, and is of type Node.

Every instance of Node, .next is either

- null
- a Node instance or sublcass instance

**But what about cases like**

```java
Node n = new Node();
n.next = n;
```

We will never have termination given the current algorithm!

So now it seems useful to introduce a pre-condition like one of

1. We're given a list created by a specific program (e.g. one that has "add node" which ensures [provably] only linear linked-lists)
2. We just state it directly as a pre-condition. Like {No cycles in list; at least one null in list} etc.

For 1. We could do something like

```java
void addNode(T item) {
	Node n = new Node();
	n.item = item;

	// but now we have to add to the end of the damned list!
	LAST.next = n
	LAST = n
}

// we do this instead of

void addNode(Node n) {
...
}

// Since n may already be in list.

// Allow duplicate Items, not Nodes
```

## Termination

```
L1.		Every list is of bounded length
P(k). 	node = kth node in list

L1 -> L2. There exists an integer, n, that is the length of the list

Consider iteration n of the loop.
P(n) holds.
P(n) -> !C (*)
!C -> termination

* Detailed proof of that step:

L3. In a list of length n, the nth node is the last node
^ have to prove by induction. Base P(1). P(k)->P(k+1) using addNode code.
P(n): node = nth node in list
List of length n -> node is last.
Last node iff no next node. No next node iff node.next == null.
```

# Proving a LinkedList Traversal Correct
> Use a different color for the Hoare blocks  

Say we want to return the last item from a LinkedList. How would we write that program, and importantly, how would we verify that the program was correct?

```java
node = FIRST
while node.next != null
	node = node.next
return node.item
```

Will work towards these invariants
* `I` = `l[k] = node`

With this goal
* `Q` = `Last(node)`

And we’ll have to figure out our preconditions. So this is our current schema

```java
{P}
node = FIRST
{I}
while node.next != null
	{I}
	node = node.next
	{I}
{Q} = {Last(node)}
```

In the body, `{I and C}` hold, and after the while `{I and !C}` hold.

```java
{P}
node = FIRST
{I} = {l[k] = node}
while node.next != null
	{I^C}
	node = node.next
	{I}
{I^!C}
{Q} = {Last(node)}
```

Lets first ensure one part of the invariant, `{I}`. To make `{I}` true before the loop,  let’s focus on

```java
{P}
k = 1
node = FIRST
{I1} = {l[k] = node}
```

We know that `k=1`  so

```java
{P}
k = 1
{k=1}
node = FIRST
{I} = {l[k] = node} = {l[1] = node}
```

Since we’re assigning `FIRST` to node, whatever properties of node we desire, we can require of `FIRST`, and then by assignment axiom, the properties will hold of `node`.

```java
{P1} = {l[1] = FIRST}
k = 1
{k=1}
node = FIRST
{I1} = {l[k] = node} = {l[1] = node}
```

So lets apply the assignment axiom twice to go from `{P1}` to `{I1}`

```java
{P1} = {l[1] = FIRST}
k = 1
{P3} = {P1(1|k)} = {l[k] = FIRST}	 // by Assignment Axiom
node = FIRST
{P3(FIRST|node)} = {l[k] = node}	// by Assignment Axiom
```

Now the inductive part. If `{I1}` holds at the beginning of the while body, it must hold at the end. E.g. `{I1}` is both a pre- and post-condition of the body. I.e.

```java
{I1}
node = node.next
k = k + 1
{I1}
```

So let’s suppose `{I1}` holds at the beginning, which is to say `l[k] = node`. We need some theorem that connects the `.next` reference to the `k+1st` entry in the list. That is, we need

```
l[k] = n and n.next != null -> l[k+1] = n.next
```

This is a property of lists, and is a precondition of the while, let’s call it `P2`. The two conjuncts amount to `C` and `I1`, which both hold, so we are guaranteed that the consequence, `l[k+1] = n.next` holds. So we can establish that Triple/post-condition of the node assignment:

```java
{I1} = {l[k] = node}
node = node.next
{S1} = {l[k+1] = node}
k = k + 1
{I1}
```

The assignment to `k` re-sets our invariant!  That is to say, `S1(k+1|k)` is true given the Assignment Axiom, and it equals `I1`. To restate, we’ve established that `S1` is true given the inductive hypothesis `I1`, the loop guard `C` and `P2`. Then we have an assignment of `k+1` to `k`. `S1` is a theorem involving `k+1`, and given the Assignment Axiom, we can create another true statement by substituting `k` for `k+1` in `S1`, which is called `S1(k+1|k)`. And `S1(k+1|k) = I1`, which completes the proof of our invariant.

We’ve established our invariant `I1`, now let’s see how close we are to proving `{Q}={Last(node)}`.  Upon termination the loop guard `C` is false, so is `!C`. And our invariant `I1` holds, so we have `I and !C` coming out of the loop. Which is `I^!C = l[k] = node ^ node.next = null`. From this, can we conclude `{Last(node,l)}`? What is the definition of `Last(node,l)`?

```
Last(node,l) iff 
Q1. node in l
Q2. node != null
Q3. node.next == null
```

So we have to prove `{Q1, Q2, Q3}`. We can prove `Q1` by adding a similar precondition, say `P4`, like `l[k] = node ^ node.next != null —> node.next in l`. To prove `Q2`, this is actually an invariant of the loop so long as initially `node != null`. Why? `node` is only ever assigned the value `node.next`, which is guaranteed to `!= null` because of `C`.  We may require that `FIRST != null` to establish that. Finally, `Q3` is equivalent to `!C`. 

Why include `k`? It’s there as a convenience to the proofs and not used in the algorithm itself. We’ll want to establish two addition claims. Since what we’ve done is a partially correct proof. We need to establish **Termination**  and an additional invariant `For all j < k, !Last(l[j])`. 

To establish **Termination**, we can argue as follows

* Given. Every list `l` has a finite number of nodes, `n`
* `l[n].next = null`
* When `k = n`, `l[k].next = null`, therefore `!C`
* So show `k = n` on some iteration of the loop
	* Well, `k = 1` at first
	* Then each iteration sets `k = k + 1`
	* Show that at some iteration `k = n - 1`
	* Let `g = n - k`
	* Show that `g` decreases by `1` with each iteration, call it `T1`. Proof in code box below.
	* We don’t know that the entire **while** loop terminates, but we do know that each statement terminates
		* This implies that iterations **will** happen indefinitely or until `!C` (Provable from axioms)
		* Suppose `!C` is satisfied, then termination
		* Suppose `!C` is never satisfied. In this case we execute every statement in a single iteration, over-and-over. Since the loop body is executed on each iteration, `g` is decreased by 1 on each iteration. `g` starts its life equal to some `j > 0 since n - k > 0` and `k = 1` so `j = n - 1`. After iteration `i`, `g` has been decremented by `1`,  `i` times. Therefore `g = j - i`.  So on iteration `i = j`, `g = 0`. But `g = n - k`, so `0 = n - k`, which implies `n = k`.  By the invariant, `l[k] = node`, which implies `l[n] = node`, but `l[n].next = null` as a given property, therefore `node.next = null`, therefore `!C` and the loop terminates!

```java
Let n > 0
k = 1
g = n - k
node = FIRST
{g = n - 1 }
while C
	node = node.next
	{ g = n - k }
	k = k + 1
  g = n - k
	{T1} = { g = n - (k + 1) = n - k - 1 = (n - k) - 1 = g - 1 }
{ g = 0 }
```

Putting together the **partial correctness** proof and the **proof of termination** we get the following

```java
{L} = {
	For every non-empty LinkedList, l

	L0. l[k] = node iff node is kth node in LinkedList, l
	L1. l[1] = FIRST
	L2. l[k] = node and node.next != null imply l[k+1] = node.next
  L3. l[k] = node and node.next != null imply l[k+1] in l
	L4. len(l) = n for some n > 0
  L5. len(l) = n implies l[n].next = null
}

{P} = {FIRST != null, FIRST in l, len(l) = n > 0}
k = 1
g = n - k
node = FIRST
{I} = {l[k] = node}
while node.next != null
	{I ^ C}
	node = node.next
	k = k + 1
  g = n - k
	{I}
{I ^ L1..L5 ^ !C} -> {Q} = {Last(node,l)}
```
