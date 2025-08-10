---
tags:
  - project/annotation
  - dashboard/sector
  - release
aliases:
  - anno
  - Research Sector
last updated: 2025-07-25
type: sector
status: stable
---
# Definitions
Readings and annotations are either **structured** or **spontaneous**. 

A **structured annotation** is planned ahead of time, usually for a long work.

It tends to take a long time (at least a few weeks), and hence requires a longer and deeper set of notes for (1) outlining/capturing insights from the reading, (2) distilling the reading into summative structures/insights, and (3) incorporating that knowledge into future perennials and projects.

A spontaneous annotation, on the other hand, usually happens on the fly when I'm listening to a video or podcast and realize I need to take some notes on it, or better yet when I'm at a lecture or event and want to record a few insights from it.

# Structured Annotations
```dataview
table file.name as Annotation 
where contains(file.tags,"project/annotation") and contains(file.tags, "dashboard")
sort file.cday
```

# Additional Data

| Book/Work                                                                           | Start                  | Complete |
| ----------------------------------------------------------------------------------- | ---------------------- | -------- |
| [[Annotation of Godel, Escher, Bach]]<br>                                                         |                        |          |
| [[Annotation of Practical Foundations for Programming Languages]]<br>                             | 09/12 <br>(`RI` 11/27) |          |
| [[The Practice of Moral Judgment]]<br>                                              |                        |          |
| [[Annotation of Building a Second Brain]] <br>                                      | 10/24                  | 11/09/24 |
| [[Annotation of Crafting Interpreters]]<br>                                         | 10/26                  |          |
| [Creative Visualization](<Annotation of Creative Visualization.md>)                 | 11/27                  |          |
| [How to Take Smart Notes](<Annotation of How to Take Smart Notes.md>)               | 12/17                  | 12/29    |
| [Kant’s Transcendental Idealism](<Annotation of Kant's Transcendental Idealism.md>) |                        |          |
| [The Sources of Normativity](<Annotation of the Sources of Normativity>)            |                        |          |

# Spontaneous Annotations
```dataview
table file.name as Annotation
where contains(file.tags, "project/annotation/spontaneous")
```