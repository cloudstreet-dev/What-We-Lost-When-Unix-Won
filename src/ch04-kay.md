# Alan Kay's Actual Proposal

Alan Kay is a difficult subject for this book. He is one of the most cited and least correctly understood figures in computing. He has spent fifty years talking and writing about ideas that the industry has, by Kay's own assessment, largely failed to absorb. He is associated, in popular memory, with the invention of object-oriented programming, the invention of the personal computer, the invention of the graphical user interface, and a kind of computing-as-humanism that the industry never built. Some of those associations are accurate. Some are not. Most are accurate in a sense Kay would describe as superficial.

This chapter is not going to try to do Kay justice in the sense of summarizing his views; Kay has done that many times and is still doing it. The chapter is going to do something narrower and more useful: extract the *actual technical and intellectual proposal* that the Smalltalk research at Xerox PARC represented, separate it from the museum-piece version that survived in the textbook treatment of object-oriented programming, and put it in the context of the operating-systems argument this book is making.

The reason this matters is that Smalltalk lost, in much the same way the Lisp Machines lost, and for some of the same reasons and some different ones. The reason its loss is harder to see is that pieces of it kept surfacing in commercially successful systems — the Mac, Java, Ruby, the modern web browser — in forms diluted enough that, by 2026, "object-oriented" means something almost unrelated to what Kay meant by the term, and the parts of Smalltalk that Kay considered essential are mostly absent from systems that nevertheless claim descent from it.

## The Dynabook and Personal Dynamic Media

The frame Kay worked inside in the early 1970s was not "we are inventing a new operating system." It was a frame about *what computers are for*. The frame was articulated most clearly in two documents: a 1972 paper by Kay called "A Personal Computer for Children of All Ages," which described a portable, networked, child-usable computer called the Dynabook; and a 1977 paper Kay co-wrote with Adele Goldberg, "Personal Dynamic Media," which described how a computer like the Dynabook could function as a medium in the sense that print, film, and television were media.

The frame is worth taking seriously because it constrains everything else about Smalltalk. Kay was not trying to build a better tool for programmers. He was trying to build a *medium for thought*, and his model for what a medium did was the printing press: a technology that, by being available to everyone and by giving everyone the means of authorship, changed how civilizations thought. Computers, in Kay's view, had the same potential, and most contemporary computer designs — including the timesharing systems of the era and the workstation systems being designed at PARC's other groups — were misusing that potential by treating computers as tools rather than as media.

A medium, in Kay's sense, had three properties that mattered for the design.

It was *active*. A printed page sat there; a Dynabook page could compute. The content was not just data but data and behavior together, simulating, responding, transforming.

It was *malleable*. The user did not just consume the medium; they authored within it. A child reading a book about geometry on a Dynabook could change the geometry, ask it questions, build their own simulations on top of it. The medium had no first-class consumers and second-class producers. Everyone was a producer.

It was *integrated*. The medium did not consist of separate applications stitched together. The text, graphics, sound, simulation, and program code were all the same kind of object, interoperable by default, navigable by uniform means. The system did not have a word processor, a drawing program, and a calculator as separate things. It had a substrate in which a document could contain words, drawings, calculations, and simulations as different shapes of the same underlying object.

The Dynabook hardware never shipped as Kay envisioned it. The form factor — book-sized, portable, networked, with high-resolution display and stylus or keyboard — was approximately what laptops became, twenty years later, after fits and starts. The software substrate — the part that mattered to Kay more than the form factor — was Smalltalk, and Smalltalk did ship, in successive forms, on the workstations PARC could afford to build.

## Smalltalk-72: language as performance

Smalltalk did not begin as a finished proposal. It began as a series of experiments, and the experiments differed enough that the dialects each carry a different intellectual emphasis. The earliest version that ran on the Alto, Smalltalk-72, was almost unrecognizable compared to later Smalltalks.

Smalltalk-72 took the message-passing idea so seriously that it dispensed with syntax in any normal sense. An object received a stream of tokens. The object decided, by examining the tokens, what to do with them. There were no fixed grammar rules at the language level; each object parsed its incoming messages itself. This made the language extensible in directions no later mainstream language has been. New constructs — conditionals, loops, control structures — could be defined by users by writing objects that parsed the right token streams.

The cost was that Smalltalk-72 was idiosyncratic and hard to learn. Different objects responded to messages with different rules, and reading code required understanding which object was receiving and how it parsed. The expressiveness was paid for in legibility. By 1974, the group was building Smalltalk-74, which began to constrain the message-passing model — fixed selectors instead of arbitrary token streams — and Smalltalk-72 was retired as an active platform.

The Smalltalk-72 experiment is worth remembering because it shows what *taking message-passing as a worldview* looks like when carried as far as it can go. The mainstream object-oriented languages that descend from later Smalltalks — Java, C++, Python — preserve a much weaker version of message-passing, where method dispatch is the only message-like operation and the language's other structures (control flow, declarations, types) are syntactic features the user cannot extend. Smalltalk-72 said that even the syntactic features were objects' responses to messages. It was an extreme position, and the moderation that came in Smalltalk-76 made the language usable, but moderation also made it less radical than it had begun.

## Smalltalk-76: the language coheres

Smalltalk-76 was the version where the language we now think of as "Smalltalk" came into focus. Dan Ingalls, Adele Goldberg, Diana Merry, Ted Kaehler, and the rest of the team — Kay was the intellectual leader but a smaller fraction of the implementation — wrote a Smalltalk that ran efficiently on the Alto, supported a complete bitmap graphical user interface, and shipped with an integrated browser, editor, debugger, inspector, and a library of system code that the user could read and modify.

The technical shape of Smalltalk-76 deserves naming because it is what later Smalltalks elaborated rather than replaced.

*Everything is an object.* Integers are objects. Booleans are objects. Classes are objects. The compiler is an object. The execution stack is an object. There is no primitive type that is not an object. Comparisons, arithmetic, control flow, and reflection are all uniform: send a message to an object and the object responds.

*Computation is message-passing.* The only operation is sending a message. An expression like `3 + 4` is sending the message `+` with argument `4` to the object `3`. An expression like `aCollection do: [:each | each print]` is sending the message `do:` to `aCollection` with a block of code as argument. There is no statement form distinct from expression form. There is no operator dispatch separate from method dispatch. Everything is method dispatch.

*Behavior lives in classes; classes are objects.* Each object has a class. Each class has its own methods. Method lookup follows the inheritance chain. A subclass inherits methods from its superclass and can override them. Classes themselves are objects, instances of `Metaclass`, with their own methods and their own metaclass. This regularity — turtles all the way up — is one of the durable properties of Smalltalk. Java has classes-as-objects in a limited form (reflection); Python has it more fully; Ruby has it more fully still. Smalltalk had it from the beginning, without bolts.

*The system is its own library.* The Smalltalk environment shipped with the source for the language's libraries and tools as Smalltalk classes that the user could browse, modify, and recompile from within the running system. The class browser was the central tool. You navigated the system by navigating its class hierarchy. You learned the system by reading its source through the browser. The system's source was, in a strong sense, the system's documentation.

*The user interface is part of the language.* Smalltalk-76 had a bitmap display, overlapping windows, a mouse-driven interaction model, and a set of widgets — text editors, browsers, inspectors, debuggers — that lived in the same Smalltalk image as user code. The UI was not a separate subsystem with its own programming model. It was a library of classes that the user could read, extend, and replace. The same uniformity that applied to the language applied to the system the language presented to the user.

This is the Smalltalk that made the case Kay had been making since the 1960s. It was a complete, working environment in which the medium ideas — active, malleable, integrated — were demonstrated rather than theorized. By 1979, Smalltalk-76 was being used inside PARC for real work: by the kids in the Learning Research Group's tutorials, by PARC employees designing user interfaces, by researchers building simulations and education prototypes.

## Smalltalk-80: the public version

Smalltalk-80 was the version Xerox released to the outside world, in a deliberate effort to seed the system in other research institutions and companies. It was published as a set of books — the "blue book" describing the language, the "purple book" describing the implementation, the "orange book" describing the system's design — and a portable specification for a virtual machine that anyone could implement.

Smalltalk-80 made the system available but it also made a quiet, important compromise. The Smalltalk-80 image, as published, was a more conservative version of Smalltalk-76, hardened for portability and for the assumption that the receiving institutions would not have PARC's specific hardware. The integrated bitmap display, the mouse-driven interface, and the assumed-always-available high-resolution graphics had to be portable across whatever workstations Sun, Apollo, HP, and the universities were running. The result was a Smalltalk that could be ported widely — and was, to dozens of platforms — but that on most platforms ran inside a window of someone else's operating system, rather than being the operating system itself.

This is a subtle loss that matters for the operating-systems argument of this book. On the Alto, Smalltalk *was* the operating system. The Alto booted into the Smalltalk image. There was no host OS underneath. The system Kay's group had built was a complete environment, not an application. On a Sun workstation in 1985 running Smalltalk-80, the same image ran inside a window on top of SunOS, and the user's experience was of a guest environment, not a host one. The integration that Smalltalk-76 had achieved at the system level was, for most Smalltalk-80 users, an integration only within the Smalltalk window.

This is the form in which Smalltalk reached most readers of this book, if they have encountered it at all. It is the form in which the academic study of object-oriented programming encountered it. The Smalltalk-80 books were enormously influential — they were how Bjarne Stroustrup, who designed C++, learned about classes and inheritance; how the original Java team, which included James Gosling, formed its understanding of OO; how Ruby's Yukihiro Matsumoto thought about message passing. But the form in which Smalltalk-80 traveled out into the world was already a downsized version of what PARC had built, and the further dilutions that happened as the ideas moved through C++ and Java were dilutions of an already partial picture.

## Message-passing as worldview

The single most important idea Kay tried to communicate, and the one that has been most thoroughly misunderstood by the industry, is the idea that *message-passing is a worldview, not a feature*.

The dominant industry interpretation of object-oriented programming, by the late 1990s, was: data and the functions that operate on it should be packaged together as objects, and methods on those objects should be dispatched dynamically based on the object's class. This is, technically, what Smalltalk did. It is also a description in which the central idea is missing.

The central idea, in Kay's framing, is that an object is a small computer. Objects do not share memory. Objects do not call each other's functions. Objects send each other messages, and the receiving object decides, internally and privately, what to do with each message. The system as a whole is not a program but a network of these small computers communicating by sending messages. The closest analogy Kay used was biological cells: discrete, encapsulated, communicating only by signaling, never by reaching into each other's interior.

The reason this matters is that the worldview implies a very different model of computation than the one C++ and Java made standard. In Kay's model:

*Encapsulation is total.* An object's internal state is invisible to other objects, not by convention but by architecture. Other objects cannot read an object's fields. They can only ask, by sending a message, and let the recipient decide what to answer.

*Polymorphism is universal.* Any object can respond to any message, so long as it knows how. There is no compile-time check that the receiver supports the message; the runtime asks the object, and the object responds or signals a "doesn't understand" condition. This is structurally what duck typing in Python or Ruby is. Static-type-system devotees consider this a weakness; in Kay's view it was the point. The interactions between objects should be discoverable at runtime, not committed to at compile time.

*Composition is by message.* You build a system not by writing functions that call other functions but by setting up arrangements of objects that send each other messages in response to user actions and other messages. The control flow is in the messages, distributed across the participants.

*Late binding is everywhere.* The decision about what code will run in response to a message is made as late as possible: at the moment the message is received. This is what makes the system extensible — new objects can plug in and respond to existing messages without recompiling anything — but it is also what makes the system feel different from a statically-compiled program. The behavior emerges from the runtime configuration, not from the source code in any one place.

When Kay later said, in a famous interview, that C++ was not what he had in mind when he invented the term "object-oriented" — and that nobody had asked him to define the term — he was not being eccentric. He was pointing out that the term had come to denote a feature set (classes, inheritance, virtual methods) rather than a worldview (objects as small computers, messages as the only operation, late binding as the default).

The worldview is the part that matters for the operating-systems argument. A system organized around small computers communicating by messages is structurally different from a system organized around files, processes, and pipes. The Smalltalk environment was one possible expression of the worldview. The web — which we will return to in Chapter 10 — turned out, by accident, to be another, with HTTP playing the role of the message-passing protocol and HTML documents playing the role of the receiving objects. The worldview in pure form survives in some research languages and in agent-oriented frameworks; it does not survive in any of the systems that became standard.

## What PARC did and did not get right

Smalltalk was not perfect. Some of its failures are worth naming.

The performance gap was real. On the hardware of the late 1970s, Smalltalk was slow compared to compiled C. The virtual machine was efficient by interpreted-language standards, but it was an interpreted language, and the performance penalty showed in graphics, numeric computation, and large-scale data processing. Later JIT compilation techniques — pioneered for Smalltalk dialects like Self and then transferred to Java HotSpot and to JavaScript in V8 — closed the gap, but those came too late to help Smalltalk's market position.

The single-image model was hard to deploy. If your application was an image, packaging and distributing it was awkward. You could ship the whole environment to users, but the whole environment was tens of megabytes and required the Smalltalk VM as a runtime. Compared to shipping a C executable, this was substantial overhead. Even today, Pharo and Squeak applications struggle with the same issue.

The standalone-application model was missing. Smalltalk's vision was that the user would live in the image and customize it. Industry wanted users to launch applications, do work, and quit. The image model resisted this kind of compartmentalization. Commercial Smalltalk environments — Digitalk's Smalltalk/V, ParcPlace's VisualWorks — added apparatus for packaging Smalltalk applications as more or less standalone programs, but the seam was always visible.

Kay himself, in retrospective talks since the 1990s, has expressed frustration with the implementation choices the team made and would have made differently — he has, for instance, questioned whether class-based inheritance was the right model, and pointed to prototype-based systems like Self (which came out of PARC and was a strong influence on JavaScript) as closer to what he was after. The Smalltalk that was shipped, in his view, was a snapshot of a design conversation that he wished had gone several more rounds.

## What the worldview made imaginable

Despite these limitations, the Smalltalk worldview made imaginable things that no other contemporary system did. Three are worth mentioning:

*The graphical user interface as we know it* was substantially developed in Smalltalk, on the Alto, by Smalltalk programmers using Smalltalk as the medium. The Macintosh team that visited PARC in 1979 saw the Smalltalk environment and went home with a set of ideas — overlapping windows, the mouse, click-and-drag manipulation, menus, the desktop metaphor — that they re-implemented in the language Apple was building for the Mac. The Macintosh user interface is, at one remove, a Smalltalk artifact, even though the Mac itself ran neither Smalltalk nor anything like it.

*Object-oriented programming as a discipline* became, despite the dilutions, a working idiom for industry. The Smalltalk books trained two generations of language designers in a way of thinking that became the default for new languages from 1985 through 2010. Even languages that did not adopt the message-passing worldview adopted its surface — classes, methods, inheritance, polymorphism — as the standard vocabulary. The vocabulary persists.

*The integrated development environment* — particularly the modern IDE with live error reporting, interactive debugging, code browsing, and refactoring tools — descends in part from Smalltalk's class browser and debugger. Eclipse and IntelliJ are, at one remove, Smalltalk-environment ideas embedded in a more conservative file-and-process model.

The next chapter looks at the parts of Smalltalk that did not get diluted: the image, Morphic, and the living descendants in Squeak, Pharo, and Glamorous Toolkit, where the worldview is still being practiced by people who did not give up. Then the book turns to Plan 9, which approached some of the same problems — uniform composition, namespace-as-system — from the Unix side. Smalltalk and Plan 9 are not usually mentioned in the same paragraph; they should be, because they are two of the strongest attempts at coherent system design, by very different cultures, in the same fifteen-year window.
