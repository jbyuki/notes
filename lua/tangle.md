The tangling algorithm
----------------------

These are some ideas on the data structure to store
and process untangled code, to tangle it, more specifically
incrementally tangle it (similar to incremental compilation).

Let's take an example
```
@hello=
@sec1
aaaaaaaa
@sec1+=
bbbbbbbbb
@sec1+=
ccccccccc
```

The code is composed of a root section `hello` which contains
a reference to sec1 followed by a line "aaaaaaaa". The sec1
section contains "bbbbbbbbb" followed by "ccccccccc". 

This structure ressemble one of a DAG (directed acyclic graph).
This is good to keep in mind but this is probably more anecdotic
than useful.

```
   ┌───┐       │ ┌────────────┐
   │MAP│       │ │LINKED LISTS│
   └───┘       │ └────────────┘
 KEY     VALUE │
               │
"hello" [HEAD]───► hello 
               │     ▼
               │   @sec1
               │     ▼ 
               │   aaaaaaaaa
               │
"sec1"  [HEAD]───► sec1 ──────► sec1
               │     ▼           ▼ 
               │   bbbbbbbbb    ccccccccc
```

This is the data structure used in most of my plugins such as
[ntangle.nvim](www.github.com/jbyuki/ntangle.nvim). It is composed
of a main map called sectionlist where the key is the section name
and value is the head of a linkedlist of every section definition
and under each section definition is another linked list or array
with the lines which can either be text or references to sections.

The idea is to augment this data structure so that it also plays
nicely with incremental tangling.

It has to support:

1) Fast insertion/deletion of a line specified by a line number
   in the untangled code. Fast is O(log n).

2) Fast lookup of the line number given a line where the line
   number is in the tangled code.

My initial idea is to use a B-tree or B+ tree for 1) and augment
the data structure with a LOC number of each map entry for 2),
then given a line walk backward until the root section to find
the absolute line number for a line. Each section part in the
linked list could also hold its LOC for a faster lookup.
