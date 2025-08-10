---
tags:
  - kernel/process
  - release
date created: 2024-11-05
last updated: 2025-08-01
description: Description of RI's kernel operators.
aliases:
  - Kernel Standard Library
  - lib:ker
  - Command Reference
status: stable
---
>A large volume of thought must be systematically ordered by appropriate rules and regulations.

# Definition 
An **operator** or **op** is an algorithm executed by the [kernel](<The Kernel>) over some subset of `RI`.

Details of the operations are determined by the context of the domain they operate over. In other words, be flexible in applying the definitions to different areas of `RI`.

# Most Frequently Used 
## Initializers
[`divinatio`](<Op divinatio.md>)
[`admonitio`](<Op admonitio.md>)
[`memoratio`](<Op memoratio.md>)
[`praemissio`](<Op praemissio>)
[`spawn`](<Op spawn.md>)
[`new`](<Op new.md>)

## Transitions
[`invoke`](<Op invoke.md>) (`echo`)
[`interject`](<Op interject.md>) (`inter`)
[`extend`](<Op extend.md>) (`ext`)
[`analog`](<Op analog.md>)

## Idea Transfer
[`graft`](<Op graft>)
[`portal`](<Op portal.md>)
[`restore`](<Op restore>)
[`extend`](<Op extend>)

## Conceptual Transformers 
[`pivot`](<Op pivot.md>)
[`audit`](<Op audit.md>)
[`invert`](<Op invert.md>)
[`emulate`](<Op emulate.md>)
[`dualize`](<Op dualize.md>)
[`divorce`](<Op divorce.md>)
[`mult`](<Op mult.md>)
[`splay`](<Op splay.md>)
[`sample`](<Op sample.md>)
[`as if`](<Op as if.md>)
[`resolve`](<Op resolve.md>)
[`complete`](<Op complete.md>)
[`otherize`](<Op otherize.md>)
[`dislodge`](<Op dislodge.md>)
[`dispel`](<Op dispel.md>)
[`abs`](<Op absolution>)
[`gen`](<Op generalize>)

## State shifts
[`reset`](<Op reset>)
[`suspend`](<Op suspend>)
[`decisis`](<Op decisis>)

# Index
```dataviewjs
dv.table(
  ["File", "Last Updated"],
  dv.pages()
    .where(p => p["last updated"] && p["type"] === "operator")
    .sort(p => p["last updated"], 'desc')
    .map(p => [
      p.file.link,
      p["last updated"].toFormat("MMMM dd, yyyy") // e.g., July 18, 2025
    ])
);
```

