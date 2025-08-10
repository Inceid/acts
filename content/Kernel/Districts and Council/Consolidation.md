---
tags:
  - strategy
  - transcription
  - release
date created: 2024-10-15
date transcribed: 2025-07-17
source: Notion

status: stable
---
`dual` [[Decentralization]]

# Definition 
**Consolidation** is the act of combining similar ideas under a single idea. 

In order for an action to consolidate a set of concepts, the combining act must either preserve or reduce the amount of information required to express the set of ideas.

More precisely, suppose we are given a set of concepts $c_1, ...,c_n$ over a concept space $\mathbb{C}$, each  requiring  $I(c_i)$ information to express it. As concepts exist to be applied, we say $A(c_i)$ is the applicability of a concept.

Intuitively, we would like consolidation to reduce the amount of unique information that we have to keep track of in our minds while preserving the functional utility of the concepts being consolidated.

Let $\mathcal{C}: \mathbb{C} \mapsto \mathbb{C}$ be a function that maps concepts to their consolidated version. Then $\mathcal{C}$ is a consolidator iff:
1. $\sum\limits_{i=1}^{n}I(c_i) \geq \sum\limits_{i=1}^{n}I(\mathcal{C}(c_i))$ : the total information is maintained or reduced by consolidation.

2. $\sum\limits_{i=1}^{n}A(c_i) \leq \sum\limits_{i=1}^{n}A(\mathcal{C}(c_i))$ Consolidation preserves or improves usefulness.

  **Example 1:** Consolidating concepts from Pharmacology.
- Medical pharmacology, especially when considered among the other stuff you have to learn, is incredibly hard to put into your brain. However, there’s a very neat structure (made explicit by Paul Abourjally during his lecture) that you can sort the information into for purposes of exams, if not for life itself: category, mechanism of action, indication, side effects/contraindications. These four categories represent the consolidator for the relevant pieces of information from the lecture.
    - This implies that the way to master pharmacology is to embed, apply, and modify this consolidator as you go through medical school and medicine.

# Motivation 
If you find yourself a person who continually emits extremely inspiring ideas, and happen to find spontaneous connections amongst those ideas, the best thing that you can do is to collect them into a single platform. As expression platforms themselves are designed to dissipate and scatter over time (given the increasing pace of technological innovation and production), exerting some extra effort to systematically collect and unify the information across those platforms 

# Implementation: Consolidation into `RI`
1. For each platform you use that has a set of scattered objects, spawn a single “hub” object and link or paste all of them into the hub. 
2. Link the hub into `RI` in [[Externs]].
3. Consider transferring the hub’s data into `RI` and deleting it. 

