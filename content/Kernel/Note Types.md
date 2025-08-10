---
tags:
  - release
  - concept/primitive
  - kernel/rim
date created: 2025-07-17
last updated: 2025-08-03
aliases:
  - ntype
status: stable
type: concept
---
# Definition 
A **note** is the basic unit of information in any Obsidian vault and hence in `RI`. 

Notes are statically typed: key aspects of their structure, are determined *a priori* according to the type assigned by the user. 

# Note Types 
## Concept
>A [concept note](<Concepts>) contains an original idea. 

Concepts may be *primitive*, *derived*, or *polymorphic/higher-order*.

A *primitive* concept is self-contained and considered a fundamental building block for other concepts.

A *derived* concept is built from primitive concepts. 

A *polymorphic/higher-order* concept is a fuzzier, broad pattern defined by the user in a manner that does not fully reduce to a combination of derived concepts.

## Register 
>A register note holds a small set of data for short-term memory and rapid execution. Its contents can be modified freely (except for its properties).

This is implemented as [[Queue]].

## Project 
>A project note is either a project itself or a relevant note for a project. 

Projects are further subtyped into [[Project Types]].

## Metaproject
>A metaproject note is a project about projects. 

A metaproject typically addresses some fundamental aspect of how I incept, design, execute, or archive projects, or something even more fundamental about my psychology and approach to thinking and acting. 

## Eidesis
>An eidesis note contains dreams and visions. 

Eidesis notes are unstructured “fuzzy” ideas, dreams, visions, or other experimental modes of thought. They may be tagged by additional descriptors, e.g. #dream, #vision, that describes the eidetic type. 

## Style
>A style note describes a style or informal pattern of writing and behavior. 

## Dashboard/Sector
>A dashboard/sector note is an index of notes relevant to the topic or header of the dashboard. 

Dashboard/sector notes serve as “Maps of Content” (MOCs), which are birds-eye-biew navigational aids for a vault. 

### Dashboard Hierarchy
- A *dashboard* is the lowest level of a map-of-content.
- A *district* is a higher-order map-of-content than a dashboard. It groups dashboards with shared topics together.
- A *sector* is the highest level of organization. It groups topics by their similarity in organization and interaction frequency. 

## Summa
>A summa note is a summative or foundational note that informs the content of all other notes in `RI`.

# Rules for types 
- Notes can only have one type (with appropriately defined subtypes); 
- Types must be organized under a tree. The leaves of the tree are primitive types. The internal nodes are derived types; 
- The type structure must be as minimal as possible while describing the whole expressive range of notes within `RI`. 

`meta` It may be advantageous for us to implement rules for type inference in `RI`. We can do that later.

# Expansion Paradigms
Type systems can get convoluted quickly, and it may be difficult to understand the intuition behind why a system chose certain types over other types. I list the paradigms I use to inform expansions of this type system below.
- *Practical Foundations for Programming Languages* by Robert Harper is an introduction to the formal analysis and design of programming languages with an emphasis on type theory. 