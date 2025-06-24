---
layout: post
title: "Paper Review: *Man-Computer Symbiosis* by J. C. R. Licklider, 1960"
date: 2025-06-19
description: |
    *Man-Computer Symbiosis* is a short paper describing Licklider's vision for a future of computing in which men and machines are tightly linked in a cooperative relationship
draft: true
tags:
  - paper review
---


## [Man-Computer Symbiosis](https://worrydream.com/refs/Licklider_1960_-_Man-Computer_Symbiosis.pdf)
### J. C. R. Licklider, 1960

*Man-Computer Symbiosis* is a short paper (8 pages) describing Licklider's vision for a future of computing that he expects to come about in roughly 1965 (spoiler: it did not).

> The main aims are 1) to let computers facilitate formulative thinking as they now facilitate the solution of formulated problems, and 2) to enable men and computers to cooperate in making decisions and controlling complex situations without inflexible dependence on predetermined programs.

In my opinion, the paper says too much and also too little.  The paper goes into detail about technological challenges and "solutions" which are, obviously, dated today, and does not go into explicit detail about what is meant by the aims described above, how they are not being achieved by the technology of the time, and what their anticipated benefits would be.

The paper opens (Part 1, Section A) with a description of the symbiotic relationship between the fig tree and *blastophaga grossorum* (now commonly known as *blastophaga psenes*) or "fig wasp" which pollinates it.  I am, personally, a big fan of papers that open with a discussion of a topic that is not, strictly speaking, related, so this was a fun part of the introduction for me.

> The hope is that, in not too many years, human brains and computing machines will be coupled together very tightly, and that the resulting partnership will think as no human brain has ever thought and process data in a way not approached by the information-handling machines we know today.

It's difficult for me to argue that in the year 2025, when I read this paper and am writing this summary, that human brains are *not* thinking differently from how they were in 1960 or that close association with computers did not have a great deal to do with that.  And while the computers of today do, literally, process data "in a way not approached by the information-handling machines" of 1960, it is more in a "smaller, faster" way than what I assume Licklider was imagining when he wrote this sentence.

In Part 1, Section B: "Between 'Mechanically Extended Man' and 'Artificial Intelligence'", Licklider distinguishes his vision of man-computer symbiosis from J. D. North's notion of the "mechanically extended man" (a man's own faculties are extended by tools like hammers, microscopes, etc) and "semi-autonomous systems" (which are *mostly* autonomous, but require human supervision and occasional intervention).  Licklider also references (but does not cite) a study for the Air Force that suggested by 1980 computers could be doing all the thinking and problem-solving "of military significance" when indicating that the end of man-computer symbiosis would be when machines take over performing all cognition.  Analysis of this claim is subject to how one defines "thinking and problem-solving", but I would argue that we humans are still doing a lot of cognition today, which implies (by Licklider's thinking) that if we have not achieved man-computer symbiosis (subject for debate) it would be worthwhile to do so.

Part 2: "Aims of Man-Computer Symbiosis" is the shortest and yet most interesting of all part of this paper.  In three paragraphs, Licklider expounds upon the aims of man-computer symbiosis but, curiously, they appear to be distinct from the two aims laid out in the abstract.  The necessary developments in computing machinery stated in Part 2 appear to be:
1. Helping the user "think through" a problem.  Licklider says problems "would be easier to solve ... through an intuitively guided trial-and-error procedure in which the computer cooperated, turning up flaws in the reasoning or revealing unexpected turns in the solution."  Licklider also adds "One of the main aims of man-computer symbiosis is to bring the computing machine effectively into the formulative parts of technical problems."
2. "...bring computing machines into processes of thinking that must go on in 'real time'."  Licklider bemoans the days-long feedback loops of 1960 computing.

Reading these goals, I immediately thought of Clojurists' notion of "REPL-driven development"; which is to say developing a solution by experimenting in the rapid feedback loop of a REPL.  Indeed, this is nearly as close to "real time" as one could imagine, is a "trial-and-error procedure", and, arguably, helps one "think through a problem".  However, I'm not sure this flow is "bringing the computing machine into the formulative parts of technical problems" in the way Licklider envisioned.  In the context of the rest of the paper, it seems that Licklider is expecting the computer part of the symbiote to ask questions and refine a vague goal posed by the user; to guide their thinking.  A REPL certainly does not do this (though there's an interesting consideration to be made as to "what if it did?"), but an LLM such as ChatGPT can perform this *if appropriately prompted*.  I think that realistically the true vision here would be a system where an LLM has access to the facilities of a computer and, when given a goal by the user, will ask clarifying questions as it attempts to perform the task.  I have not seen an LLM employed in this way, but it stands to reason that it should be imminently feasible.

In Part 3: "Need for Computer Participation in Formulative and Real-Time Thinking", Licklider attempts to justify why computers would be useful in the tasks of knowledge workers, which fortunately I did not need to be convinced of.  He makes the argument that "men" (Licklider uses the term "man" and "men" for historical reasons, but the terms can refer to people generally) are good at some kinds of things whereas computers are good at others, and therefore the work of their symbiote would be better or more efficient than the work of only one or the other, but the point is made clumsily.  For one, he argues that men are largely parallel while computers are sequential, which today has some truth to it (less than in 1960), but why this matters is not made clear.  Another point Licklider makes in this argument is that men are good at responding to unforeseen circumstances, which is true, but it is something that he also advocates for as a development for computers, which would eliminate it as a distinguishing characteristic.

Part 4: "Separable Functions of Men and Computers in the Anticipated Symbiotic Association" is a natural continuation from Part 3 in which Licklider specifies which tasks he thinks are best suited to humans ("men") and which are best suited to computers.

> Men will set the goals and supply the motivations, of course, at least in the early years.  They will formulate hypotheses.  They will ask questions.  They will think of mechanisms, procedures, and models.

> In general, they will make, approximate and fallible, but leading, contributions, and they will define criteria and serve as evaluators, judging the contributions of the equipment and guiding the general line of thought.

> In addition, men will handle the very-low-probability situation when such situations do actually arise.

The idea that humans should define goals and assess outcomes comports neatly with how people use LLMs today.  The ways in which humans handle "very-low-probability" situations is typically that they are paged, which is not ideal but does conform to the stated expectation.

> The information-processing equipment, for its part, will convert hypotheses into testable models and then test the models against data (which the human operator may designate roughly and identify as relevant when the computer presents them for his approval).  The equipment will answer questions.  It will simulate the mechanisms and models, carry out the procedures, and display the results to the operator.

An interesting aspect of all of this to me is that Licklider expects the computer to perform some generally human tasks (e.g. discern intent) and *also* standard machine tasks (e.g. simulate models, display results).  The role Licklider is describing for machines does not feel akin to the way either conventional (let's say, pre-2020) tools or LLMs are used today.  We do not typically start with a question, e.g. "Given this body of data, can I accept or reject my hypothesis?", and then let a tool or LLM run the analyses and provide an answer.  Indeed, traditional tools (e.g. R, Python) require more specific instructions, and LLMs may attempt to answer the question but are liable to give a wrong (and, regardless, unjustified) answer.  *Instead*, I think Licklider's idea could be achieved if the computing machine, let's say LLM, took your question and transformed it into "testable model" (e.g. a Python program) and ran it, displaying the results.  There's a deeper question posed by the fact that, today, the program itself is typically the goal in itself.  Even following this formula, one ought to exercise skepticism that the result produced by the LLM is correct as they are prone to making subtle mistakes.  Perhaps that is indicate of the entire idea's Achilles' heel, which is trusting that a computer correctly understood the intent and that it's attempt to achieve the conveyed goal is not wrong.  We are accustomed to fellow humans making errors, but we are not accustomed to (and I am downright opposed to) machines making errors.

The final part, Part 5: "Prerequisites for the Realization of Man-Computer Symbiosis" mostly discusses technical challenges of the day, such as the size and speed of computers, limited memory, unintuitive interfaces, etc.  However, there are a few sections that merit consideration.

As a programming language enthusiast, my attention was naturally drawn to Section D: "The Language Problem".  In this section, Licklider highlights two issues with communication between people and machines:
1. Machine language is very different from human language.
2. "Men appear to think more naturally and easily in terms of goals than in terms of courses."

On the first point, Licklider notes that "great strides have already been made, through interperative programs and particularly through assembly and compiling programs such as FORTRAN to adapt computers to human language forms."  I'm sure Licklider would be delighted to know that the programming languages of today are much more natural and approachable than FORTRAN, despite not being identical to human language.  Though, that is in fact for good reason, as human language is riddled with ambiguity that is best avoided when instructing machines.

As for the second point, I was reminded of logic programming (e.g. Prolog, Datalog, miniKanren, etc), where programs are executed by the user stating a goal (the term "goal" is literally used) and the program attempting to achieve that goal.  Now, logic programming is very lovely, but I think Licklider would bulk at the degree to which a programmer must provide procedures by which the computer solves said goals.  A large library of logic programs would be a formidable ally generally, though logic programming is also plagued by being terribly slow.

The other section I wished to discuss is Section E: "Input and Output Equipment", of which there are three subsections: *Desk-Surface Display and Control*, *Computer-Posted Wall Display*, and *Automatic Speech Production and Recognition*.  In the first subsection, Licklider states:

> Certainly, for effective man-computer interaction, it will be necessary for the man and the computer to draw graphs and pictures and to write notes and equations to each other on the same display surface.

In 2025, we have all the requisite technology for this to be the case, but it's difficult to argue that we have an interface as described.

After this, the paper ends rather abruptly, without conclusion.
