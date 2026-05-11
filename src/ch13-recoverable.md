# What's Recoverable, What's Permanently Gone

The previous twelve chapters have surveyed alternatives to Unix that lost. This chapter takes inventory. Some of what was lost is recoverable, in the sense that a person or team in 2026 can adopt the ideas and use them productively. Some of what was lost is not coming back — not because Unix won, but because the world in which the alternatives could be built has changed in ways that make rebuilding them either impossible or pointless. The chapter's job is to distinguish these two categories honestly.

A few ground rules for the inventory.

First, recoverable does not mean *mainstream-adoptable*. Some things are recoverable for an individual or a small team and not recoverable for the industry. The two are different categories and the chapter will keep them apart.

Second, permanently gone does not mean *unmissed* or *unimportant*. Some of the most valuable things this book has covered are in the permanently-gone category. Their permanence does not diminish their value; if anything, it heightens the cost of the loss.

Third, the dividing line between the categories runs through specific facts about the contemporary world — networking, security, hardware diversity, the threat model — not through facts about Unix specifically. The book has been careful, throughout, to avoid blaming Unix for everything. The same care applies here. Some losses are downstream of Unix's win; others are downstream of unrelated changes in the world; many are both.

## Recoverable: image-based development for individuals

The single most clearly recoverable property covered in this book is image-based development as a personal practice. The Smalltalk and Lisp Machine traditions both had this property at their core. It survives, in usable form, in 2026 in Pharo, Squeak, Glamorous Toolkit, SBCL with SLIME or Sly, Clojure with its REPL-based workflow, and Emacs with various extensions that approximate it. The cost of adopting image-based development for an individual is the cost of installing one of these environments and committing to using it for a class of work that suits it.

The class of work is real but specific. Image-based development pays off when you are iterating heavily on a complex system you will use for years, when fast feedback is valuable, when persistent state across sessions saves work, and when you do not need to ship a static artifact to many users. Research, custom internal tooling, long-running exploratory analysis, simulations, scientific computing, certain kinds of personal productivity tooling — all of these are well-suited.

The class of work where image-based development pays off poorly is also real: shipping a static artifact to many users, building software that has to run on a server you do not control, working on a team where everyone needs to be productive in the same environment, working on a system whose code base must be visible to standard text-based tools (linters, search, automated review). In these cases, the file-based model has fewer seams.

What is recoverable here is the choice. The 2026 reader does not have to choose between image-based and file-based development as ideologies. Both are available. The choice should be made per project, based on the work's actual shape. The recovery is in the *availability of the option*. The dominant industry choice does not foreclose the option; it just means you have to ask for it.

## Recoverable: live code reloading and condition-style error handling

The Lisp Machine practice of replacing a running function and having the system pick up the new definition immediately is, in 2026, broadly available. Erlang, Elixir, and Clojure ship this property as a first-class part of their development workflow. Common Lisp ships it directly. Smalltalk has it. Many web development frameworks (Phoenix, Rails in dev mode, the Vite ecosystem with HMR) provide weaker versions of it. Python's notebook environments have it. Even Emacs Lisp's `eval-defun` has it.

The condition system — structured error handling with restarts that the calling code can register — is rarer but recoverable. Common Lisp has it. Clojure has it. Smalltalk has it. The Ruby and Python `rescue`/`except` mechanisms are weaker forms; they recover but they do not let the recovery point be chosen by the caller from a menu of options that the called code registered. The full condition-system experience is available if you use one of the languages that has it.

What this means for a 2026 developer is that these properties are choices you can make at the language and framework level. They are no longer experimental research-system features. They are mature, productionized capabilities that any team can adopt. The recovery is in the available options, not in their universal application.

## Recoverable: structural composition of services

Plan 9's per-process namespaces and 9P-style uniform protocols are, in compromised form, recoverable in 2026 through the container ecosystem. Linux mount namespaces, cgroups, network namespaces, and the various userland tools built on them (Docker, Podman, Kubernetes, containerd) provide a worse-engineered version of what Plan 9 had natively. The composition that Plan 9 achieved by mounting file systems is achieved, in 2026 Linux, by composing container images, volumes, and network configurations.

The recovery is partial. The Plan 9 model was coherent; the Linux container model is a federation of separately-evolved subsystems with overlapping responsibilities. The composition that worked smoothly in Plan 9 because everything was a file in a uniform namespace works less smoothly in Linux because the underlying primitives are heterogeneous (mount namespaces handle some things, cgroups others, network namespaces others, the various capability bits others still). The user pays an integration tax.

What is recoverable is the structural pattern. A team that wants to build a system out of small, composable services with clean isolation can do so. The tools exist. They are less elegant than what Plan 9 offered. They are vastly more popular and vastly better documented. The trade is a real one and most teams take it; the result is the contemporary cloud-native architecture, which is in important ways a recovery of Plan 9 sensibilities in a Linux environment.

## Recoverable: integrated documentation and reflective tooling

The Lisp Machine and Smalltalk practice of having the documentation be queryable structured data, integrated with the running system's code, has been recovered in modern IDE tooling. Language servers (the LSP protocol), tree-sitter parsing, type-system-aware tooling, integrated documentation lookup in editors like VS Code, IntelliJ, and Emacs — all of these are partial recoveries of what the Lisp Machines had natively. The recovery is unevenly distributed (statically-typed languages get more of it than dynamic ones, except for the languages with mature dynamic-introspection tools), but it is broadly available.

What is recoverable for a developer in 2026: a tooling experience in which navigating from a function call to its definition, from a variable to its type, from a type to its documentation, is instant and integrated. This is the part of the Lisp Machine experience that the industry has, by long effort, brought into the mainstream. It is not perfect — the integration is often partial, with several tools collaborating across protocols rather than living in a single environment — but it is much better than it was twenty years ago.

The notebook environments and the moldable-tooling research are pushing this further. A user of Glamorous Toolkit or a sophisticated Jupyter setup can have a working environment in which the data they are looking at carries its own viewer code, where the inspector knows what kind of object it is showing and presents an appropriate UI. This is the part of the Smalltalk and Lisp Machine experience that is *most* recoverable, because the tooling does not require operating-system-level changes; it requires editor and IDE work, which has been steady.

## Partially recoverable: a single integrated environment

Where recovery becomes harder is at the level of *full integration*. The Lisp Machines and Smalltalk environments offered a single coherent environment in which the language, the runtime, the editor, the debugger, the documentation, the libraries, the UI, and the user's data lived in one image with uniform access. A 2026 developer can recover most of the pieces but cannot recover the integration in the dominant computing context.

The reasons are largely the same as the reasons covered in the OS chapters: the modern computing stack assumes a host operating system providing process isolation, a user shell providing the file-and-process model, a language runtime that runs as a guest in the host OS, and applications that exist as separate processes. The integration that the Lisp Machines and Smalltalks had at the system level cannot be recovered without rebuilding the system level. Pharo recovers it inside its image and pays the cost of being a guest. Emacs recovers it inside its process and pays the cost of being just an editor. Neither is the full thing.

What is partially recoverable: a developer can commit to working inside Pharo, Emacs, or a similar guest environment for a significant fraction of their work, and gain a meaningful portion of the integration benefit. The cost is that they live in the guest environment, with whatever it does and does not provide, and the host environment is the place they go for things the guest does not handle. This is a viable workflow for some kinds of work. It is not a viable workflow for everything.

## Partially recoverable: composable graphical computing

Morphic-style direct manipulation of every visible thing, the property that made Squeak and Glamorous Toolkit feel different from conventional GUI toolkits, is partially recoverable. The recovery channels are: working inside Pharo or another Morphic-using environment for whatever fraction of one's work suits; using research tools like Glamorous Toolkit that push direct manipulation further than mainstream GUI frameworks; or — in a different but related direction — using design tools (Figma, the various contemporary creative software) that expose strong direct-manipulation models for specific domains.

What is not recoverable, in the mainstream, is the *uniform* direct-manipulability of every visible element across all applications. The mainstream UI stacks (Cocoa, Win32, Qt, the web platform, the React ecosystem) all assume that direct manipulation is something each application implements for the affordances it chooses to expose, not something the substrate provides by default. The result is that any given application may be quite manipulable internally and entirely opaque from outside. The PARC vision of a unified direct-manipulation substrate has not transferred and probably will not transfer to the mainstream.

The partial recovery is real for users willing to spend significant fractions of their time inside Pharo or GT. It is not available outside those environments at a comparable depth.

## Permanently gone: hardware-supported runtime semantics

The Lisp Machine's hardware support for tagged data, hardware-assisted garbage collection, and microcoded high-level instructions is gone and is not coming back. Modern processors are general-purpose RISC or RISC-derived architectures, with extensions for specific workloads (vector operations, machine-learning acceleration, cryptography) but not for runtime semantics of high-level languages. The market does not support custom processors for niche language workloads. JIT compilation has filled most of the performance gap that hardware support used to bridge.

This loss is real but mostly does not matter in 2026. Modern JIT-compiled dynamic-language runtimes are fast enough that the performance argument for tagged hardware is weak. The Lisp Machine performance advantage was real on 1985 hardware; it would be marginal on 2026 hardware. The recovery is in software techniques (HotSpot, V8, the various other JITs), and the software techniques are good enough that the hardware loss is mostly invisible.

## Permanently gone: the trust assumptions that allowed open systems

The ITS culture — open systems, no passwords, no user-to-user protection, mutual trust enforced by social means — is gone and is not coming back. The reasons have nothing to do with Unix winning. They have to do with networks becoming hostile environments, with the rise of organized cybercrime, with the use of computers for purposes (financial transactions, medical records, election infrastructure, military systems) where openness is structurally unsafe.

The Lisp Machines and the early Unix systems both assumed a trusted user base. Unix's security model was a 1970s artifact, but Unix was at least built for a small set of users with separate accounts; ITS was built for a community of trusted peers. Both models are inappropriate to the contemporary computing environment, where the threat model includes adversarial users, network attackers, malicious software, and state-level actors targeting infrastructure.

What this means concretely: a 2026 system designed along the lines of ITS would be unusable within minutes of being put on the public network. Even private deployments could not safely assume the trust model. The systems we have, including Unix, have evolved security mechanisms — capability-based isolation, mandatory access control, sandboxing, the various Linux security modules — that consume substantial design space in every modern OS. That design space is permanently committed. The Lisp Machine practice of letting users modify any part of the running system from inside it cannot be safely reproduced at the system level, only inside per-user sandboxes where the consequences of modification are contained.

This is the deepest permanent loss in this book. The Lisp Machine experience required trust. The world we have does not support it.

## Permanently gone: single-vendor hardware-software integration

The Lisp Machines, the Alto, the Multics hardware-software co-design, BeOS's relationship to the BeBox — all of these benefited from a tight relationship between hardware design and operating system design. The hardware could be tailored to what the OS needed; the OS could exploit hardware features that commodity systems did not have.

This integration is gone in the personal-computer space. It survives in narrow niches: Apple's hardware-OS integration in the M-series Macs is the closest contemporary example, and Apple gets real performance and efficiency benefits from it. But Apple is not building a Lisp Machine. The integration is in service of a Unix-derived OS, not in service of an alternative. The economics that supported the Lisp Machines' custom hardware do not exist in 2026 except for very large vertically-integrated players (Apple, Google with its Pixel hardware, the cloud providers with their custom silicon for specific workloads).

What this means: a new operating-system idea cannot count on its own hardware in 2026 unless its sponsor is a hyperscaler or a near-monopoly platform vendor. The hardware is going to be commodity x86 or ARM. The OS has to run on what is there. This is a meaningful constraint on the design space; some of the things the Lisp Machines did cannot be done as well on commodity hardware.

## Permanently gone: the patient institutional sponsor

Multics took years to ship and decades to refine. Smalltalk had Xerox PARC's open-ended research budget for ten years. Genera had Symbolics as a single-product company supporting it through the 1980s. Plan 9 had Bell Labs for fifteen years. Each of these systems benefited from an institutional environment in which a small group of designers could work on a long-horizon project without quarterly revenue pressure.

That kind of institutional environment is rare in 2026. The closest contemporary equivalents are the long-horizon research divisions of the largest tech companies (Google's various research efforts, Microsoft Research, Meta's Reality Labs, the open-source foundations supported by the cloud providers), and they are oriented toward different work than operating-system design. There is no contemporary Xerox PARC. There is no contemporary Bell Labs. The closest analog might be the academic operating-systems research community, which produces interesting work but operates on graduate-student timescales and budgets.

What this means: a Multics-scale or Plan-9-scale alternative OS project is, in 2026, very unlikely to find institutional sponsorship. The work that does get funded is incremental work on the existing OS stack, or domain-specific research that does not aim at full-system replacement. The alternative-systems energy that produced Genera and Plan 9 does not have a contemporary home.

## Partially recoverable: the cultural conditions

The cultural conditions in which the Lisp Machine and Smalltalk environments thrived — small communities of expert users committed to a particular system, willing to invest months in learning a deep environment, treating system modification as a normal user activity — are partially recoverable. They exist now in the small communities around Pharo, Glamorous Toolkit, Common Lisp, Emacs power-users, and similar enclaves. They do not exist at the scale of mainstream computing, and they are not coming back to that scale.

A person who wants to live in such a community in 2026 can. They have to seek it out. They have to commit to spending their time in environments where most of the world's software does not run. They get, in exchange, much of the working experience that the original Lisp Machine and Smalltalk users had. The community is small. The community exists.

This is, I think, the most encouraging finding in this chapter. The cultural property — being part of a small community of expert users of a deep system — is independent of any specific technology. As long as there is one alternative environment with a working user base and an active maintainer set, that property is available to anyone willing to join. In 2026 there are several such environments. Their continued existence depends on the people who run them, not on industry trends. The communities have survived thirty to forty years of Unix's dominance and they are likely to keep surviving.

## What to do with this inventory

A reader who has gotten this far is probably looking for guidance. Here is what I would offer:

*If you are a working developer in 2026* and you have never used an image-based environment or a deep Lisp REPL workflow, spend a week with Pharo or with SBCL+SLIME. The investment is small. The mental adjustment is substantial. You will, at the end of the week, understand things about software development that you cannot understand from reading about them. You will also see, more clearly than this book can describe, why the people who use these environments do not give them up.

*If you are designing a system in 2026* that has any open-ended exploratory character — a research tool, a long-running internal product, a complex domain-specific application — consider whether the file-based architecture you are about to default to is actually what suits the work. The image-based alternative is available. The file-based alternative is the default; the alternative requires conscious choice.

*If you are an educator* — formal or informal — preparing students for the contemporary industry, you have an obligation to show them at least one alternative to the dominant model. This does not mean teaching Smalltalk as a course; it means making sure students know the alternative exists and have at least encountered it. A graduate who has never seen anything except Unix-derived systems and web-derived applications is missing knowledge that, at the level of design intuition, they cannot recover later without effort. The alternatives are not large. They are not hard to find. They are easy to omit, and they are systematically being omitted.

*If you are designing a system that other people will use*, take seriously the question of where the user's data lives, who owns it, and how composable it is with other systems' data. The web's federated-sites model has produced enormous value and enormous costs. Many of the costs come from the model's assumption that each site is sovereign over its own data. Alternative architectures — where data lives with the user rather than with the application, where composition between applications is enabled rather than blocked — are possible, are being built, and are worth knowing about.

*If you are responsible for what gets taught in computer science programs*, look at whether your operating-systems curriculum still spends time on alternatives to the Unix-derived model. Most do not. Most spend their time on the implementation of Unix-derived kernels, with brief mentions of other systems. The Multics, Lisp Machine, Plan 9, and Smalltalk literatures are accessible. They should be in front of students who are going to spend their careers designing systems.

## The honest summary

The honest summary of what was lost when Unix won is this. The pieces are recoverable. The integration is not, except inside guest environments that pay their own integration costs. The cultural conditions in which the original integrations were built are partially recoverable in small communities and not recoverable at the industry scale. The hardware-software integration that some of the alternatives depended on is permanently gone in the consumer space and only available to a few very large platform owners. The trust assumptions that allowed certain kinds of open systems are permanently gone for reasons that have nothing to do with Unix.

What this means for a designer in 2026 is that the design space is wider than the default suggests, but the available choices are constrained in ways that the original alternatives' designers did not face. A new alternative system, if it were to be built, would have to be a Pharo-style guest in a commodity OS, not a Lisp Machine on custom silicon. It would have to assume a hostile network, not a trusted community. It would have to support standard collaboration patterns, not require single-user image-based commitment. It would have to be approachable for incremental adoption, not require months of investment before producing value. These constraints are not what killed the original alternatives, but they are what shapes the design space for any successor.

The closing chapter returns to the thesis. The point of recovering the visibility of these alternatives is not to revive them as they were. The point is to make the design space visible again, so that the next round of decisions — whatever they turn out to be about — is made by people who can see more than one option.
