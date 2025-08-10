---
tags:
  - project/sys
  - external/med
  - dashboard/sector
  - adapter
  - release
date created: 2024-10-17
last updated: 2025-08-03
description: My personal medical knowledgebase.
type: sector
status: stable
---
# Definition
The **Medical Sector** is concerned with the encoding, maintenance, application, and refinement of medical knowledge. 

The Medical Sector is my implementation of a medical knowledge base. 

# Concepts
>Current Focus: OBGYN
```dataviewjs
dv.table(
  ["File", "UTD"],
  dv.pages()
    .where(p => p["last updated"] && 
		Array.isArray(p.tags) &&
		p.type == "representation" &&
		p.tags.some(tag => typeof tag === "string" && 
			tag.startsWith("external/med/obgyn"))
	)
    .sort(p => p["last updated"], 'desc')
    .map(p => [
      p.file.link,
      p["last updated"].toFormat("MMMM dd, yyyy") // e.g., July 18, 2025
    ])
);
```

## Templates and Guides
[[Scripts]]
[[Core Clerkship Strategy]]

## Career Exploration
[Psychiatry](<Psychiatry Career Exploration Dashboard.md>)
[[Notes for M3 Career Meetings]]

### CAP
[[CAP Midterm Evaluation]]
[[CAP Final Reflection]]

### CSL
[[CSL Reflection Drafter]]
[[CSL Hours]]

## Step 1
[[Step 1 Subsystem]]
[[Step 1 Study Generators]]

## AMA-MMS
[[AMA-MMS Dashboard]]

## Historical
[[M27 Artifacts]]

## Tools
[[Medical Information Tools]]
