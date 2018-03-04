# The π-calculus

The π-calculus is a successor of CCS that tries to additionally model the dynamically changing communication topology.
This is often called _mobility_.
To enable mobility messages can carry names.


## Motivating Example

Let us look at a simplified version of the GSM handover protocol from [The Polyadic pi-Calculus: A Tutorial](http://www.lfcs.inf.ed.ac.uk/reports/91/ECS-LFCS-91-180/).
The system is composed of:
* a car that moves around,
* two base stations which act as relay for the communication to the car,
* a switching center which tries to keep the connection to the car though one of the base station.

Intuitively, the system looks like:
```
     ┌───┐
     │Car│
     └───┘
 talk₁||switch₁
    ┌────┐        ┌────────┐
    │Base│        │IdleBase│
    └────┘        └────────┘
 alert₁\\give₁     //
        \\   give₂//alert₂
         ┌────────┐
         │ Center │   
         └────────┘
```

```
Car(talk, switch) ≝ ?talk.Car(talk, switch)
                    + !talk.Car(talk, switch)
                    + ?switch(talk′, switch′).Car(talk′, switch′)
Base(talk, switch, give, alert) ≝ ?talk.Base(talk, switch, give, alert)
                                  + !talk.Base(talk, switch, give, alert)
                                  + ?give(t, s).!switch(t, s).IdleBase(talk, switch, give, alert)
IdleBase(talk, switch, give, alert) ≝ ?alert.Base(talk, switch, give, alert)
Center(t₁, t₂, s₁, s₂, g₁, g₂, a₁, a₂) ≝ !g₁(t₂, s₂).!a₂.Center(t₂, t₁, s₂, s₁, g₂, g₁, a₂, a₁)
```


And the initial state of the system is:
```
(ν talk₁ talk₂ switch₁ switch₂ give₁ give₂ alert₁ alert₂)(
    Car(talk₁, switch₁) |
    Base(talk₁, switch₁, give₁, alert₁) |
    IdleBase(talk₂, switch₂, give₂, alert₂) |
    Center(talk₁, talk₂, switch₁, switch₂, give₁, give₂, alert₁, alert₂)
)
```

Let us look at what happens during the handover.

By expanding some definitions from the initial state we get:
```
(ν talk₁ talk₂ switch₁ switch₂ give₁ give₂ alert₁ alert₂)(
    (?talk₁.Car(talk₁, switch₁) + !talk₁.Car(talk₁, switch₁) + ?switch₁(t, s).Car(t, s)) |
    (?talk₁.Base(talk₁, switch₁, give₁, alert₁) + !talk₁.Base(talk₁, switch₁, give₁, alert₁) + ?give₁(t, s).!switch₁(t, s).IdleBase(talk₁, switch₁, give₁, alert₁)) |
    ?alert₂.Base(talk₂, switch₂, give₂, alert₂) |
    !give₁(talk₂, switch₂).!alert₂.Center(talk₂, talk₁, switch₂, switch₁, give₂, give₁, alert₂, alert₁)
)   
```
after synchronizing on `give₁` we have:
```
(ν talk₁ talk₂ switch₁ switch₂ give₁ give₂ alert₁ alert₂)(
    (?talk₁.Car(talk₁, switch₁) + !talk₁.Car(talk₁, switch₁) + ?switch₁(t, s).Car(t, s)) |
    !switch₁(talk₂, switch₂).IdleBase(talk₁, switch₁, give₁, alert₁) |
    ?alert₂.Base(talk₂, switch₂, give₂, alert₂) |
    !alert₂.Center(talk₂, talk₁, switch₂, switch₁, give₂, give₁, alert₂, alert₁)
)   
```
after synchronizing on `switch₁` we have:
```
(ν talk₁ talk₂ switch₁ switch₂ give₁ give₂ alert₁ alert₂)(
    Car(talk₂, switch₂) |
    IdleBase(talk₁, switch₁, give₁, alert₁) |
    ?alert₂.Base(talk₂, switch₂, give₂, alert₂) |
    !alert₂.Center(talk₂, talk₁, switch₂, switch₁, give₂, give₁, alert₂, alert₁)
)   
```
after synchronizing on `alert₂` we have:
```
(ν talk₁ talk₂ switch₁ switch₂ give₁ give₂ alert₁ alert₂)(
    Car(talk₂, switch₂) |
    IdleBase(talk₁, switch₁, give₁, alert₁) |
    Base(talk₂, switch₂, give₂, alert₂) |
    Center(talk₂, talk₁, switch₂, switch₁, give₂, give₁, alert₂, alert₁)
)   
```

Visually, the new state is:
```
                    ┌───┐
                    │Car│
                    └───┘
                talk₂||switch₂
    ┌────────┐     ┌────┐
    │IdleBase│     │Base│
    └────────┘     └────┘
 alert₁\\give₁     //
        \\   give₂//alert₂
         ┌────────┐
         │ Center │   
         └────────┘
```


## Syntax

__Process definitions.__
A set of mutually recursive definitions of the form:
```
A(a) ≝ P
```
where
* `A` is the name of the process
* `a` is a list of _names_
* `P` is a process

__Processes.__
```
P ::= π.P       (action)
    | P + P     (choice)
    | P | P     (composition)
    | (νa) P    (restriction)
    | A(a)      (named process, a can be a list of agruments)
    | 0         (end)
```

__Actions.__
```
π ::= !a(a)     (send/output)
    | ?a(a)     (receive/input)
    | τ         (silent)
```

_Remark._
The difference between CCS and the π-calculus is the number of names in input/output prefix:
* when no name are involved: CCS
* with 1 name in the prefix: _monadic_ π-calculus
* with >1 name in the prefix: _polyadic_ π-calculus

We can encode CCS in the π-calculus by sending a dummy name that is never used to synchronize.

To encode the polyadic π-calculus into the monadic π-calculus, we can serialize the argument over a private name:
* `!a(b₁ … b_n)` becomes `(ν arg)!a(arg).!arg(b₁).…!arg(b_n)`
* `?a(b₁ … b_n)` becomes `?a(arg).?arg(b₁).…?arg(b_n)`


## Free names and bound names

Compared to CCS, the bound and free names have to account for message reception as it binds name.

The free names (`fn`) are the names that occurs in a processes but are not restricted:
* `fn(0) = ∅`
* `fn(τ.P) = fn(P)`
* `fn(!a(b).P) = fn(P) ∪ {a, b}`
* `fn(?a(b).P) = fn(P) ∪ {a} ∖ {b}`
* `fn(P + Q) = fn(P) ∪ fn(Q)`
* `fn(P | Q) = fn(P) ∪ fn(Q)`
* `fn((νa)P) = fn(P) ∖ {a}`
* `fn(A(a)) = {a}`

The dual of free names are bound names (`bn`).
The names occurring under a restriction or an input prefix:
* `bn(0) = ∅`
* `bn(τ.P) = bn(P)`
* `bn(!a(b).P) = bn(P)`
* `bn(?a(b).P) = bn(P) ∪ {b}`
* `bn(P + Q) = bn(P) ∪ bn(Q)`
* `bn(P | Q) = bn(P) ∪ bn(Q)`
* `bn((νa)P) = bn(P) ∪ {a}`
* `bn(A(a)) = ∅`

__No clash assumption.__
Like for CCS, we assume that `fn(P) ∩ nb(P) = ∅`.


## Structural congruence

The structural congruence rules are the same as in CSS with the addition of renaming of the names bound by input prefixes:
- `c ∉ fn(P) ∧ c ∉ bn(P) ⇒ ?a(b).P ≡ ?a(c).P[b ↦ c]`


## Semantics

### Labeled semantics (for open or closed world)

The semantics of the pi-calculus is very similar to CCS but it also carries the payload of the messages:

* Silent action
  ```
  ───────────
  τ.P  ─τ→  P
  ```
* Send action
  ```
  ───────────────────
  !a(b).P  ─!a(b)→  P
  ```
* Receive action
  ```
  ──────────────────────────
  ?a(b).P  ─?a(c)→  P[b ↦ c]
  ```
* Choice
  ```
   P ─π→ P′
  ──────────
  P+Q ─π→ P′
  ```
* Parallel
  ```
    P ─π→ P′
  ────────────
  P|Q ─π→ P′|Q
  ```
* Communication
  ```
  P ─!a(b)→ P′  Q ─?a(b)→ Q′
  ──────────────────────────
        P|Q ─τ→ P′|Q′
  ```
* Restriction
  ```
  P ─π→ P′  π ≠ !a(_)  π ≠ ?a(_)  π ≠ !_(a)  π ≠ ?_(a)
  ────────────────────────────────────────────────────
      (νa)P ─π→ (νa)P′
  ```
* Congruence
  ```
  P ≡ P′  P′ ─π→ Q  Q ≡ Q′
  ────────────────────────
          P ─π→ Q′
  ```

_Remark._
Unfortunately, this semantics is not finitely branching.
Consider the process `?a(b).!b.0`.
This process can take infinitely many transitions:
* `?a(b).!b.0 ─?a(c)→ !c.0`
* `?a(b).!b.0 ─?a(d)→ !d.0`
* ...


### Unlabeled semantics (only for closed world)

An infinitely branching semantics can be problem for some type of analysis and people have developed alternative semantics that are finitely branching but only work under the closed-world hypothesis.
In this semantics, the send and receive are directly matched to resolve the name of the payload.
Therefore, all the transition silent and this semantics is unlabeled.

* Silent action
  ```
  ─────────
  τ.P  →  P
  ```
* Parallel
  ```
    P → P′
  ──────────
  P|Q → P′|Q
  ```
* Communication
  ```
  P = … + !a(c).P′  Q = … + ?a(b).Q′
  ──────────────────────────────────
        P|Q  →  P′|Q′[b ↦ c]
  ```
* Restriction
  ```
      P → P′
  ──────────────
  (νa)P → (νa)P′
  ```
* Congruence
  ```
  P ≡ P′  P′ → Q  Q ≡ Q′
  ────────────────────────
          P → Q′
  ```


## More Examples


### Server with unboundedly many clients (names as network addresses)

We saw an example of mobility in a system with a constant number of processes.
However, mobility is particularly useful in systems with many processes.

For instance, consider the following system:
```
Server(s) ≝ ?s(c).(Server(s) | Session(c))
Session(c) ≝ ?c().!c().Session(c)
NewClient(s) ≝ τ.(NewClient(s) | Client(s))
Client(s) ≝ (νc) !s(c).ClientConnected(c)
ClientConnected(c) ≝ !c().?c().ClientConnected(c)
```
with the initial state `(νs)(Server(s) | NewClient(s))`

In this system, a process represent a server and each time a client connects to the server, the server learn a new address and create a session to handle the communication with the client.


### Concurrent list (names as pointers)

Even though the π-calculus was initial targeted at message-passing systems, mobility can express other types of systems.
In particular, names can be seen as references or pointers.


For instance, we can look at the insertion in a lock-coupling concurrent list ([source of the example](https://people.mpi-sws.org/~viktor/cave/examples/LC_set.cav)).
For the sake of simplicity we look at the abstracted version below (no data, infinite list, ...):
```c
 0: void add() {
 1:     Node prev = head;
 2:     prev->lock();
 3:     Node curr = prev->next;
 4:     while(*) {
 5:         curr->lock();
 6:         prev->unlock();
 7:         prev = curr;
 8:         curr = prev->next;
 9:     }
10:     if (*) {
11:         curr->lock();
12:         Node temp = new();
13:         temp->next = curr;
14:         prev->next = temp;
15:         curr->unlock();
16:     }
17:     prev->unlock();
18: }
```

We can encode this in the π-calculus as follow:
```
NodeUnlocked(this, get, set, lock, unlock, next) ≝ !this(get, set, lock, unlock).NodeUnlocked(this, get, set, lock, unlock, next)
                                                   + ?lock().NodeLocked(this, get, set, lock, unlock, next)
                                                   + !get(next).NodeUnlocked(this, get, set, lock, unlock, next)
                                                   + ?set(n).NodeUnlocked(this, get, set, lock, unlock, n)
NodeLocked(this, get, set, lock, unlock, next) ≝ !this(get, set, lock, unlock).NodeLocked(this, get, set, lock, unlock, next)
                                                 + ?unlock().NodeUnlocked(this, get, set, lock, unlock, next)
                                                 + !get(next).NodeLocked(this, get, set, lock, unlock, next)
                                                 + ?set(n).NodeLocked(this, get, set, lock, unlock, n)
Add01() ≝ τ.Add02(head)
Add02(prev) ≝ ?prev(get, set, lock, unlock).!lock().?get(curr).Add04(prev,curr)
Add04(prev,curr) ≝ τ.Add05(prev,curr)
                   + τ.Add10(prev,curr)
Add05(prev,curr) ≝ ?curr(get, set, lock, unlock).!lock().Add06(prev,curr)
Add06(prev,curr) ≝ ?prev(get, set, lock, unlock).!unlock().Add08(curr,curr)
Add08(prev,curr) ≝ ?prev(get, set, lock, unlock).?get(next).Add04(prev,next)
Add10(prev,curr) ≝ τ.Add11(prev,curr)
                   + τ.Add17(prev)
Add11(prev,curr) ≝ ?curr(get, set, lock, unlock).!lock().Add12(prev,curr)
Add12(prev,curr) ≝ (ν temp, get, set, lock, unlock)(Add14(prev,curr,temp) | NodeUnlocked(temp, get, set, lock, unlock, curr))
Add14(prev,curr,temp) ≝ ?prev(get, set, lock, unlock).!set(temp).Add15(prev,curr)
Add15(prev,curr) ≝ ?curr(get, set, lock, unlock).!unlock().Add17(prev)
Add17(prev) ≝ ?prev(get, set, lock, unlock).!unlock().0
```

## Expressive Power

Since CCS can encode Minsky machine and the π-calculus is a superset of CCS, the π-calculus is also a Turing complete model.


## π-calculus as a graph

__Normal form.__
Without loss of generality, we assume a normal form for the definitions and process configurations.

Process definitions are sums of prefixes: `A(a) ≝ ∑_i π_i.(ν a_i) ∏_j A_{ij}(a_{ij})`.
Furthermore, we assume that there are no free names in the definitions `fn(∑_i π_i.(ν a_i) ∏_j A_{ij}(a_{ij})) = {a}`.

Configurations are the parallel composition of Process definitions: `(ν a₁ a₂ …) P₁(…) | P₂(…) | …`

To convert any term to that form, we can use congruence rules and, possibly, insert new definitions to abstract any prefix.
For instance, `?a.P()` is replaced by `Q(a)` and a new definitions `Q(a) ≝ ?a.P()` is added.

### Communication topology

We can represent a state of a system with as a bi-partite labeled graph where one kind nodes are process nodes and the other kind are for names.

__Communication graph.__
The graph is obtained as follow:
* Nodes (`V = V_n ⊎ V_p`):
  - For each name `a` in the top-level resolve, add a node in `V_n`.
  - For each `P_i(…)` in the list of processes, add a node with label `P_i` in `V_p`
* Edges: For each `P_i(a₁, a₂, …)`, add an edge with label `j` from the node in `V_p` corresponding to `P_i` and the node corresponding to `a_j` in `V_n`

__Example.__
The process `(ν a b c)(A(a,b) | A(a,c) | B(b,c))` corresponds to the graph:
```
        1
    A───────●       
  2 │       │ 1
    ●       A       
  1 │       │ 2
    B───────●
        2
```

__Dangling names.__
Because `a ∉ fn(P) ⇒ (νa)P ≡ P` any node in `V_n` that is not connected can be removed.


__Equivalence of configurations.__
We use the graph encoding to compare two processes using only the congruence rules that change the scope of restriction and substitute names.
In that setting, this equivalence check reduces to checking that the corresponding communication graphs are isomorphic.


### Covering Problem

In the graph view, we can express covering properties as finding a subgraph (error state) in another graph (global configuration).
This is a subgraph isomorphism check.

The _subgraph isomorphism problem_ for labelled graphs: Given `G = (V, E, L)` and `H = (V′,E′, L′)` find `G₀ = (V₀,E₀,L)` such that:
* `V₀ ⊆ V`
* `E₀ = E ∩ (V₀ × V₀)`
* there exists a bijection `f: V′ → V₀` (injective function from `V′` to `V`) such that
  - `(v₁,v₂) ∈ E′ ⇔ (f(v₁),f(v₂)) ∈ E₀`,
  - `∀ v₁ ∈ V′. L′(v₁) = L(f(v₁))`,
  - `∀ (v₁,v₂) ∈ E′. L′(v₁,v₂) = L(f(v₁),f(v₂))`.


__Example.__
Consider a variation of the client-server example where the server has a public address `s` and is connected to a private database.
If the database address leaks, this would be an error.

Assume the definitions are:
```
Server(s, db) ≝ …
Database(db) ≝ …
Client(s) ≝ …
…
```

We can express the error (to cover) in π-calculus as `(ν s db) Server(s, db) | Database(db) | Client(db)` and it corresponds to the graph:
```
           2       1
  Server───────●───────Client
  1 │        1 │
    ●       Database
```


### Transitions as graph rewrite rules

Transitions can also be expressed as graph rewrite rules.

#### Graph rewrite rules

__Definition.__
In this approach, a _rewrite rule_ is a triple `(L, R, m)` where
* `L` is a graph (left-hand-side, pattern),
* `R` is a graph (right-hand-side, what replace the pattern),
* `m` is a partial injective mapping from `L` to `R`.


__Notation.__
* For a graph `G`, we use `G.V`, `G.E`, and `G.L`, for the set of vertices, edges, and labelling function in `G`.
* For an injective (partial) function `m`, we write `m⁻¹` for its inverse: `m⁻¹(x) = y ⇔ m(y) = x`.
* Applying a function defined over elements to a set of elements simply apply the function to ever element in the set.

__Semantics.__
Let us apply `(L, R, m)` to a graph `G`.
Intuitively, we replace the part of `G` matching `L` by `R` while keeping the connections using `m`.

Let us assume that `G.V`, `L.V`, and `R.V` are disjoint sets.

1. Check that `L` is a subgraph of `G` (otherwise abort).
2. Let `f` the injective function from the vertices of `L` to `G` obtained by the subgraph isomorphism.
3. Construct a graph `H` with
  - `H.V = (G.V ∖ f(L.V)) ∪  R.V`
  - `H.E = { (v₁,v₂) ∈ H.V×H.V | (v₁,v₂) ∈ G.E  ∨  (v₁,v₂) ∈ R.E  ∨  (v₁,f(m⁻¹(v₂))) ∈ G.E  ∨  (f(m⁻¹(v₁)),v₂) ∈ G.E }`
  - vertex labels: `H.L(v) = l  ⇔  R.L(v) = l ∨ G.L(v) = l`
  - edges labels: `H.L((v₁,v₂)) = l  ⇔  R.L((v₁,v₂)) = l ∨ R.L((v₁,v₂)) = l ∨ G.L(v₁,f(m⁻¹(v₂))) = l ∨ G.L(f(m⁻¹(v₁)),v₂) = l`

The edges are the trickiest part.
The edges that are "completely" within `G.V ∖ f(L.V)` and `R.V` are kept.
Then we need to glue the boundaries of `G.V ∖ f(L.V)` and `R.V`.
`f(m⁻¹(v))` follows the mapping in reverse:
`m⁻¹(v)` (if defined) trace a node of `R.V` back to `L.V` and then `f` find the corresponding node in `G.V`.


__Example.__
Given
```
A(a, b) ≝ !a(b).0
B(a) ≝ ?a(c).C(a,c)
…
```
A transition from `A(a, b) | B(a)` to `C(b)` corresponds to the graph rewrite rule:
```
                 _________
                /         \
      1     1  /         1 \
    A────●────B    →   ●────C
  2 │     \___________/    2│    
    ●-----------------------●
```
as `(L,R,m)` with
* `L = ({w,x,y,z}, {(w,y),(w,z),(x,y)}, {w→A,x→B,y→●,z→●,(w,y)→1,(w,z)→2,(x,y)→1})`
* `R = ({m,n,o}, {(m,n),(m,o)}, {m→C,n→●,o→●,(m,n)→1,(m,o)→2})`
* `m = {x→m,y→n,z→o}`

Applying the rewrite rule to the graph `G`
```
        D
    1  1│  1  
   A────●────B
 2 │
   ●
```
with
- `G.V = {e,f,g,h,i}`
- `G.E = {(e,h), (e,i), (f,h), (g,h)}`
- `G.L = {e→A, f→B, g→D, h→●, i→●, (e,h)→1, (e,i)→2, (f,h)→1, (g,h)→1}`
and the subgraph isomorphism with the mapping `{w→e, x→f, y→h, z→i}`.

gives `H`
```
        D
       1│  1  
        ●────C
            2│
   ●─────────┘
```
with
- `H.V = {m,n,o,g}`
- `H.E = {(m,n),(m,o),(g,n)}`
- `H.L = {m→C, n→●, o→●, (m,n)→1, (m,o)→2, (g,n)→1}`


#### Graph rewrite rule: single pushout approach

__Remark.__
You don't need to understand this part for the exam.
I'm leaving it here as this is the kind of definitions you are likely to encounter in the literature.

__Definition.__
In this approach, a _rewrite rule_ is a triple `(L, R, m)` where
* `L` is a graph (left-hand-side, pattern),
* `R` is a graph (right-hand-side, what replace the pattern),
* `m` is a partial homomorphism from `L` to `R`.

Given two partial graph homomorphism `h: G₀ ⇀ G₁` and `g: G₀ ⇀ G₂`, the _pushout_ of `h` and `g` is:
* a graph `G₃`
* a partial graph homomorphism `g′: G₁ ⇀ G₃`
* a partial graph homomorphism `h′: G₂ ⇀ G₃`
such that:
* `g′∘h = h′∘g`
* for every pair of homomorphisms `g″: G₁ ⇀ G₃′` and `h″: G₂ ⇀ G₃′` there exists a unique homomorphism `f: G₃ ⇀ G₃′` with `f∘g′ = g″` and `f∘h′ = h″`

Intuitively, `G₃` is the minimal graph such that we get `G₃` when applying `h` and `g` to `G₀` and `h`,`g` "commute".

Visually it looks like:
```
       h
    G₀ → G₁
  g ↓    ↓ g′
    G₂ → G₃
       h′
```


__Semantics.__
Applying `(L, R, m)` to a graph `G`:
1. Compute a match `l` (total injective morphism) from `L` to `G` (`L` is a subgraph of `G`).
2. Replace the part `G` matching `L` by `R` while keeping the connections using `m`. More formally, the result is the pushout of `m` and `l`.
