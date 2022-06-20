---
title: "Check your diagrams"
subtitle: "before you wreck your diagrams"
published: false
image_url: "featured/default.png"
code_sample: false
---

# Doc rot and other laments

This is a post about trying to find a nice way to combat issues around keeping diagrams fresh using “documentation as code”. The motivation comes from complaints you may have heard before from the usual suspects:

> “Our diagrams quickly fall out of sync with the code we have written”— a developer whose diagram is probably too code-y
>

> “Diagramming takes time and our product is evolving quickly, we prefer to spend our time shipping features”— a leader not invested in code transparency
>

I believe that diagrams serve as a tool for you to explain the design of your system and code at an abstract level. They help you get on the same page as your teammates and communicate what you’ve built at [multiple levels of detail](https://c4model.com/).

Despite the importance, many teams setting out with the noble intent of providing helpful information in the form of diagrams are reportedly running into [documentation rot](https://remotejavadev.com/documentation-rot/), in this case a particularly insidious form of rot. This can be observed in anecdotes from the not-so-usual suspects:

> “I shouldn’t need to tell a drawing tool where to place boxes and lines when *ideas* change. I’m just trying to keep an accurate picture of ideas”— an architect tired of drawing diagrams
>

> “[that architect who left a year ago] left the diagram as a JPG in a Powerpoint”— a senior engineer who is reluctant to expand on that diagram
>

> “I can’t catch mistakes in diagrams on our wiki because the diagrams aren’t text-searchable”— AFAIK nobody said this but I think it’s salient
>

Diagrams are uniquely hard to keep fresh. Even with nice tooling like embedded diagrams that can be updated over time, you are still asking your heroic custodians of correctness to switch gears into a box and line drawing tool every time they spot something on a diagram that looks fishy, that is *if they spot it*.

# Diagrams as code to the rescue

Well… sort of to the rescue. A cogent argument that is out there in the industry is that by incorporating tooling like [PlantUML](https://plantuml.com/) into your development workflow, specifications of your diagrams can live as text files alongside your code. As a result, those sweet stewards of your project we call “developers” will keep diagrams tidy at the same time they make code changes. I can see several specific benefits of this:

- When devs are doing ctrl+F refactoring, they might find old domain names that have changed and update diagrams at the same time
- Your diagram changes can be managed by git— for example if adding Service X to your system actually brought your system down, roll back the change to your system architecture diagram while you revert to the last working commit, cool right?
- Nobody has to open up another tool to edit diagrams, they can just do it in the IDE (IntelliJ has a [plugin](https://plugins.jetbrains.com/plugin/7017-plantuml-integration) for visualizing .puml files, I’m certain VScode does as well)
- Diagram files can intermingle with your project structure, [a modeling decision](https://dev.to/stevescruz/domain-driven-design-ddd-file-structure-4pja) in and of itself

These benefits are true to an extent, but assuming that this simple change will fix all the problems is naiive. Here’s why:

- Developers don’t always work that way. This is evidenced by your non-diagram documents, which already live beside your code in GitHub readmes and are already text-searchable, and alas, are still rotting ☹️
- PlantUML is a Java-based rendering engine for diagrams that requires you to either manually render updated diagrams after changes or incorporate some tooling to do so
- Bulk-editing PlantUML by hand isn’t a delightful experience, and principally lends itself to simple name-based refactorings

Beyond these issues of ergonomics, diagrams as code introduce a larger philosophical issue. This matter seems to continue to crop up in otherwise polite cocktail conversation amongst technologists, an idea so evil that I hate to even bring it up here. The line of thinking goes like this: if I have code, and I can represent my diagram as code, why not generate changes to my diagram as I change my code?

This is not a blog post about generated diagrams. Generated diagrams are an insult to the institution of human knowledge.

# A map analogy

Here’s another way to think about it. A diagram is like a map.

This is a useful analogy. A map is a visual representation of a physical space. It gives you named entities like streets, buildings, lakes, etc. It also gives your structural information like relationships of proximity, direction, and relative size.

A diagram is like a map for your code. It can depict named entities in your system, their properties, and how they relate to each other. Like a map of a physical location, a diagram is most useful not because of its completeness, but because of how it abstracts information.

Imagine you were planning a trip to visit all of the Smithsonian Museums in Washington D.C. You wouldn’t want a map of Washington D.C. **so detailed** that it is a paper facsimile of the physical Washington D.C., you would want a map that depicts landmarks and destinations (such as Smithsonian Museums), maybe you would also want a map depicting public transit offerings.

The best exact picture of your codebase is the codebase itself. This has been said in other words by some smart people. [Readable code is the best documentation](https://martinfowler.com/bliki/CodeAsDocumentation.html), and [unit tests (read, “examples”) are the best documentation](https://capgemini.github.io/development/unit-tests-as-documentation/). If you want the most current, most detailed representation of what a system is or does, then you should read the code. That isn’t what a diagram is good for. Diagramming is about distilling an abstract mental model of the system into something that is useful for other people. It is useful in the way it limits information.

The issue is, a map can decrease in usefulness over time. For example, a map of the [Soviet Union](https://en.wikipedia.org/wiki/Soviet_Union) would not be useful if you were trying to travel to Russia today. This is a version of the very same problem we are trying to solve around diagrams. A diagram doesn’t cease to be useful any time that the system changes, but when it stops being fundamentally correct picture of the system.

# I propose “Diagram-driven development”

Stay with me. Let’s review what has been covered.

- Diagrams are abstractions of our code/system design that limit information in a useful way
- Diagrams fall out of sync with reality as the system changes and become less useful
- Keeping diagrams close to code and in a code-like format helps prevent this but not completely
- Generated diagrams are a scourge

We want diagrams as code, but we also need a way to validate that our code changes haven’t broken the correctness of our diagrams. An attentive reader would jump to the conclusion that we need some tooling that can detect incorrectness in our diagrams and flag it, but hold your horses!

Diagram validation is diagram generation in reverse. Its inversion has no such effect on the underlying evil, and it should not be attempted.

No, keeping diagrams correct is a matter of discipline, not automation. There is also a discipline practiced around [software testing](https://en.wikipedia.org/wiki/Test-driven_development) that isn’t just about catching mistakes but also about improving the design of software. I propose a similar workflow here for diagrams.

1. Create or update an existing diagram in PlantUML or [Mermaid](https://mermaid-js.github.io/mermaid/#/) (I advocate for Mermaid because it can render your diagrams in markdown directly in your GitHub readme)
2. Write some code and commit it
3. Look back at your diagram, is it correct and useful?
4. If not, was your original design wrong? Or is the code wrong? React accordingly
5. Push your changes and create a PR (a GitHub action could happen here that calls out changes you made around diagrams and asks your teammates to review the diagrams before reviewing the code)

“Diagram-driven development”, as I’ll call it, is about coding with intention and leading development efforts with design. Your diagram is like a failing test for your code that you and your team can agree is passing once it is correct and useful. Seeking out the diagrams you need to change as a part of making your change can help you understand the task at hand and plan your approach. If you practice updating diagrams as a prerequisite to any change, you will get better at creating and updating diagrams, and adopt [tooling and workflows](https://www.thoughtworks.com/radar/techniques/diagrams-as-code) around it.

DDD (shh! let’s pretend there isn’t already [one of those](https://martinfowler.com/bliki/DomainDrivenDesign.html)) is not the same as [Model-driven development](https://www.archetypesoftware.com/blog/rebel-bitbucket). It doesn’t rely on diagram generation, code generation, or automated validation. It relies on human discipline and prioritizes utility and correctness over completeness. The end result will be a better picture of a system as it changes over time.