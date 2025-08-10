---
tags:
  - kernel/council
  - release
date created: 2025-06-21
last updated: 2025-07-06
aliases:
  - perm

status: stable
---
>All empires fall. Only philosophy and its governance ever remains. 

# Definition 
The **Permanon** is the Senate of the council. It is the council’s default formal deliberative space.

The purpose of a session of the Permanon is to clarify one's various competing values in a deliberation, problem, or decision. 

# Notation 
To summon a permanon session, call

```
Permanon <members>
{
  RESOLVED: <topic>
  <body of discussion>
}
OUTCOME: <outcome>
(decisis <decision>)
```
where 
- `<members>` specifies the parties attending the `Permanon` session, 
- `<topic>` is the topic under discussion, 
- `<body of discussion>` is the content of the session, logged roughly contiguously in real time with the discussion. 
- `<outcome>` is the summary and overall outcome of deliberation. 
- `decisis` `<decision` is an optional command at the end of a permanon session that is a directive or decision to do something significant. 

# Rules of Deliberation
Permanon deliberations occur roughly by Robert’s Rules of Order and parliamentary procedure. 

