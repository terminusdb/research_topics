# Research Topics

Running and updated list of research topics regarding efficient
storage and search of versioned graphs.

# Data Structures

Which data-structures support the following operations on versioned
graphs with large numbers of commits efficiently.

We view an edge as a tuple (s,p,o) for s∈S p∈P o∈O for countable
alphabets S (node), P (edge) , V (value) where O := S⊕V a disjoint
union between values and nodes.

1. Search by (s,?,?), (s,p,?), (?,p,o) and (?,?,o)
2. Check cardinality of (s,p,?), (s,?,?), (?,p,o), (?,?,o)
3. search additions, deletions at a commit as with 1
4. add / delete edges

## Some potential approaches

### Delta compression

Currently in TerminusDB we use an
[HDT](https://en.wikipedia.org/wiki/HDT_(data_format)) style format
for the initial commit, and pairs of HDT style formats for each delta,
storing additions and deletions.

Large numbers of small writes will have different behaviours than
small numbers of large writes. When do we find it is cheaper to
compress delta-stacks. If you can do "short-cut" queries over a delta
compression you get much better behaviour.

```
                deltas      head
initial       ↓   ↓   ↓    ↙
commit →  ◯ - ◯ - ◯ - ◯ - ◯
              |       |
              \―――――― ⬤
                      ↑
                 delta compression

```

Where should these delta compressions be introduced to maximise
performance with different usage profiles? How should we characterise
these profiles?

### Persistent Radix Tree

[Persistent Radix Trees](https://ankurdave.com/dl/part-tr.pdf) give a
method which could work for a commit based data structure. The
technique of extending radix trees to
[k-dimensions](https://www.cs.umd.edu/~hjs/mkbook/chapter1.pdf) and
equiping them with range queries may provide a faster and more compact
representation.

However this approach will probably also benefit from delta-compressions.

### Persistent k2-tree

[k2 trees](https://arxiv.org/abs/2002.11622) have been shown to
represent large networks very efficiently and support the types of
queries required. However they have not been described with a
persistent extensions. Other persistent succinct data structures have
been described and it may be possible to adapt these approaches to k-2
trees.

Alternatively the chained approach used with HDT above can also be
used with k2 trees.

## Efficient subgraph search

While networks are extremely useful ways of representing data, it is
often the case that we want to have sub-graphs of the data treated as
a coherent bundle. In TerminusDB we represent sub-graphs as
interconvertable to JSON documents. Essentially we require the
sub-graph to be a DAG and the full network can only exist between
these DAG sub-graphs.

It is convenient to treat these sub-graphs as having links in, and
links out, such that the entire document can be thought of as a
pseudo-node, somewhat like the way we treat hypertext documents on the
web (where outgoing links are actually embedded in a structured
document).

Efficient techniques of treating the subgraphs as psuedo-nodes for
network analysis have not yet been developed and could be an
interesting problem.

## Patch manipulation

Patches are important for understanding what updates are taking place,
especially in the presence of concurrency. When patches happen at
different repositories with different cadences, sometimes we must
calculate whether a merge is possible without conflict by checking the
commutivity of patches.

- What algorithm should be used to check whether an insert / delete
  patch over a subgraph commutes with another subgraph patch?
- What types of patches will improve commutivity for tree+list (JSON)
  structures.
- What sub-space of JSON style can be equiped with CRDT patch logic to
  speed up checks of commutivity?
