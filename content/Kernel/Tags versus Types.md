---
tags:
  - kernel/rim/metadata
  - project/essay
  - adapter
  - release
date created: 2025-07-17
last updated: 2025-07-22
status: stable
type: adapter
---
In theory, types (and any other YAML frontmatter property) could just be tags. The reasons I denote a type as a unique property (as opposed to another kind of tag) are as follows:

1. Tags can be flexible, redundant, and orthogonal. Types must be hierarchical. 

As my tag system was naturally evolving into a pretty well-formed (and sometimes rigid) hierarchy (an essay became a subtag under project, as did annotation, speech, etc; I came up with informal rules for putting one tag as a child of another versus having it be unrelated, and so on), I noticed a conflict between wanting to assign tag/topic labels freely to notes without caring for conformity to a broader system versus wanting to maintain overall integrity and consistency of the essential “types” that the tags pointed to. Spontaneity and structure. 

If tags are meant to be flexible, then I want another system that defines the essential character of a note for purposes of serializing it as a data point in `RI`. This allows me to free up tags to be what they are, which are user-defined keys, and implement some stricter guidelines for types. 