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

## Shrinking delta dictionaries
As described above, we use a delta compression strategy, where a range
of data layers can be turned into one single data layer for more
efficient querying. This compression is transparent. we ensure that
the resulting compressed layer behaves the same way as the original
range. In particular, we ensure that any value that previously existed
(even if it has since been deleted) will be assigned the same id when
it appears in a triple, regardless of whether you use the original
layer stack or the delta layer as the basis of your operation. This
ensures we can do delta compression at the same time as operations on
the graph are occuring, as we can be sure that newly built layers will
still work with our delta layer.

One disadvantage of this approach is that we never throw away a
value. If we did throw away unused values, then when a new layer tries
to reuse these values, it'd allocate a new ID if built on top of a
delta-compressed layer, but reuse the existing ID if built on top of
the original layer stack. Therefore, because we wish to retain
equivalent behavior, we need to keep around old, unused values. This
means that if a particular database has a lot of changes in data
(rather than a lot of changes in links), over time, this database will
accrue a lot of unused dictionary entries.

Contrast this with a squash operation, where we build a completely new
layer without taking care to be compatible. In this case, we can
actually throw away old values, and build new dictionaries where these
values simply do not occur, resulting in smaller dictionaries. But
squashes cannot be done concurrently with graph modifications. They
have to stop the world while they're built, preventing new layers from
being constructed on top of the old data until that operation is done.

It may be worthwhile to see if there's some way to create a delta
layer which does omit unused dictionary values, but which can still be
built concurrently. Possible solutions:
- have special layers where particular ids are permanently released
- Use squash and rebuild any new concurrently built layers afterwards

A related problem is that over time IDs get larger, requiring more
storage to hold them. It would be good if it was possible to reorder
IDs such that the most used IDs are small.
