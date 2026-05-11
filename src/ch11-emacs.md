# Emacs as the Last Lisp Machine in Costume

Emacs is the most successful piece of software ever shipped from the Lisp Machine tradition. It has been in continuous use for forty-eight years as of 2026. It has tens of millions of users by any reasonable counting. It runs on every operating system anyone cares about. It is, in the dimensions that matter for software longevity, more successful than Genera was, more successful than Smalltalk has been, more successful than Plan 9 ever became. It is also, in 2026, by any reasonable accounting, the largest and most widely used Lisp environment in active development anywhere in the world.

This is a strange fact, because Emacs is not generally described as a Lisp environment. Emacs is described as a text editor. The Wikipedia article calls it a text editor. The Linux distributions package it as a text editor. Most of the people who use it think of it as a text editor. The fact that it is structurally a Lisp Machine in disguise is a detail that the system has been quietly carrying for almost half a century, mostly successfully hiding from the world that, were the world to notice, might object to the underlying architecture.

This chapter is about what Emacs is, what it preserves from the Lisp Machine tradition, and what it has had to dilute to survive in the world that adopted it. The answer is mostly "everything important survives" and "the dilutions are real but not load-bearing." Emacs is the cleanest counter-example in this book to the pattern of the alternatives losing. It also did not win the way Unix won, but it won enough to keep going, and the way it won is informative.

## What Emacs is, structurally

The naïve description of Emacs is: a text editor with a programming language for customization. The accurate description is: a Lisp environment that includes, as one of its responsibilities, a text editor.

When you start Emacs, what runs is a Lisp interpreter (more accurately, a bytecode VM with a JIT in recent versions) loaded with a large quantity of Lisp code. The Lisp code implements the buffer model, the display, the command interface, the keymaps, the syntax-highlighting machinery, the indentation rules for various languages, the file management apparatus, the package management, the documentation system, and tens of thousands of optional features that users have layered on top.

The C core of Emacs — the part that is not Lisp — provides the things that Lisp cannot easily do for itself: the bytecode interpreter, the display rendering, the underlying I/O, the memory management. The C core has, by 2026, grown to around several hundred thousand lines, mostly stable. The Lisp on top is many millions of lines if you count the standard library and the user-contributed packages indexed by ELPA and MELPA. The C is the substrate; the Lisp is the system.

This division mirrors the Lisp Machine model almost exactly. The Lisp Machine had microcode at the bottom, supporting tagged-data operations and garbage collection in hardware; above the microcode was Lisp, all the way up. Emacs has C at the bottom, supporting bytecode execution and display rendering; above the C is Lisp, all the way up. Both systems present, to the user, a Lisp environment whose features are implemented in Lisp and whose customization is done in Lisp at runtime.

The user, in Emacs, can do everything a Lisp Machine user could do in spirit if not in detail. Open a buffer, evaluate Lisp expressions in it, see the results, modify functions while the system is running, inspect the state of any variable, attach a debugger to any failing computation. Pull up the source of any built-in command and read it inline. Replace any keybinding with a new function written on the spot. Save the modified configuration as part of your init file so that the modifications persist across sessions. Build entire applications inside the editor — mail clients, IRC clients, news readers, web browsers, project management systems, version control front-ends, code editors for languages the original Emacs authors never imagined — by writing Lisp.

This is what Genera users did. The fact that Emacs users do it without describing it that way is one of the more impressive feats of marketing-by-omission in computing history.

## What Emacs preserved

The specific Lisp Machine properties that Emacs preserved are worth enumerating, because they are the ones that have continued to do real work for the system's users.

*Image-like persistence, in a limited form.* Emacs does not save its entire heap to disk on exit, the way Genera did. But it does support persistent buffers and project state, it does load all of its Lisp libraries at startup into a single image-like environment, and the user can dump a partly-customized Emacs to disk as a new executable (using the "dump" feature) that starts up with that customization pre-loaded. The full image model is not there, but the practical effect of having a unified runtime environment is.

*Live introspection of system code.* Every built-in Emacs command is, to the user, a button that says "describe this function" away from being a piece of readable source code. `C-h f` (function help) on any command shows you what it does, what arguments it takes, and where the source lives, with a clickable link to the definition. `C-h v` does the same for variables. The system's source is the system's documentation, exactly as it was on the Lisp Machines.

*Live modification.* You can redefine a function inside Emacs and the new definition is live immediately. The next time the function is called — whether by a user keypress, by a hook, by another function — the new definition runs. There is no restart. The Lisp Machine practice of patching a running system applies to Emacs without modification.

*A condition system.* Emacs Lisp's `condition-case` and `signal` are direct descendants of the Lisp Machine condition system. Errors in Lisp code do not crash the editor; they open a debugger from which the user can examine the state and recover. This was inherited rather than reinvented, and it is part of why Emacs is so resilient to user customizations gone wrong.

*Customization through the substrate.* The Lisp Machine practice — that you customize the system by writing Lisp code that becomes part of the system — is the default Emacs practice. The init file (originally `.emacs`, now usually `init.el` or `early-init.el`) is a Lisp program. Configuration is programming. There is no separate configuration language, no domain-specific config syntax, no rules layer that is distinct from the runtime layer. It is all Lisp, all the way down.

*Documentation as queryable structured data.* Emacs's documentation system links commands to their bindings, bindings to their commands, variables to their values, faces to their definitions. The user navigates documentation by interrogating the system rather than by reading external files. The Info manuals, while text-based, are integrated with the editor's navigation primitives and live inside the same environment as the running code.

These are not minor preservations. They constitute the substantial part of what made the Lisp Machine experience distinctive, applied within a single editor application. The fact that they apply within an application rather than across an operating system is the dilution.

## What Emacs diluted

The dilutions are real. Emacs lives inside a Unix process. It interacts with the rest of the operating system through the Unix process model. The buffer is not a file; it is an in-memory editing surface that can be loaded from or saved to a file. The Emacs Lisp environment cannot reach into other processes' memory, cannot redefine kernel facilities, cannot present its objects to the network as files (in the Plan 9 sense). The integration that the Lisp Machines had at the operating-system level, Emacs has only inside its own process.

This boundary has consequences. Some specific ones:

*Emacs cannot make other applications introspectable.* Inside Emacs, every function is browseable. Outside Emacs, you are back in the Unix process model, with whatever introspection apparatus the host OS and the running programs choose to expose. An Emacs user editing a JavaScript file can browse the Emacs Lisp implementation of the JavaScript mode but not the JavaScript engine's internals.

*Emacs cannot persist arbitrary running state.* Emacs can save buffers, projects, and configurations across sessions. It cannot save the full running state of the Lisp environment the way Genera could save a band. Restarting Emacs reloads the Lisp libraries and reconstructs the environment; the user's interactive state — the history, the open buffers, the position in each — is preserved through specific mechanisms (desktop.el, savehist, recentf), not through image-level persistence.

*Emacs is a guest in the OS.* It does not own the display the way a Lisp Machine did. It cannot drive devices directly. It cannot speak to the network without going through OS-level APIs. The Lisp environment is sovereign within itself; outside its boundaries, it is just another Unix process competing for the operating system's attention.

These dilutions are the price Emacs paid for being a Unix application instead of a Unix replacement. The price was, in retrospect, worth paying. Emacs got to live in the Unix world rather than fighting it, and as a result Emacs got to live, while the Lisp Machines did not.

## Why Emacs survived

The Emacs survival story is, in some ways, the most encouraging piece of evidence in this book that the Lisp Machine ideas were not stranded by their hardware. They could be carried in a Unix process, they could be made portable, they could be made distributable as free software, and they could acquire and keep a user base across decades. The proof is Emacs, sitting on every developer's machine that has a Unix-like OS underneath.

The specific reasons Emacs made the transition that Genera could not are worth being honest about.

*Emacs ran on commodity hardware.* From its earliest portable versions, Emacs targeted whatever Unix workstation the user had. There was no Emacs hardware. The system did not require custom silicon, did not require an unusual memory configuration, did not need specialized graphics. It ran in a Unix process on whatever Unix the user had bought a workstation for. The cost of adopting Emacs was downloading and compiling it.

*Emacs was free.* Richard Stallman's choice, made in the early 1980s, to release Emacs under the GPL (then in its evolving forms) meant that Emacs was available to anyone with a Unix machine for the cost of typing some commands. There was no licensing apparatus, no proprietary restriction. Universities adopted Emacs because there was nothing to negotiate. Developers adopted Emacs because they could read the source. This was the same property AT&T's Unix licensing had given Unix; Stallman's choice gave Emacs the same advantage.

*Emacs hid its Lisp nature.* The Emacs presentation to new users was as an editor. The fact that you could program it in Lisp was an advanced feature, not a barrier to entry. A user could be productive in Emacs for years before they wrote a single line of Lisp, using the system as a powerful editor with built-in commands. This avoided the cost-of-entry problem that had killed the Lisp Machines, where you had to commit to the Lisp environment before you could do anything. Emacs let the user start with the editor and discover the environment later.

*Emacs was useful at every scale.* From a simple text editor to a complete email-news-IRC-development environment, Emacs scales smoothly with how much the user wants to invest. Most users use ten percent of the system and never look at the other ninety. A few users use most of it. Both groups are well-served. The Lisp Machines had a higher minimum buy-in; Emacs has effectively none.

*Emacs was carried by a community.* The GNU Emacs project has had continuous active development since 1985. The XEmacs fork in the 1990s and 2000s provided a competing line that eventually rejoined into modern GNU Emacs's feature set. The community has produced massive package ecosystems (ELPA, MELPA), distributions (Spacemacs, Doom Emacs, Prelude), and configurations. The Lisp Machines never had this kind of broad community; their user base was small and the people maintaining them were a handful at any given time. Emacs's community, by contrast, has been continuously several orders of magnitude larger.

## The cost of carrying the tradition this way

Carrying the Lisp Machine ideas inside a Unix process has had costs that are worth being clear about.

*Performance.* Emacs Lisp, by long-standing convention, has been an interpreted language, then a bytecode-interpreted language, with a JIT in recent years (native-comp, which became default in Emacs 28). It has historically been slower than the compiled Lisps that ran on Lisp Machines. The native-comp work has closed much of the gap, but Emacs still pays a cost for being a Lisp environment hosted inside a Unix process running on commodity hardware. Users notice this in startup time, in syntax highlighting on large files, in the responsiveness of editor commands that do a lot of computation.

*The buffer model is not the heap model.* Emacs's primary data structure is the buffer — a sequence of characters with attached properties. This is a fine data structure for editing text. It is not the same as the Lisp Machine model where the data structure is the heap and the editor is one of the things that displays parts of the heap. The buffer model means that Emacs is, structurally, an editor; the Lisp environment serves the editor rather than being the primary surface. A Genera user who wanted to manipulate a database table did not put the table in a buffer; they manipulated the table directly through inspectors that understood tables. An Emacs user who wants to manipulate a database table edits text that represents the table or builds custom modes for tabular data, with the buffer underneath.

*The display model is text-first.* Emacs has acquired some graphical capability over the decades — embedded images, more sophisticated overlays, the recent work on tree-sitter integration and richer display — but the underlying display model is a grid of characters with attached properties. The Lisp Machines had a full bitmap graphics model from the start, with the Dynamic Window facility on Genera providing fully graphical, mouse-interactive, object-oriented displays. Emacs's display is, by comparison, a constrained surface. The constraint is part of why Emacs is portable — character grids work on text terminals as well as graphical ones — but it is a real limit on what Emacs can express.

*The user has to do the integration.* Like with the web, Emacs users find themselves doing the cross-application integration that Genera would have done at the system level. An Emacs user wanting to use email, calendar, project management, source code editing, and chat in an integrated way ends up wiring together separate Emacs packages, each with its own conventions, and dealing with the seams. The integration is better than it would be across separate native applications, but it is worse than it would have been in a single image with uniform object access.

## Why this chapter matters

The reason to spend a chapter on Emacs in a book about operating systems is that Emacs is the strongest available evidence that the alternatives this book has covered are not impossibly far away. The Lisp Machine model, in particular, has a working implementation right now, on the machine of almost every reader who is going to read this book, with a user base of millions. The ideas are not gone. They are sitting in /usr/bin/emacs.

This matters for the closing argument of the book. The "what's recoverable" question of Chapter 13 has, in Emacs, an existence proof on the optimistic side. Some of what we lost when Unix won is recoverable, because it has been quietly continuing inside a Unix tool the whole time. An Emacs user, while editing a configuration file, is *living inside* a partial Lisp Machine. The fact that they may not realize it does not make it less true.

The Emacs story also tells us something about the transmission mechanism. Ideas survive when they can be carried in a form that does not require everyone to switch operating systems. Emacs survived because it did not ask Unix users to leave Unix. It asked them to install one more program. The cost was small enough that the program got installed, and the cost stayed small enough that the program kept being used. The Lisp Machine ideas, packaged this way, traveled. Packaged as a complete operating system requiring custom hardware, they did not.

This is a generalizable lesson. The systems in this book that died at the OS level survive better when they are repackaged as guests inside the dominant OS. Smalltalk survives in Pharo, hosted by Linux or macOS. Lisp Machine ideas survive in Emacs and in SLIME-style tooling. Plan 9 ideas survive as containers in Linux. The host environment, having won, becomes the carrier of the alternative ideas. The carrier is a compromise, but it is a working compromise, and the alternative — extinction — is worse.

The next chapter looks at where Smalltalk's ideas are living in 2026, including the modern Pharo and Glamorous Toolkit projects covered in Chapter 5 plus the broader pattern of Smalltalk concepts surfacing in other languages and tools. Then the book turns to honest reckoning: what is recoverable, what is permanently gone, and how a reader in 2026 should think about both.
