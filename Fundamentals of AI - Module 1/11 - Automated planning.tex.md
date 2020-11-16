# Automated planning

**Automated planning** is a problem solving activity which consists in synthetising a **sequence of actions** performed bu an agent that leads from an inital state of the world to a given target state. 

In other words, it is devoted to create a **plan** from an initial state to a final state.

Let's move step by step: we want to reach a **goal**, e.g. we have a robot that has to be moved to another room. Apparently, this is something that we do everyday: trips, routes, events.

While it is basically an *obvious activity*, it is quite complex for a machine: it is **semi-decidable**, so if a plan exists it's possible to find it, but if it does not exist we could end in an infinite loop.

An important notion is that of the **action**: it depends on the kind of moves that you can do.

We start with: an *initial state*, a *set of actions we can perform*, and a *state to achieve*. We want to find **a plan**: a partially or totally ordered set of actions needed to achieve the goal from the initial state. Planning is one application per se, or a common activity in many applications such as *diagnosis, scheduling, robotics*...

An **automated planner** is an intelligent agent that operates in a certain domain described by a *representation of the initial state*, a *representation of the goal* and *a formal description of the executable actions*. I repeat: we need a **non-ambiguous, formal** way of describing these.

## Action representation

The planner relies on the formal description of the action, called **domain theory**. We want to express two list of properties: the **preconditions of the action**, i.e. the set of properties that should be true for the action to be executed (e.g. to drink a glass of water, there should be water inside it), and the **postconditions**, which represent the effects of the action on the world.

## Planning

When we want to create a planning solution, we can say that it is **non decomposable** (there can be interaction between subgoals, i.e. if we have more goals these are obviously interacting; this will be a problem for linear solvers) and **reversible** (i.e. the choices made during the plan are backtrackable).

Note that the process is **offline**: you have a very tight hypotesis, i.e. **nobody else is changing the world**.

A planner is **complete** if it always finds a plan, given that a plan exists. 

A planner is **correct** when the solution found is correct, i.e. the solution leads from the initial state to the goal.

The **exectution** is generally **irreversible** (often the execution of an action is not backtrackable, e.g. I can drink a bottle of water but can't put it back) and **non-deterministic**, i.e. the plan can have effects that are different from what we expect: working in the real world, the effects are not deterministic/totally predictable.

## Generative Planning

This is an off-line planning that produces the whole plan before execution: this is cool, we can always restart if we don't find a solution!

### Planning as search

Planning can be seen as a search activity! At the end of the day, though, we'll resort to special purpose algorithms because general purpose search is often too complex.

The search algorithm could proceed **forward** (starting from the initial state and finding a state which is a superset of the goal) or **backward** (starting from the goal and finding a state which is subset of the initial state).

The latter is basically **goal regression**, i.e. a mechanism to reduce a goal in subgoals during search by applying rules.

## Deductive planning

The previous two are never used, because they have a problem: *we still don't know what that is, tho.*

**Deductive planning** is a general purpose type of planning, based on *First Order Logic*, which describes the initial states, the actions and the goal.

The **situation** is a world snapshot describing properties (*fluents*) that hold in a given state $s$.
$$
Example: block\ world\\
o n(b, a, s)\\
ontable (c, s)
$$
The **actions** define which fluents are true as a consequence of an action. Note that uppercase letters are constants. The *put on table* action might be defined as follows:
$$
\begin{aligned}
&\begin{array}{l}
\text { on }(X, Y, S) \text { and } \operatorname{clear}(X, S) \rightarrow \\
\text { (ontable }(X, d o \text { (putOntable }(X), S))) \text { and }
\end{array}\\
&\text { (clear(Y,do(putOn Table }(X), \text { S }) \text { ) }
\end{aligned}
$$
The action *put on table* has two preconditions, before the arrow, and consequences after.

Given the situation calculus (initial state, actions) we can use *resolution* (i.e. the traditional way we use in logic to solve problems) to create a plan. We'll use  *unification* too, as in logic. 

If, for example, we ask *Is it possible to find an $s$ where $b$ is on the table?* We can use our resolution to put $b$ on top of the table, because it is on top of it and the table is clear. 

The resolution tells us that it is true that you can achieve the required state.

### Frame problem

We have to explicitly list all fluents that change and **do not** change after the state transition. We can't just state those that change! We have to bring all of them to the next level.

To solve this, we can transform the implications with *not A, or B*,. **We take the whole implications negged and orred, orred with one of the conclusions, then the other one.**

Resolution works by finding contradictions: we negate the goal and we try to find the empty clause, by unifying the clauses.

Since we have to move all of the fluents, we need a frame aziom for each condition that is not changed by each action! To describe an action, we therefore need all the fluents. This means that **if the problem is too complex, we have too many axioms!**

The frame axioms specify, for all the possible fluents, that if they do not change thanks to the action, they have to stay right where they are.

### Kowalski formulation

We use a predicate $holds(rel,s/a)$ to describe all the true relations in a state $s$ or made true by an action $a$. Then, we have a preidcate $poss(s)$ that states if a state is *possible* (i.e. reachable). Finally, a predicate $pact(a,s)$ to indicate that it is possible to execute an action $A$ in state $s$, namely the preconditions of $a$ are true in $s$. 

So, we need one frame assertion per action (which is good compared to Green).

We have moved the property as **terms**, which can be done in PROLOG etc. We say that the initial state is reachable, and in the initial state $s_0$ we list the properties that hold. For the action *move*, we have two facts that describe the effects. So we can say that for every state $s$, if we apply the action *move*, it holds *clear(Y)*...

When you have the effects, you have to use one single fact for all of the effects. For the preconditions, we use $pact$ (*precondition actions*).  

## STRIPS

**Stanford Research Institute Problem Solver** is a specific language for the actions, having an easier syntax and an ad hoc algorithm for the plan construction. The state is represented through the fluents that are true in a given state, and the goal is represented as the fluent that are true in the goal state. 

The **action representation** is composed of 3 components: 

- *preconditions*;
- *DELETE list* (fluents that become false after the move);
- *ADD list* (fluents becoming true after the move).

### Algorithm

First of all, as for all the other planners we have seen so far, **STRIPS is a generative planner** (i.e. nobody else is changing our world while we search). Remember that what we state in the state is true, what we don't is false. We have two data structures: the **goal stack** and the **description of the current state**, which obviously starts from the initial one. The goal stack **proceeds backward**, while the current state **proceeds forward**. Here we have the two components: it applies the **goal regression** and the **the state forward search**. These two structures should *meet*.

- We initialize the stack with the goals to reach
- At each step
  - We remove the top from the stack and analyze it. The thing is simple: there can be different things here. Initially, we just have a goal. We can then check if the goal can be unified (the unified is stated as $\theta$) with the *current state*. We remove the goal, because it is true, and apply the unification on the other stack. If it is not true in the current state, we have to find an action that achieves this goal! How can we find it? Where should this goal unified with the other stack? We have to find an action where this $A$ is in the *ADD list*. 
  - Then, you remove the goal from the stack, you insert the action and all the preconditions in the *GOAL stack*
  - Finally, having pushed the action on the stack, this will emerge when all the preconditions are true.
  - This is the only time where you change the state description by **applying this action**!

### Example

The only thing that emerges from the goal stack is an AND. Note that the order in which we put these goals is extremely important. So, now, we extract $on(a,c)$ and we ask ourselves if it is already true in the initial state. The answer is no: we extract it from the stack, and have to find an action that does so, which will be $stack(a,c)$, and *all its preconditions*. We put the action and the precondition in the goal stack, where $on(a,c)$ was. Now, we order the preconditions and leave an AND. $holding(c)$ is not true, so we have to find an action that makes that true! We then substitute $holding(c)$ with the preconditions and action that makes that true. This proceeds until we find the preconditions in the initial state! When that happens, we can apply the action in the state stack! We therefore add to the DELETE list $handempty$ and $on(c,a)$, and ADD $holding(c)$. So, we remove from the goal stack what has to be removed, and so on....

### Some notes

Up to now, having found a conjunction of goals, we consider them to be independent. If my goal is putting on my shoes, it is independent if I put the left one first or the right one. But if the goal is putting the socks on then the shoe, it's a problem. **There are cases in which the goals are not independent**: they interact with each other. It may, for example, happen that you reach a goal, and destroy the other one. Every time we select a conjunction (AND) of goals, you have to keep it on the stack! After achieving each of them separately, it may be the case that the AND is not achieved too! Note that the order in which subgoals are inserted is a *non-deterministic* choice point.

The *Sussman anomaly* is a good example of this.

The **search space is very large**, and we can solve this by using heuristics like the *means-ends analysis*.

### What we'll find in the exam

We always find one exercise that is either a **STRIPS**, asking to explore one **single** path from the starting state to the goal, or a **POP**, *partial order planning*. Then, the theoretical questions might be like the *Kowalski formulation* (express, given the domain theory, the initial state and one possible move in the Kowalski formulation), and one on graph plan.


