---
tags:
  - kernel/rim
  - external/med
  - strategy
  - release
date created: 2025-06-25
last updated: 2025-08-04
aliases:
  - medbases
  - knowledge bases
  - building a medbase
status: stable
---
>There’s a method to the madness - and a madness to the method. 

# Preamble 
There is a very specific set of things every physician should know before entering clinical practice. As medical education continues to standardize, it pays to develop one’s own process for building a standard library for that medical information *which is likely to be implemented regularly in practice*. 

Not every physician needs to do this, but I do. I don’t respond well to guidelines, but I respond very well to endogenous, [personalized representations](<Vernacular Craft>) of those guidelines. 

Of course there are already all kinds of knowledge bases out there that have much more time, energy, and raw funding behind them than anything I will come up with myself. And I don’t fucking care. I didn’t get into this field because I’m obsessed with medical knowledge or because I like memorizing facts from databases. I got into this field to learn how to use my analytical skills to take effective care of people, and to integrate them with my emotional processing skills to improve and harmonize both. That goes to say, this is a personal journey. That requires that I create personal ways of representing and retaining any and all knowledge that I come across. 

As such, I understand that everything that I’m doing to learn medicine *is not* for the sake of passing a test. It’s for the sake of embedding a knowledge base in my mind that I can refer to for the rest of my personal and professional life, whether or not I engage in full-time clinical practice for my whole life. 

The other reason for taking a radically [re-creative](<Re-creation>) approach to building a medical knowledge base is that in the end, in your medical practice, whatever aspect of the knowledgebase you personalize is what you will retain and invoke. You know what you are. You invoke spells that you have designed or recast. And all we have are our schemas, our spells. 

Rather, all we have are our practices. And the practice of building and extending a knowledgebase is one of the best methods to continuously reinforce that knowledge. 

So I’m going to build my own version of an interactive medical library. My own UWorld / AMBOSS / GPT / MedCalc / DynaMed / UpToDate. The result of this attempt is the [[Medical Sector]].

# Level 1: Scan 
Medical school exams, Step 1, shelf exams, and Step 2 prioritize **breadth over depth**. It’s better to know a few facts about lots of different conditions than a lot of facts about a few conditions. 

Additionally, each exam prioritizes a fairly limited set of question types for each topic covered. You’re going to be asked to describe the epidemiology, pathophysiology, risk factors, diagnosis, treatment, or complications of a disease or finding, and generally not much else. (Starting clerkship year, you’re also asked about the management of conditions: what’s the “very next step” for a patient with a certain disease).

As such, the most efficient way to learn a large database of clinical facts and knowledge structures in a short period of time is a **scan**. 

A scan is a brief skim of the topic that builds a conceptual skeleton or “big picture” of the topic area. Take for example Internal Medicine, largely considered the hardest rotation aside from Surgery. There’s an enormous breadth of topics to learn in a very short period of time. 

>*Example:* A scan of the Internal Medicine clerkship curriculum consists of:
>1. writing out every topic and a brief description of it. 
>2. clustering topics into logical groupings (e.g. Heart Failure, HTN, Coronary Artery Disease go into the cardiology cluster)
>3. Drawing a map of each cluster.
>	Here, you want to focus on a high-level structure that you can recall by asking questions like: “how shallow is this cluster?” “what shape is ‘heart failure’? how many branches are there that I should know (3: HFrEF, HFpEF, high-output HF)?”

After this, you can hang details on these skeleta as they arise during your questionbanking sessions. 

As said before, exams tend to ask the same kinds of questions. As you gain facility traversing the trees you assemble from your scans, you’ll be able to answer harder questions more quickly. 

# Level 2: The Classical Approach 
Building one’s own medical knowledgebase from the data available falls under what I would call a **classical approach** to learning and practicing medicine. The classical approach assumes the student(s) and teacher to exist in a vacuum with no established knowledgebases or institutions outside the educational interaction.

When the instructor is gone, it is up to the student to rebuild the knowledgebase anew and invoke it on a regular basis in their practice. So that knowledgebase needs to somehow be saved in a portable, platform-independent format that can be referenced at a later time quickly for retrieval. 

>This may seem like a silly exercise, since UWorld, AMBOSS, Boards and Beyond, and Online Med Ed already exist. Surely, they provide sufficient background knowledge to not need to build my own knowledgebase! But I find this assumption flawed personally. I retain things that I deliberately reconstruct. There’s no other option for me realistically. 

So the classical approach entails building a UWorld, an AMBOSS, a syllabus, a didactic curriculum, from scratch. It’s the only way I’d prefer to do it anyhow. 

# Diagrams 
>A diagram is a visual depiction of a disease, disease category, or concept.

Diagrams are generally tables, trees, or circuits when depicting anatomic abstractions, hiearchical concepts, and pathway-based concepts. They are visual maps of concept.

A good map functions like visual code - it can be quickly navigated and executed. 

# Scripts 
>A script is a set of pathologies grouped by chief concern. 

Each script begins with a chief concern then, after discussing its differential diagnosis, delineates a general workup scheme to rule in or rule out the most classic pathologies responsible for the chief concern. 

See [[Scripts]] for an implementation. 

Scripts are advantageous to review on the wards prior to taking care of a prototypical patient with this chief concern. They are *not* advantageous for studying for exams, as they start from chief concern rather than giving the entire vignette with a predefined question type in mind. 

# Rapid Recall
Developing this knowledgebase must occur *entirely separately and with higher priority* than passing/excelling on exams. You can ace exams and be a terrible clinician, as exams test recall in the moment. But, so long as they exist, we need to figure out how to adapt to them. 

The best way to do well on exams is to test yourself regularly. This can happen either by doing practice tests or problem sets that interleave topics, or with intelligently designed [Anki](<Anki Notes>) decks. 

# Postamble
>Once a large-volume knowledge base is created, it must be [managed](<Managing a Large-Volume Knowledgebase.md>) effectively. 

**Knowledge management** is the act of intelligently applying, manipulating, modifying, and extending one’s body of knowledge. Techniques for doing so are described below.

## Iconize 
Once a pattern becomes sufficiently familiar so as to be second-nature, one can optimize its recall and usage by **iconizing** it. In other words, a familiar pattern can be wrapped in a single symbol which acts as a symbolic pointer to the pattern. 

The pattern is then reinforced by using the icon regularly in the appropriate manner in the construction of higher-order patterns.

See an implementation [here](<Iconizing Physiologic and Pathophysiologic Patterns.md>).

## Use it!
Knowledge naturally atrophies (leaks, expires, whatever analogy you want to use) without use. The more you use something you know, the longer you’ll remember and master it.

So whatever you know, look for ways to apply it to your daily life or projects.

One of the lowest-effort ways of doing this is teaching the content. Another easy way to do this is to answer questions on the content. 

A harder but higher-quality method is to build something that solves a relevant problem using this knowledge.

# Sources
>The following are all the sources I’ve used to create my own notes that I’ve then transcribed into the current knowledgebase. 

- Course notes from preclinical courses at [Tufts](<https://www.medicine.tufts.edu>)
- Harrison’s Principles of Internal Medicine 
- Pocket Medicine 
- Online libraries and databases like [uWISE](https://apgo.org/page/uwisev3-2), [UWorld](https://www.uworld.com), [AMBOSS](https://www.amboss.com)
