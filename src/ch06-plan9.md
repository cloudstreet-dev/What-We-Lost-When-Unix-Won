# The Bell Labs Second Swing

In the mid-1980s, the same group of people at Bell Labs who had built Unix decided to build another operating system. They had been using Unix for about fifteen years by then. They knew, more intimately than anyone else, what was wrong with it. They had also watched as the world adopted Unix on the back of cheap minicomputers, then cheap workstations, then cheap PCs, and as the design decisions they had made under tight resource constraints in 1970 calcified into an industry-wide architecture by 1985. They had the rare combination of capability, motivation, and institutional patience that would let them try again.

What they built was Plan 9 from Bell Labs. The name is a Bell Labs joke about Plan 9 from Outer Space, the Ed Wood film widely considered one of the worst movies ever made. The system was the opposite: a coherent, carefully designed second attempt at what Unix had been a first attempt at. The people who built it were Rob Pike, Dave Presotto, Ken Thompson, Phil Winterbottom, Howard Trickey, Tom Duff, Brian Kernighan, and a small group of others — names that anyone with passing exposure to Unix history will recognize. The fact that this team, with this much credibility, with this much institutional support, built Plan 9 and that Plan 9 then *failed to dent the market* is one of the most instructive facts in this book.

This chapter is about what Plan 9 was. The next chapter is about what happened after.

## The premise

Plan 9 started from the observation that Unix had won, that its early design choices were now embedded in everything, and that some of those choices had aged badly. In particular, the Plan 9 authors had three complaints about late-1980s Unix that motivated the new system.

The first complaint was that "everything is a file" had been a slogan more than a discipline. Unix made the file abstraction available for many things, but it also made many things *not* files. The network was not a file; you used sockets. Inter-process communication was not a file; you used pipes for some cases and System V IPC primitives for others. Process management was not a file; you used `ps` and `kill`, which interrogated the kernel through ad-hoc system calls. The window system was not a file; you talked to X via a binary protocol. The mouse and keyboard were not files in any directly useful sense. The actual practice of Unix had drifted away from the file abstraction in a hundred places, and the result was that *the user could not learn one composition mechanism and apply it uniformly to everything*.

The second complaint was that Unix had no good story for distributed computing. By 1985, Sun's Network File System and a few competing protocols had made it possible to share files across machines, but the rest of the system — processes, devices, network connections, system services — was machine-local. If you wanted to run a job that used the compute power of one machine, the disk of another, and the printer of a third, you assembled the pieces by hand, with different tools for each, and the abstraction the user lived in was always "I am on this machine, talking to that machine." There was no system-level construct that let you act on a federation of machines as a single resource.

The third complaint was that Unix's authentication and security model was a 1970s artifact that did not scale. The user was identified by a numeric UID on a single machine; cross-machine identity required separately-maintained mappings; the model assumed a small set of users on a single host, all of whom trusted the operator of the host. Networks of workstations broke this model and the patches applied to fix it — NIS, Kerberos, eventually LDAP — were complicated and full of failure modes that Unix users had had to absorb.

Plan 9's authors set out to fix all three.

## 9P: the unifying protocol

The architectural heart of Plan 9 is a network protocol called 9P. Everything in Plan 9 — every file, every device, every service, every running process — is accessed by sending 9P messages over a connection. 9P is, structurally, a remote file system protocol, but the trick is that *everything in Plan 9 is exposed as a file system*, and 9P is therefore the only protocol you ever need.

This is the slogan "everything is a file" taken literally and pushed all the way through the system, with no exceptions for inconvenient cases. Some examples of what this looks like in practice:

The network stack is a file system. To make a TCP connection, you do not call `socket()`. You open a file in `/net/tcp/clone`, read it to get a connection number, write the destination address to the appropriate connection's control file, and then read and write the data file. To listen for incoming connections, you open a file in the appropriate directory and read it; the read blocks until a connection arrives. The user is doing the same kinds of operations — open, read, write — that they would do on any file. Network programming becomes file programming.

The window system is a file system. Each window appears as a directory containing files representing the window's state: a file you write to to display text, a file you read from to get mouse events, a file you read from to get keyboard input. Programs that want to use the window system open and read and write those files. There is no protocol to learn beyond the file operations.

The process table is a file system. Each running process appears as a directory under `/proc`, containing files for its memory, registers, control, status, and so on. To send a signal to a process, you write to the appropriate control file. To examine its memory, you read from the memory file. To stop it, you write to the control file. The `ps` command becomes a directory listing; `kill` becomes a file write.

The system clock, the audio device, the disk, the graphics card — all of these are exposed as file systems. Each driver presents a directory hierarchy with the operations the driver supports. The user composes them with the same shell tools they use to manipulate text files.

This is more than a uniformity gimmick. The consequence is that the system has exactly one access protocol. Any tool that can read and write files can interact with any system service. Any service can be implemented by any program that can serve a file system. Any service can be made available remotely with no additional protocol work: you mount the remote machine's file system at a path on the local machine, and the services become available at that path as if they were local. The composition is the same.

## Per-process namespaces

The second technical idea that distinguishes Plan 9 is the per-process namespace. In Unix, the file system layout is a global property of the machine. Every process sees the same root, the same `/etc`, the same `/usr/local/bin`. Plan 9 abandoned this. Each process — or, more precisely, each group of cooperating processes — has its own file system namespace. The same path means different things in different processes' contexts.

This matters because of the previous point. Since everything in Plan 9 is a file, the namespace controls everything the process can see. Two processes with different namespaces are, effectively, running on different machines, in terms of which devices, which services, which directories they can reach. A process can mount another machine's file system into its namespace and gain access to that machine's services. A process can hide parts of the local file system from its children by *not* including them in their namespace.

The mechanism Plan 9 used to compose these namespaces was the union mount. A directory in a namespace could be the union of several other directories, with a stacking order, so that lookups walked the stack until something was found. The user could, with a single command, layer a new directory on top of an existing one — say, layering their own `bin` on top of the system's `bin` — without modifying any file or environment variable, simply by changing the namespace.

The practical effect was that the configuration model that Unix had grown up around — environment variables, dotfiles, search paths — could be replaced by namespace composition. Want a different compiler in your shell? Mount it over the standard `bin`. Want to test a service without affecting other users? Run it in a private namespace where the standard service is hidden. Want to provide a sandboxed environment for a guest user? Compose a namespace with only the services they should see. The mechanism is *the* mechanism, not a special-case sandbox layer bolted on top.

This is one of the ideas that has, slowly, leaked into the Unix world over the last twenty years. Linux's mount namespaces, used by container runtimes like Docker, are a partial reinvention of the Plan 9 idea. The Linux version is implemented as a set of features on top of the conventional Unix model, with backward compatibility to single-namespace assumptions; the Plan 9 version was the model from the start. The fact that containers became Linux's most consequential 2010s innovation is not unrelated to the fact that Plan 9 had been doing this for twenty-five years and the kernel developers, eventually, ported the idea.

## CPU servers, file servers, terminals

Plan 9's distribution model assumed three roles for machines, each specialized:

*Terminals* were the machines users sat at. A terminal had a screen, keyboard, mouse, and modest CPU. Its job was to render the user's interface and provide a local environment. Terminals were not where serious computation happened.

*CPU servers* were machines with substantial compute resources but no users physically attached. CPU servers ran the heavy computation that the terminals delegated to them. A user at a terminal would, when they wanted to compile a large program, send the compilation to a CPU server and watch the output appear on their terminal as if it were local.

*File servers* were dedicated machines that stored data. The file servers exposed their storage as 9P file systems to the network. Terminals and CPU servers mounted the file server's file systems into their namespaces and interacted with the data as files.

The key trick was that, because of the per-process namespace and the uniform 9P protocol, the distinction between local and remote was almost invisible to the user. A user at a Plan 9 terminal had, in their namespace, files from the file server, processes running on the CPU server, devices from the terminal itself, and services from any other machine they had mounted. They interacted with all of these as files in a single namespace. The system did not have to remind them which physical machine each file lived on, because the system did not care, and neither did most programs.

This is, again, a model that Unix has since approached from many directions. Distributed file systems (NFS, AFS, CIFS), remote process execution (rsh, ssh), distributed object systems (CORBA, gRPC), and most recently cloud-native architectures with their service meshes and remote-state assumptions, all reach toward what Plan 9 had. None has reached it as cleanly, because Unix's model — local namespace, separate services, distinct protocols — gets in the way. Plan 9's architecture put the distribution at the bottom of the design rather than the top.

## The user model

Plan 9 also rethought authentication and the user model. The system did not assume that users had accounts on individual machines. Instead, an authentication server (the *factotum* later, but originally a related component) held the user's keys and answered authentication challenges on the user's behalf when the user's processes needed to access remote services. The user's identity was an attribute of a session, not of a machine. The same user could log in from any terminal and have the same namespace, the same access, the same environment, because the environment was constructed from services and file servers rather than from local machine state.

This part of Plan 9 was, by the standards of late-1980s Unix, alien. Unix users were used to having an account on each machine, with separate home directories, separate dotfiles, separate everything. Plan 9 made this seem antiquated. The price was that you had to commit to running a Plan 9 cluster — terminals, CPU servers, file servers — to get the benefit. You could not migrate piecemeal.

## What was good about it

There is a strange genre of writing about Plan 9, mostly from people who used it for a few months and never went back to a Unix workstation feeling the same way about Unix. The common observation in this writing is not that Plan 9 was fast or pretty or featureful — it was, by most accounts, fewer of those things than contemporary Unix — but that Plan 9 was *coherent*. The same patterns worked everywhere. You learned one set of file operations and you could administer the network, manage processes, work with devices, and program the window system using them. The friction of the system was uniformly low, because there were not five different mechanisms to learn, each with their own quirks.

This is the thing Plan 9 had that Unix lacked. It is also the thing that is hardest to convey to someone who has not used the system. A demo of Plan 9 looks unimpressive: a window manager that is austere by 1990s standards, an editor (`acme`) that does not look like anything else, a shell that has clean syntax but feels like a Unix shell to a Unix user. The advantage of the system is not in any individual feature; it is in the smoothness with which the features combine. Plan 9 paid a one-time cost for radical uniformity and received, in return, a system where most non-obvious things worked smoothly because they had been designed by people who took uniformity seriously.

Several specific tools deserve mention. The Plan 9 C compiler suite, written from scratch by Ken Thompson and others, was small, fast, and supported a clean cross-compilation model that Unix's mainstream C compilers spent the next twenty years failing to match. The `acme` editor, by Rob Pike, was a mouse-driven text environment that integrated with the file system and the shell in a way no Unix editor had — you could type a command in the middle of a document and middle-click it to execute, with the output going wherever you placed the cursor. The Plan 9 shell, `rc`, was a clean redesign of the Bourne shell with fewer quoting bugs and cleaner control flow. The `sam` editor, also by Pike, had a structural-regex-based command language that prefigured later editors' multiple-cursor and search-and-edit features by twenty years. The Plan 9 system had, in short, a set of tools that were small, well-designed, and built by people who understood Unix's tools and were trying to do them again with the lessons applied.

## Why it did not win

Plan 9 was made publicly available in 1992 and again, in successive releases, through the 1990s and 2000s. It had institutional support from Bell Labs through the early 2000s. It was free for non-commercial use. It was, by most accounts, technically superior to contemporary Unix workstations for the kind of work its authors did.

It did not win. By 2026, the install base of Plan 9 — across the original Bell Labs distribution, the 9front fork, and various other derivatives — is in the low thousands of active users. The system has a reputation as a beautiful curiosity. Working programmers who have used Plan 9 are an exotic minority within the population of programmers who have heard of it.

The reasons are not subtle, and they are mostly not Plan 9's fault.

*Plan 9 arrived after Unix had won.* By 1992, the BSD and System V lineages had standardized enough of the Unix world that an entire industry was being built on Unix assumptions. Sun, HP, DEC, SGI, and IBM were all shipping Unix workstations with overlapping but incompatible variants. The pain of moving from one Unix to another was already non-trivial. Moving from Unix to *not Unix* was a different category of decision. The relevant decision-makers — IT departments, university administrators, startup founders — were going to choose between Unix variants, not between Unix and Plan 9.

*Plan 9 was incompatible with everything.* Plan 9 was not a Unix variant. Plan 9 binaries did not run on Unix. Unix binaries did not run on Plan 9 (there was a Unix emulation environment, but it was not a first-class part of the system). The X window system did not exist on Plan 9, by design — Plan 9 had its own window system. The Plan 9 C dialect (Alef and later Limbo, plus a simplified C without the preprocessor's macro system) was not the C the rest of the world was writing. Even the file system layout was unlike Unix's. Plan 9 was a different operating system, in the strong sense, and adopting it meant porting or rewriting everything.

*The hardware story did not help.* Plan 9 ran on Bell Labs's custom terminals (the Gnot, the Magnum), then on commodity hardware, but it never had the close-knit relationship to a specific hardware platform that Unix had to the DEC and SPARC ranges. Without a flagship hardware partner, Plan 9 stayed in the research-system category for its commercial-decision-maker audience.

*The web showed up.* By 1995, the World Wide Web had become the dominant story about distributed computing. The Plan 9 architectural insight — that you could federate machines through a uniform protocol — was already being expressed, at a much higher level of abstraction and with much weaker guarantees, by HTTP and URLs. The web was Plan-9-ish enough that it absorbed much of the conceptual oxygen Plan 9 was breathing. We will return to this in Chapter 10.

*Bell Labs changed.* The institutional support that had carried Plan 9 from the mid-1980s through the 1990s was a function of a particular Bell Labs management environment — the one that had also carried Unix's first ten years. That environment did not survive the breakup of AT&T, the formation of Lucent, the dot-com bust, and the subsequent reorganizations. By the early 2000s, the Plan 9 authors had mostly moved on. Rob Pike and Ken Thompson eventually ended up at Google, where they used what they had learned in Plan 9 to design the Go programming language. Plan 9 itself was open-sourced and left to its community.

## What survives in 2026

The ideas in Plan 9 have survived in three forms.

*As features in Linux.* Mount namespaces, cgroups, and the container model are partial reinventions of Plan 9 ideas. The container ecosystem in 2026 — Docker, Kubernetes, the various runtime stacks — is doing things Plan 9 was doing in 1990, with substantially worse coherence but vastly more market success.

*As Go.* Pike and Thompson's later language carries forward several Plan 9 sensibilities: small surface area, plain text, channel-based concurrency (which descends from the Newsqueak language Pike wrote in the 1980s and which Plan 9's threading model borrowed from), and the assumption that programs should compose. Go is the closest thing to a Plan 9 sensibility that has entered the mainstream.

*As 9front.* The Plan 9 community — particularly the 9front fork, which we will cover in the next chapter — is small but active. People run Plan 9 on commodity hardware in 2026. The system is still being improved. It will probably never be more than a research and enthusiast platform, but it is not dead, and the operating-systems researchers who care about the ideas in it still have a working system to point to.

The next chapter looks at Inferno, Limbo, the 9 lineage's afterlife in 9front, and what happens to a system designed by people whose institutional environment changes. Plan 9 is not the last word on the operating systems argument. It is, however, the strongest argument from inside the Unix tradition that Unix as it shipped was not the only possible Unix, and that the alternatives the same group of people could imagine — given a second chance — looked very different from what we ended up with.
