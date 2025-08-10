---
tags:
  - concept
  - project/essay
  - release
  - adapter
date created: 2025-06-29
last updated: 2025-07-21
todo:
  - add examples/links to simple imp of RI
  - make Substack
  - group with other essays to post
  - add constitutional convention algorithm
  - add diagram of RI
status: stable
---
`ext` [[दृष्टि]]

# Muses
I recommend reading this essay while listening to one of the following pieces: 
```
- Palimpsest for String Septet, Simeon ten Holt 
- Harmony of the Spheres, Joep Franssens 
- Become Ocean, John Luther Adams 
- Sleep, Max Richter
```

# Definition 
Governance is the practice of maintaining the well-functioning of a human organization. It functions as a set of processes for organizational homeostasis.

`dual` Governance is distinguished from regulation. Regulation establishes rules from outside an organization to maintain its function. Regulation comes from without; governance from within.

**Internal Governance** is the practice of maintaining the well-functioning of oneself and one’s mind in a structured, rule-governed manner.

In this essay, I define what I think are the basic requirements of an effective internal government, how to instantiate and run an internal government, and provide a basic model of my own democratic implementation called `Republic Internum` or `RI`.

As a disclaimer: the implementation of `RI` I provide is an early version of what I use today and is shown only for expositive purposes. As I put a lot of private information into `RI`, I’m not sharing the full version.

# Instantiation 
The first step of governance is to establish a government. 

## In the external setting, 
governance is established either by coercion, consent, or assent. Corresponding tools include lethal/military force, assembly and dialogue, or cultural pressure and reliance on social norms.

## In the internal setting, 
governance is established analogously, but the tools are more restrictive. You can’t use lethal force on yourself to establish governance. You can figuratively dialogue with yourself, but there is only one person in the usual sense to talk to. And you may leverage social norms or cultural pressures, but this establishes regulations, not governance.

The key here is in exploiting the second piece: you can *figuratively* dialogue with yourself. One can often identify competing values and interests within themselves that require extended dialogue to draw out and clarify. We might say that such an extended dialogue comprises an internal constitutional convention, whereby each distinct competing value system meets to debate which foundational values to adopt as part of the internal constitution. The end-deliverable is your constitution. 

I strongly advise simulating such a discussion within yourself, writing down its events, and summarizing them into meeting notes. To simulate the requisite reflection to process these notes into a constitution, *take a break* from these deliberations to clear them from your working memory. Then return to them and compile them into a constitution.

Here’s an algorithm that summarizes the above: 
\<algo: constitutional convention\>

# Continuation 
The next step is to develop and implement processes for running the government. 

# Appendix: Artificial Intelligence and Simulation 
You can definitely take advantage of AI for these purposes. In my experience, I’ve found AI to be very useful for automating certain aspects of internal governance and much less useful in other aspects.

## AI can automate shallow governance tasks 
As in the real world, much of governance involves shunting information from one location to another. Secretaries, paperwork, and bureaucracy pervade. This may be barely tolerable in the human world, but it cannot be tolerated in the internal setting. 

You’re designing a *government* for *you*. If it doesn’t work efficiently, you’re going to be wasting a lot of time. Because of this, I am explicitly in favor of implementing any processes for automating anything that would otherwise be a massive waste of time for you to do manually. This is also where some general programming knowledge comes in quite handy, as you can write batch scripts for groups of files to rapidly file them into different locations, then give an AI control over the script and automatically sort new documents into their appropriate location. 

>Aside: In a sense, governance in the external setting is itself automation of a kind. The whole point of setting up a government is that it will work on its own after the generation that has set it up leaves. In the internal setting, this mimics you summoning a set of “founders”, them setting up a government, then getting out of dodge and letting the government work by itself. To the extent that you can automate shallow parts of this process, you should do so.

## AI should not be a voting member 
While AI systems display profound advantages in their ability to enable and rapidly generalize patterns of human expression, they are not themselves patterns of human expression. This is important as you consider who you will allow or design to be a voting member of your government. 

## Why not simulate it all?
It’s entirely feasible to fully simulate a government for yourself using a program and a few AIs after providing that program a set of foundational values that you write and a few instructions for running it. 

Then you don’t have to do any more work - you can just listen to what the simulated government tells you. 

I’m not inherently opposed to this method, but in practice I would advise against it, and I try to avoid exogenously simulating my councilors when possible. The reason this is ill-advised is due to a general limitation in probabilistic reasoning, which all AI systems rely on: 

>To generate outputs that are internally coherent and consistent with a general observed pattern, an AI requires a **representative** sample of data that captures the overall distribution’s pattern.

There are many examples of this criterion in medicine: consider any clinical trial. Clinical trials enroll participants according to extremely rigorous criteria (which include randomization, sufficient sample sizes, appropriate study design and controls) precisely because their results are intended to generalize to a much larger segment of the population. 

The same criterion applies to any commercially available AI, as these systems rely on probabilistic models of inference that expose patterns in large datasets. But if there’s no large dataset, the model puts out junk. 

As such, I’d recommend people only consider complete automation if they think their government meets an appropriate size cutoff, either in the number of actions/tasks/notes (or any other standardizable data points) generated or size of the governance model itself (i.e. number of representatives, length or complexity of values/constitution).

# Fidelity 
Up until this point, we’ve assumed that running a government within oneself is a straightforward, feasible task. 

In my experience, it’s mindshatteringly hard. Files and documents get lost. I forget stuff. I have an idea that I thought I wrote down and discover I accidentally deleted it. And of course, ideas can get scattered over different places. All of these are threats to **fidelity** or **integrity**, which is the essential character of an entity. 

I take three approaches to try to preserve fidelity in my models. First, any and all substantial modifications or events in internal governance models are written down or drawn out in a notebook or set of notebooks. Second, I aggressively version and back up every major form of governance I’ve adopted on a regular basis to track development over time. Third, I try very hard to map out generalizable values and principles on which the governance is originally based - this comes down to a high quality of your internal constitution. 

# Implementation: Republic Internum
I implement internal governance as Republic Internum or `RI`. At its most basic level, `RI` consists of the elements shown below.

\<Insert diagram of `RI` components\>

My implementation of `RI` consists of:
- A *Council* of multiple inner voices, each signifying a distinct set of values that may conflict with those of other councilors. I implement this in [[The Council]].
- A *Senate* or conceptual space equipped with formal rules for deliberation among the Council about important decisions. I implement this as the [[Permanon]].
- A set of *formal rules* for deliberation and manipulation of resources (in my case, my resources are ideas). I specify these rules in [[The Kernel]], which mostly uses [[Ops]] to interact with and manipulate `RI`.
- A location for the resource to be governed. I implement this in two ways: Obsidian, which has separate folders for my daily journal, projects, captures, and [ideas](Eidesis) that I intend to cultivate over time, and [[The Reflectorate]], my network of notebooks that I use for coordinating my self study of different disciplines I’m interested in. 

`RI` is implemented as a federal republic of inquiries. *Federal* implies that it is a higher-order governing structure of multiple autonomous “states” which govern themselves to an extent. *Republic* implies that there’s some level of democracy involved: no one inquiry should necessarily dominate all of the others for too long. The states being governed in `RI` are inquiries - projects, metaprojects, essays, and chains of ideas. The citizens of `RI` are the ideas themselves. 

# Appendix: Internal International Relations 
All of us have internal governments; not all of us are conscious of them. 

This realization is central to considerations regarding with governments you want to deal with aside from yours. 

Many individuals operate on internal governments under occupation by other internal or external forces. Many others operate on fascist/authoritarian governments, which correspond to the characteristically harsh or abusive self-image we hear from people who struggle with self esteem. No matter what government you’re dealing with in an interaction, you’ll want to establish some general principles for establishing (or not establishing) relations with them. 

The following section will proceed under the extended analogy of the self as president of your internal government, and other individuals as leaders of their own internal nations. We will say

It’s good to be wary here about what nations you choose to ally with. Not all relationships are beneficial, and some of them demonstrably weaken your ability to govern yourself. 

# Appendix: Symmetry Breaks 
Politics and relations in the internal setting and external setting are analogous, and the analogy extends quite nicely into economics and international relations. But they are not completely isomorphic. 

Citizens within a democratic government in the external setting have moral rights. I do not think there is a good isomorphic argument for the rights of citizens in an internal government. 

# Appendix: Not everyone needs an internal government
*There must be a good reason* to establish internal governance. I try to outline some soft prerequisites below.

## Stakes
Governments are typically set up when there’s something *large* at stake that affects a large number of people, be it a decision how to allocate a large amount of money or resources, a common set of values or way of living, or a recurring set of important deals with another such group. 

That requires there to be large stakes - something extremely valuable to be divyed up. There’s no point in establishing an internal government if you have nothing (or nothing sufficient) to govern. 

In my case, I know that my *time, attention, and ideas* are extremely valuable. I’m in my third year of medical school. I know how to program fairly decently and reason about large datasets and systems. I’m involved in several projects in public health. So there’s definitely something high-stakes to govern: my ideas, time, and energy. If this wasn’t the case, I wouldn’t bother coming up with a structure anywhere near as elaborate as this for my decisions. 

## Competing Interests
Governance also requires there to be plausible and persistent competing interests for high-stakes decisions. In the external setting, this manifests as liberals and conservatives (in the European setting, radicals, reactionaries, and reformers). 

If you tend to make decisions without experiencing any substantial tensions between your values, you probably don’t need a government to help you adjudicate between your values. 

In my case, I’ve realized that I have a fairly complex set of values that routinely compete with one another, and I haven’t been able to find a closed form or firm hierarchy for these values. 

# Summary
Corporations are not people. Buts so long as we continue to operate our society as if they are, it may be advantageous to consider the human person as a corporation. 

# Bibliography 
## Cybernetics & systems
Internal governance as implemented above is inspired by several emerging (and some older) paradigms in psychology, cybernetics, and complex systems thinking.
1. Internal Family Systems 
2. Psychocybernetics 
3. Cybernetics, Norbert Weiner 

## Commonplace Books 
There are also several people who talk about how to construct the equivalent of an internal governance model in analog/notebook systems.
1. [Oddly Specific Crystal on Commonplace Books](https://youtu.be/8kTZQ9N8iv8?si=Hunq8DwzDZjlA3ZA)
	* This video introduced me to the concept of Commonplace Books. Amazing introduction and highlights that commonplace books can include things other than reflective notes - drawings, lists, keywords, ideas, etc!
2. [ParkNotes on Commonplace Books](https://youtu.be/ex8ZRIGdHEQ?si=bFwo3T5cjLvo0LPo)
3. [Jared Henderson on Commonplace Books](https://youtu.be/CcBy_b_43c0?si=-rWFne51dmwRThMM)

