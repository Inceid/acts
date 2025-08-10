---
title: Atlas
tags:
  - atlas
  - release
date created: 2024-10-06
last updated: 2025-08-05
description: "Informally: A map of or hubworld for `RI`. Formally: An array of pointers to frequently accessed pages and dashboards within `RI`, augmented with a table of systems outside of `RI` that supplement its functioning and expansion."
type: sector
status: stable
---
>The larger a civilization grows, the more imperative it is to establish a hub, a town square, a sanctum where all ideas and parties can safely, openly, and frequently commune. This will be the entry point to any other ideas.

# Definition
The **Atlas** is a map, hubworld or meta-index of `RI`, describing its overall organization and allowing convenient access to any area of interest.

I recommend new readers start [here](<What is Republic Internum>) or [here](<Who am I>).

# महारत्नान
>Great achievements. Projects and creative/interpretive works that I love showing off.

- [[Syllabus Vitae 02.27.2025.pdf]]
- [clox](<https://github.com/inceid/clox>)
- [[Annotation of Crafting Interpreters]]
- [[Annotation of The Sources of Normativity]]
- [[C Axler Summary Sheets.pdf]]

# Active Sites
>An **active site** is a set of (non-project) notes undergoing active modification.

```dataviewjs
dv.table(
  ["File", "Last Updated"],
  dv.pages()
    .where(p => p["last updated"] && p["status"] === "active"
								  && p["type"] != "project")
    .sort(p => p["last updated"], 'desc')
    .map(p => [
      p.file.link,
      p["last updated"].toFormat("MMMM dd, yyyy") // e.g., July 18, 2025
    ])
);
```

# Active Projects
>A **project** is a persistent activity with an external or world-facing output.
```dataviewjs
dv.table(
  ["Proj", "Status", "Last Updated"],
  dv.pages()
    .where(p => p["last updated"] && p["type"] === "project" 
								  && p["status"] === "active")
    .sort(p => p["last updated"], 'desc')
	.sort(p => p["projstatus"], 'desc')
    .map(p => [
      p.file.link,
	  p["projstatus"],
      p["last updated"].toFormat("MMMM dd, yyyy") // e.g., July 18, 2025
    ])
);
```

# Sectors 
>A **sector** is a persistent cluster of conceptually related notes linked together with a dashboard or commonplace note. It is denoted by a note with type `sector`.
```dataview
table file.name as Sector 
where (contains(file.tags, "sector") or (type = "sector"))
```

# Summa 
>A **Summa** note is a constitutional or foundational document informing the development, structure, and content focus of `RI`. 

| Note                           | Description                                                                                                               | Function in `RI`                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [[The Kernel]]                 | Describes the governance structure and mechanisms of control/modification of `RI`.                                        | Controller of `RI`.                                                    |
| [[Constitution]]               | Constitution of `RI`.                                                                                                     | Informs implementation of the Kernel.                                  |
| [[दृष्टि\|Values Declaration]] | Declaration of the foundational values of my internal government.                                                         | Informs the Constitution.                                              |
| [[metapcms]]                   | Metadegree curriculum and deliverable set for Philosophy, Computer Science, Medicine, and Systems Thinking.               | Informs planned [self-directed inquiries](<Self-Directed Inquiry.md>). |
| [[Kernel/अन्वेषण\|Twelve]]     | Twelve Favorite Problems identifying the phenomena I find most interesting to study and reflect about on a regular basis. | Informs spontaneous self-directed inquiries.                           |
| [[Suraj's Career Masterplan]]  | Career Masterplan developed during my High-Impact Medicine Career Fellowship taken in Fall 2023. Awaiting an update.      | Informs career planning.                                               |


# Implementation Guides
## Plans | Structures
- [[Ideation and Execution]]
- [[From Idea to Implementation]]
- [[From Consumption to Production]]
- [[Project Design Theory|proj]]

## Tools | Implementers
- [[Project Types]]
- [[Project Status]]
- [[Queue]]

# [[Externs]]
>Externs are platforms outside `RI` that interact with it.

# System Description
> Description of all systems in `RI`’s device ecosystem.

Major apps and devices that interact with `RI` are described in [[Designing a Device Suprastructure|Platform-Device Suprastructure]].

Specific versions of devices I use are described in [[Analog Systems]] and [[Digital Systems]].

# [[General Glossary]]
>The general glossary defines special terms used in `RI` and its ecosystem. 
