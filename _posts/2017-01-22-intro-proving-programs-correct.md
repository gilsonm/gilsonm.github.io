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

Let's start with the definition of **LinkedList**. From *(Sedgewick and Wayne)*

> A *linked list* is a recursive data structure that is either empty (*null*) or a reference to a *node* having a generic item and a reference to a linked list.

Now let's suppose you see this implementation of the definition in a code review

```java
class LinkedList<T>
	class Node
		T item
		Node next

	Node FIRST
```

And to get the last element, a function like the following

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

Mathematical Logic is a tool for pushing truth around. Given some statements we know to be true, we use rules of deduction to find the truth-value of other statements. Such a sequence of deductions is called a proof. Statements for which proofs exist are called theorems. **Can this be applied to programs?** The answer is yes, but with some twists. The work presented herein was created by Robert Floyd and Tony Hoare.

Here's the basic idea. Programs are built up from commands. To reason about programs we first need to reason about commands. We'd like to say that executing command `C` has some consequence, call it `Q`. For example if `C = x++`, we may claim that `C` has the effect of `adding 1 to the current value of x`. This effect is the consequence `Q`. However, the current example is only true under certain conditions. What if `x` is not a numeric type, or in general does not have the autoincrement operator defined? To cover the cases we are interested in, it's necessary to state some assertions of what is true before the command executes. So that, all together, we can conclude

> If the preconditions are met, executing the command establishes the postconditions

This is usually written as `{P} C {Q}` or top-down as 

```
{P}
C
{Q}
```

Such a construct is called a **Hoare Triple** or **Floyd-Hoare Triple**. From this building block we can establish the truth of, not just isolated commands, but entire programs! Hereafter, the set of claims `{P}` will be called the **preconditions**, and the set of claims `{Q}` the **postconditions**.

Let's take a look at a concrete example involving an addition operation. In plain english, we may state **the Triple** as 

> When variable x equals 1 and y equals 2, adding x to y and assigning the result to z results in z equal to 3

Formally, it can be written as `{x=1, y=2} z = x + y {z = 3}` or

```
{x=1, y=2}
z = x + y
{z=3}
```

### A Note on Syntax and Semantics

Just writing (or reading) an Hoare Triple does not mean it is true! First, the Hoare Triple must be syntactically correct to even posess a truth value. If it is not a well-formed formula, it cannot be evaluated! If it is well-formed, however, it is either `true` or it is `false`. Which of these values holds for the Hoare Triple is established by deduction (proof).

Another thing to note is that there is a mixture of languages here. The language of Hoare Logic with statements in `{}`--this is the language you are reasoning *with*--and the targe language, which you are reasoning *about*. Typically, I've seen Hoare Logic expressed as a mixture of the target language and the language of first order logic. Ultimately both languages need formal definitions.

![lang schematic](https://github.com/gilsonm/gilsonm.github.io/raw/master/languages.png)

In addition to a formal definition of the language of Hoare Logic, there needs to be a formal definition of the deductive calculus of Hoare Logic. The formulas and rules of the deductive calculus allow us to establish the truth-values of our Hoare Triples.

We can now model our knowledge about the program in a **formal language**, and use a **deductive calculus** to prove theorems about its behavior.

### Relationship to Testing

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

If we continued to write passing test cases, we would continue to establish Hoare triples over our sub program. However, unless we exhausted the input space, this collection of tests/triples would not guarantee correct behavior over all inputs. There is a problem of induction here that is identical to the problem of induction in Science, see `**Karl Popper (1963). Conjectures and Refutations. p. 128. ISBN 0-06-131376-9**`

If we’re testing a function that takes two ints as input, we really want to establish the Hoare triple

```java
{int x, int y}
z = x + y
{z = x + y}
```

We can verify this triple via the deductive calculus, as long as we correctly specify our preconditions and post conditions. 

As a caveat, if we ran the code on any real-world system we would certainly run into overflow scenarios that invalidate the postcondition since memory is bounded. In such cases, it would be useful to model the language or runtime specific constraints in the Hoare logic such that the reasoning translated to the target system(s).

### Establishing correctness of the LinkedList traversal...

Now back to the LinkedList... We were interested in demonstrating that the following algorithm correctly returns the last element from a **LinkedList**:

```java
T last()
	if FIRST == null
		return null

	node = FIRST

	while node.next != null
		node = node.next
	return node.item
```

Let's take a first stab at re-writing as an **Hoare Triple**. We'll drop the function signature and first focus on the while loop. There will be a bit of slop as we go over concepts but will tighten up our arguments significantly by the end of the post. 

Starting from the general schema for a triple:

```java
{P}
C
{Q}
```

We substitute in our program fragment:

```java
{P}
while node.next != null
	node = node.next
{Q}
```

And add our preconditions and requirements (postconditions):

```java
{node = FIRST}
while node.next != null
	node = node.next
{node = LAST}
```

Now let's break apart the loop and figure out how to reason about it. There's at least two important parts

- The condition
- The loop body

We can model this like

```java
{P}
while (C) 
	S
{Q}
```

Where `C` is the condition of the `while` loop, and `S` is the body (statements). We desire that given preconditions, `{P}`, the while loop with condition `C` and body of statements `S` guarantees our goal, `{Q}`.

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

Schematically this is like:

![loop schematic](https://github.com/gilsonm/gilsonm.github.io/raw/master/loop_invar.png)

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

Great! We're home free. But what about cases like..

```java
LinkedList l = new LinkedList()
Node n = new Node()
n.next = n
l.first = n
```

We will never have termination given the current algorithm! Also it seems that we need to put constraints on what types of inputs we receive. These constraints can be stated as preconditions, and later be backed up by implementations.

As an aside, looking back at the definition of **LinkedList** from the beginning of this article, does it disallow cycles? On close inspection, I would say that *it allows cycle*. 

How do we handle all of this? We could start by making all sorts of claims about the input data, and then work forward from those claims towards the postcondition, or goal, `Last(node)`. This is generally not a good idea, even though it is intuitive! A better approach is to start from the goal, and work backwards to figure out which preconditions are logical necessities for the post condition.

So with this goal
* `Q` = `Last(node)`
We will work towards these invariants
* `I` = `l[k] = node`
And ultimately define what we need in our preconditions. 

So this is our current schema

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

Great! But what do we do with the list preconditions, `{L}`? One way to handle those would be create `add(T item)` and `remove(T item)` with the statements in `{L}` as postconditions. Also, we could make `Node` private and enforce all `LinkedList` creation and manipulation through its public interface. After establishing the guarantees (postconditions) of the various public methods, you would then prove that any `LinkedList` created and modified through the public API has certain properties, as a result of the public method postconditions. That's for another post! Maybe...

## (#References)

