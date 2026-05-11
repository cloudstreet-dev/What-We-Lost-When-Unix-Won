# Multics — What Unix Was a Reaction Against

The standard treatment of Multics in Unix folklore is as the bloated, over-engineered, committee-designed predecessor that Unix improved on by being smaller and simpler. This treatment is half right and entirely misleading. Multics was over-engineered for its hardware, costly to develop, and consumed more institutional resources than its sponsors were willing to keep paying for. It was also, on its merits as an operating system, more advanced than anything its successors produced for the next two decades. Several of its design decisions were correct, were abandoned by Unix for hardware reasons, and were eventually reinvented — usually poorly — in later systems. This chapter is about taking Multics seriously as a design rather than as a cautionary tale.

The historical frame is this. In 1965, a consortium of MIT, General Electric, and Bell Labs began work on the Multiplexed Information and Computing Service, or Multics. The goal was to build a *computer utility*: a single machine that could serve hundreds of users simultaneously, the way the electric utility serves hundreds of homes simultaneously, with strong protection between users, full virtual memory, dynamic linking, and a uniform file system. The system would run on custom hardware — the GE-645, later the Honeywell 6180 — designed in close collaboration with the operating system team. Multics shipped in stages from 1969 onward and continued to run, at sites across the world, until the last installation was decommissioned in 2000.

Multics did not become the dominant operating system of its era. Bell Labs withdrew from the project in 1969, judging the cost-to-benefit unfavorable; the Bell researchers who left went on to write Unix. General Electric exited the computer business in 1970 and sold its computer division to Honeywell. Honeywell shipped Multics commercially through the 1980s, with a peak install base of about eighty sites — universities, government agencies, the Air Force, a handful of corporate customers. Compared to Unix's eventual ubiquity, this is a rounding error. Compared to most operating systems' install bases, it was a respectable run for a system that was, at every point in its life, complex and expensive.

## What Multics actually proposed

A clean way to read Multics is as a system designed to answer the question: *what would an operating system look like if it took mainframe-style timesharing seriously and added everything the engineers of the era thought a computer should be?* The answer was: it would have rings, segments, the file system in memory, dynamic linking, a high-level systems language, and a uniform protection model.

These are the parts of Multics that deserve attention now.

### Rings of protection

Multics introduced the *protection ring*, an idea that has since survived in some form into every general-purpose CPU architecture. The Multics rings were eight nested levels of privilege; code running at a higher ring (ring 7, least privileged) could not access data or call procedures at a lower ring (ring 0, kernel) except through carefully checked gates that the lower-ring code had explicitly exposed. The system used the rings to separate the kernel, the trusted utilities, the user's own privileged libraries, and the user's ordinary application code into distinct privilege domains.

The point of the rings was not just security; it was *graduated trust*. Unix has a binary distinction: kernel mode and user mode. Either you are the kernel and you can do anything, or you are user-mode and the kernel checks every system call you make. Multics had a finer distinction. The user could write code at ring 4 that called code at ring 3, where the ring 3 code had more privileges to access certain resources and could enforce its own protection rules on the ring 4 code's behavior. A library could be its own little kernel, protecting its data from its callers without being part of *the* kernel.

This is one of the Multics ideas that nominally survived — x86 processors have four rings, and ARM has its own privilege hierarchy — but in practice does not get used. Operating systems use ring 0 and ring 3 and leave the intermediate rings unused. The reasons are partly historical (Unix-derived systems were designed around a two-ring model), partly performance-related (each ring transition has a cost), and partly cultural (writing software for an eight-ring system requires a discipline most software is not written with). The Multics idea was that protection should be a continuous design space, not a binary; the systems that came after settled on the binary.

### Segments and the unified address space

The Multics memory model was based on *segments*. A segment was a named, variable-length, individually-protected region of memory. Each running process had access to a collection of segments — its code segment, its stack segment, segments containing data, segments containing shared libraries. The hardware supported direct addressing within segments, with the segment number and offset together forming the virtual address.

The crucial feature was that the file system was, structurally, a hierarchical namespace of segments. A "file" on Multics was a segment. When you wanted to read a file, you did not call an I/O routine to load its contents into a buffer in memory. You *referenced* the segment, and the hardware paged the relevant portions into physical memory on demand. The file system *was* memory, in a strong sense: opening a file meant adding it to your address space; reading a file's contents meant dereferencing pointers into the segment.

This is *memory-mapped I/O for everything*, as the default. Unix has `mmap` as an optional system call you can use to map a file into memory. Multics had the converse arrangement: the file system was always memory-mapped, and the byte-stream `read/write` model that Unix used was, on Multics, an unnecessary indirection.

The advantage was that programs could treat files as if they were data structures in memory, because they were. A program could traverse a linked list across multiple "files" by following pointers; the hardware would page in the segments containing the pointed-to data as needed. There was no impedance mismatch between in-memory data structures and persistent data structures. This is one of the things that, in retrospect, looks like a bet on a future that never quite arrived — persistent memory, mapped object stores, the long-running effort to "make everything a database" — and that Multics shipped fifty years ago.

The cost was that the model required substantial hardware support: large virtual address spaces, robust paging, hardware segmentation. The GE-645 had this hardware; the PDP-7 and PDP-11 that Unix grew up on did not. Unix's byte-stream model was, in part, what you arrived at when the hardware made the Multics model unavailable. By the time hardware caught up — by the late 1980s, virtual memory and paging were standard on workstations — the byte-stream model was so embedded in software conventions that no one rebuilt on top of segments.

### Dynamic linking

Multics had dynamic linking from the beginning. When a program called a procedure in a library, the call was not resolved at compile time. The linker would, at the moment of the first call, locate the library segment, fault it into memory if it was not already loaded, look up the procedure's entry point, and patch the call site so that subsequent calls would go directly to the procedure. Subsequent calls had no per-call overhead.

The implications were structural. Updating a library meant replacing the library segment; the next process that ran would link to the new version automatically. Sharing a library across processes was free; the segment was already in the file system, and each process's address space could reference it without copying. Patching a running system meant updating libraries; the running processes would link to the new versions on their next call to the changed code.

Unix did not have dynamic linking initially. Early Unix programs were statically linked: the libraries were copied into the executable at link time, and updating a library required rebuilding every program that used it. Shared libraries came to Unix in the 1980s and 1990s — first as SunOS .so files, then as a portable standard through ELF and the various implementations of `ld-linux.so`. The Unix shared-library mechanism, by 2026, is functional but has noticeably more friction than Multics had: the various library-versioning rules, the failure modes when versions mismatch, the surface for security vulnerabilities exposed by the loader, the build-time complexity of writing libraries that can be shared. Multics had paid the price up front and gotten a cleaner mechanism.

### PL/I as the systems language

Multics was written in a subset of PL/I, IBM's general-purpose language. The choice was, in retrospect, distinctive. PL/I in 1965 was an enormous, ambitious language designed to subsume Fortran, COBOL, and ALGOL; the subset Multics used was significantly smaller, but it was still high-level. It supported structured programming, complex data types, multi-dimensional arrays, and exception handling, at a moment when most operating systems were written in assembly or a thin macro language over assembly.

Writing an operating system in a high-level language was, in 1965, controversial. It was widely believed that the performance cost would be unacceptable and that the compiler would not be able to produce efficient code for the cases that mattered. The Multics team's bet — that the productivity gains and maintainability of high-level code outweighed the performance cost — was the same bet Unix would make a few years later with C. The difference was that Multics's PL/I was a heavier, more featureful language than Unix's C, and the bet looked more expensive when Multics made it than when Unix made it. The lesson, in retrospect, is that the bet was correct in both cases; the choice between PL/I and C was a matter of how aggressive the bet should be, not whether to take it.

The PL/I-versus-C question is also one of the cleaner cases where Unix's "simpler is better" instinct was directly load-bearing for adoption. Bringing up Unix on a new machine required bringing up a C compiler. Bringing up Multics on a new machine required bringing up a PL/I compiler. C compilers were dramatically easier to write. Once the C compiler existed, Unix followed; once the C compiler did not exist, Multics did not get ported.

### Single uniform file namespace, hierarchical

Multics had a hierarchical file system before Unix did. The Unix hierarchical file system — directories containing files and other directories, slashes as separators, a single root — is one of Multics's clearest direct contributions. Multics also had file attributes (creation date, modification date, access control), per-file access control lists with named principals, and a uniform naming scheme for everything in the system. The Unix file system inherited much of this and simplified some parts. The simplifications were not all improvements: Unix's user-and-group permission model is significantly weaker than the ACL model Multics shipped with, and the cost of that simplification was an entire industry's effort to bolt ACLs back on (POSIX ACLs, SELinux, AppArmor, the various security models in Linux distributions of the 2020s).

## Why Multics was hard

Multics was famously expensive to develop, slow to ship, and resource-intensive to run. The reasons are worth being specific about because they explain why "Multics" became a kind of object lesson in the Unix folklore.

The system was extraordinarily ambitious for the hardware of the period. Running Multics required custom hardware support for segmentation, paging, and rings. The GE-645 was not a commodity machine; it was, in effect, designed around Multics's requirements. This made the system expensive both to develop (the hardware and software teams had to coordinate closely) and to deploy (you needed a specific class of machine).

The system supported many things at once — virtual memory, dynamic linking, multi-language support, hierarchical access control, multi-level security — and the interactions between those features produced a complex code base. Bell Labs, in 1969, judged that the complexity exceeded what the institution could maintain for the value being delivered, and pulled out. This is the institutional moment that shapes the Unix folklore. The Bell Labs engineers who walked out of Multics carried with them, accurately, the impression that Multics had been *too much*. Unix's premise — small, simple, focused — was a direct reaction to that impression.

The system was hosted on a small number of expensive computers and never had a path to commodity hardware. The Multics architecture assumed a specific class of hardware that GE and Honeywell could produce in modest volumes for high prices. There was no Multics for a minicomputer, much less a microcomputer. As the industry shifted toward less expensive machines, Multics had no platform to follow it to.

The licensing and commercial story was the opposite of Unix's. Multics was a commercial product of Honeywell, sold to large customers at substantial prices, with the kind of support contracts that Honeywell's mainframe business was built around. Universities had a few sites — MIT, the original — but the system was not cheap or easy to spread the way AT&T's near-free Unix licenses spread.

## What Multics got right

Reading Multics in 2026, with the experience of a half-century of operating systems to compare it against, several of its commitments look prescient.

*Treating files and memory as the same thing.* Multics had this. Unix did not. Modern systems that approach this — memory-mapped databases, persistent memory APIs, the various "object store" architectures — are circling back to what Multics had at the start.

*Treating protection as a graduated rather than binary property.* Multics had this. Unix did not. The current state of the art in container isolation, capability-based security, and microkernel architectures is a partial recovery of what Multics's ring model already provided.

*Treating dynamic linking as the default rather than the exception.* Multics had this. Early Unix did not, and Unix's shared-library mechanism is still less clean than what Multics shipped with.

*Treating the file system's metadata as first-class.* Multics had ACLs, named principals, structured access controls. Unix had owner/group/world bits, and the industry has been bolting on better mechanisms for forty years.

*Treating the systems language as a high-level language.* Multics did this. Unix did it less aggressively, with C, and the simpler choice was instrumental in Unix's portability — but the principle Multics established was correct.

These are not points where Multics happened to be ahead of its time by accident. They are points where the Multics team made deliberate, considered choices that have, with retrospect, mostly been vindicated. The fact that Unix won despite making weaker choices on most of these axes is the fact this book is about. It is not that Multics was right and Unix was wrong; it is that being right was not enough.

## What Multics got wrong

It would be unfair to leave Multics on a clean note. The system was not perfect.

*It was complex enough to be hard to learn.* The Multics user manual was thick, the system's facilities were many, and the cost of becoming proficient was high. Unix's smaller surface was, for many users, a feature.

*It was tied to expensive hardware and a single vendor's commercial strategy.* When Honeywell exited the computer business in 1986 (the systems group was sold to Bull), Multics had no path forward. The system survived to 2000 because Bull kept supporting existing installations, but no new sites came on, and the system aged in place.

*It made some commitments — multi-level security, B2 evaluation, government-customer-focused design — that pulled its evolution toward use cases that limited its applicability elsewhere.* By the 1980s, Multics's customer base was dominated by military and intelligence agencies who valued its security model. The development priorities followed those customers, and the system's relevance to general-purpose computing eroded.

*It did not have a portable C-style language with a low compiler implementation cost.* This is the single biggest tactical failure relative to Unix. Without an easy retargeting path, the system could not follow the hardware curve as cheaper hardware became available.

## The retrospective

The Multics team built a system that, on a number of axes, made better choices than Unix did. The system also cost more, required more hardware, took more years to deliver, and depended on institutional patience that ran out. Unix, picking up the parts of Multics's design that could be made cheap and dropping the parts that could not, captured a market Multics's commitments excluded it from.

This is a recurring pattern. Multics took the maximal position on most of its design axes and committed institutional resources to implementing those positions. Unix took the minimal position on most of its design axes and committed only what it had to. In a world where hardware was scarce and engineering time was expensive, the minimal-position bet won. In a world where hardware became cheap and engineering time stayed expensive, the minimal positions calcified into permanent design choices that are now hard to revisit.

The thing to take from Multics is not that it was a misunderstood paradise. It was a real system with real shortcomings and a real institutional history that explains why it died. The thing to take is that *the design space Unix narrowed had been wider before Unix narrowed it*. Several of the choices Unix made were not technically necessary even in 1970; they were technically necessary *given Unix's resource constraints in 1970*. The constraints have not applied since the mid-1980s. The choices remain in place, because they are now ambient.

Multics is the case where the system that lost had, on the merits, an argument that — taken seriously — would have produced a different forty years. The next chapter looks at two systems that lost to Unix on substantially different time scales: Wirth's Oberon, a complete vision late in a career, and BeOS, a 1990s glimpse of what consumer computing might have been. Both are smaller in scope than Multics but, in their own ways, just as instructive about the design space Unix made hard to see.
