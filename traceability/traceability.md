# Traceability

**Informal Systems**

**Version 1**

**Abstract**. This document contains a proposal on implementing traceability in
the process of verification-driven development. Software developers are
familiar with [requirements
traceability](https://en.wikipedia.org/wiki/Requirements_traceability), which
can be understood as "the ability to describe and follow the life of a
requirement in both a forwards and backwards direction"
[[Wikipedia](https://en.wikipedia.org/wiki/Requirements_traceability)]. The
[VDD
Manifesto](https://github.com/informalsystems/VDD/blob/master/manifesto/manifesto.md)
defines several layers of specifications that have to be linked with the
implementation, unit tests, architectural proposals, etc. We propose a concrete
and lightweight approach that will allow us to implement traceability without
using heavy and expensive tooling. This approach can be automated with plain
text tools, though it can be also implemented manually.

## 1. Process artifacts

For every distributed protocol, [VDD
Manifesto](https://github.com/informalsystems/VDD/blob/master/manifesto/manifesto.md)
defines the following artifacts (this list is incomplete):

 * High level English specification of the problem solved by the protocol.
    For instance, a specification of a sequential algorithm.
 * System model specification in English.
 * Protocol specification in English.
 * TLA+ specification of the protocol.
 * TLA+ specification of the expected behavior (invariants and temporal properties).
 * Implementation in a language of choice (Golang or Rust).
 * TLA+ specification of the state machines that correspond to the implementation code.
 * TLA+ specification of the invariants and temporal properties at the implementation
   level.
 * Abstract test scenarios.

It is clear that maintaining cross references between the logical units (e.g.,
paragraphs of text, TLA+ operators, Rust functions) in all of these artifacts
is tedious and error prone. Maintaining github links is a fragile solution, as
the links expire, get moved across different branches, etc.

## 2. Logical units

Before we discuss, the implementation of the traceability properties, we
classify logical units that should be tagged. These are:

 * **Requirements in English**. We should tag every paragraph (or several adjacent
   paragraphs) that defines a single requirement. We do not have to tag the
   text that provides the reader with the motivation, explanation, examples,
   etc.

 * **TLA+ operators**. We should tag the top-level operators, that is, the operators
 that do not have parents in the call graph.

 * **Implementation methods**. We should tag the principle functions and data structures.
 The main point is not to tag every single piece of code, but to tag the code that
 implements the requirements at higher levels. At the moment, it is not reasonable
 to pursue complete code coverage with tags.

 * **Unit tests**. We should tag every unit test, as they are the main proof that the
 requirements have been implemented.

## 3. Traceability properties

We need a solution that satisfies the following properties:

 1. **[TRC-TAG.1]** Tagging a logical unit should be easy.
 
 1. **[TRC-REF.1]** Referencing a logical unit should be easy.

 1. **[TRC-IMPL.1]** Marking that one logical unit implements another one
 should be easy.

 1. **[TRC-REV.1]** Tagging revised logical units should be easy.

 1. **[TRC-GRAPH.1]** There should be a way to automatically construct the
 traceability graph, that is, the graph connecting the logical units via tag
 references.
 
 1. **[TRC-MISS.1]** There should be a way to automatically identify
 childless and orphaned logical units.
 
 1. **[TRC-GITHUB-REF.1]** The github references should be updated
 automatically.
 
 1. **[TRC-UNIQ.1]** Every identifier should be declared only once.

## 4. Tag syntax

### 4.1. Naming scheme

**[TRC-TAG.1::SYNTAX.1]** We propose a simple naming scheme for tags. We start with
the tags for top-level requirements and then proceed with the tags of the
logical units that implement higher-level requirements.

**Zero-level requirement tags.** These are the tags of the requirements that do
not implement any requirement but are the zero-level requirements themselves.
These tags have the form `<NAME>.<REVISION>`. The component `NAME` is a sequence of
capital English letters and (arabic) digits, possibly separated with the minus
sign. The component `<REVISION>` is a sequence of digits, starting with a non-zero
digit.

Example: the above-defined tag **TRC-TAG.1** is a zero-level tag, whose
`<NAME>` component is `TRC-TAG`, whereas the `<REVISION>` component is `1`.

**Non-zero-level requirement tags.** These are the tags of the logical units
that implement the higher-level requirements. For instance, these may be the tags
of English paragraphs that define requirements of lower levels, of TLA+
operators, and of Rust methods. A tag of level `k` is of the form
`<PARENT>::<NAME>.<REVISION>`.  The component `<PARENT>` is a tag of level
`k-1`, while `<NAME>` and `<REVISION>` are defined exactly as in the case of
zero-level tags.

Example: the above-defined tag **TRC-TAG.1::SCHEME.1** is a tag of level
1.

### 4.2. Tagging logical units

**[TRC-TAG.1::DEF.1]**
A logical unit should be tagged with a tag `TAG` by writing `[TAG]` in the
comment that precedes the unit (and there is no other comment block between the
unit and the comment that contains the tag).

Example: the requirement that is defined in the previous paragraph is labelled
with the tag **TRC-TAG.1::DEF.1**.

Note that several lower-level requirements may implement the same higher-level
requirement.  For instance, the requirements labelled with
**TRC-TAG.1::SCHEME.1** and **TRC-TAG.1::DEF.1** implement the requirement
**TRC-TAG.1**.

### 4.3. Referring to logical units

**[TRC-REF.1::SYNTAX.1]** To refer to a tag, one should simply use the tag name
without the square brackets. The tag syntax is sufficiently unique to
automatically identify a tag in text or in code.

Example: **TRC-TAG.1::SYNTAX.1** is a reference to the tag that is defined in
the previous paragraph.

### 4.4. Implementing logical units

**[TRC-IMPL.1::PREFIX.1]** The fact that a logical unit implements another
logical unit is reflected by the tag naming scheme. The name of a tag of level
`k` has the form `<PARENT>::<NAME>.<REVISION>`. The name indicates that the
logical unit labelled with the tag implements the logical unit that is labelled
with the tag `<PARENT>`.

Example: the requirement described in this previous paragraph has the tag
**TRC-IMPL.1::PREFIX.1** that implements the requirement **TRC-IMPL.1**.

### 4.5. Revising logical units

**[TRC-REV.1::INC.1]** Whenever the behavior of a logical unit has been changed
-- this decision being done by the maintainer of the logical unit -- the tag
revision should be incremented. The revisions of the parent tags should stay
intact.

## 5. Automation 

Tagging logical units requires manual effort from researchers, verification
engineers, and system engineers. As the tagging scheme is simple and
non-intrusive, we believe that the tagging process itself does not require any
automation. However, we indeed need automation to keep the logical units and
their tags consistent.

**[TRC-GRAPH.1::BUILD.1]** From the conceptual point of view, it should be easy
to construct the traceability graph. One has to check out all repositories that
contain English specifications, TLA+ specifications, and the implementation
code. Once it is done, the source files should be grepped for the regular
expression that defines a tag, see **TRC-TAG.1::DEF.1** and
**TRC-TAG.1::SYNTAX.1**. Having extracted the tags, we can immediately build
the graph, as the non-zero-level tags contain the names of the parent tags.
(Due to that, the graph edges do not have to be built at all.)

Example. The figure below shows the traceability graph of the requirements that
are introduced in this document.

```
         .----> TRC-TAG.1 <--.               TRC-REF.1
         |                   |                   ^
         |                   |                   |
TRC-TAG.1::SYNTAX.1   TRC-TAG.1::DEF.1    TRC-REF.1::SYNTAX.1

   .-> TRC-IMPL.1         .-> TRC-REV.1
   |                      |                     <etc.>
   |                      |
TRC-IMPL.1::PREFIX.1    TRC-REV.1::INC.1
```

**[TRC-GRAPH.1::CI.1]** Obviously, searching for the tags from scratch will take
plenty of time. It is more reasonable to update the tags database when new commits
arrive in a git repository. Such updates could be implemented with continuous
integration tools.

**[TRC-MISS.1::ANALYSIS.1]** Once we have collected the database of tags (see
**TRC-GRAPH.1::BUILD.1**), we can identify the tags that have the following
properties:

 1. The tags of the form `PARENT::NAME.REVISION` that do not have the
 corresponding `PARENT` tag registered in the database (orphan tags).

 1. The tags of the form `PARENT` that do not have a single child tag
 `PARENT::NAME.REVISION` (childless tags).

The continuous integration tool should report on the orphan tags and childless
tags.  If we do not like to see the leaf tags in the report, we should
introduce a notation for the implementation tags that are not supposed to have
children.

**[TRC-MISS.1::OUTDATED.1]** The tag revisions may help us in finding outdated
implementations. For instance, let us assume that a function `bar()` is labelled
with the tag `FOO.1::BAR.1`, and `FOO` has been advanced to revision 2, that
is, there is a requirement `FOO.2`, but no requirement `FOO.1`. In this case,
the tool could report that the function `bar()` implements the outdated requirement
`FOO.1`.

**[TRC-GITHUB-REF.1::IMPL.1]** When collecting the tags in the process of
**TRC-GRAPH.1::BUILD.1**, we can record the source location of every tag.
Having the source locations, it is easy to replace a reference to every tag
with a hyperlink (yeah!) to the source location.

**[TRC-GITHUB-REF.1::DICT.1]** If we do not like to update the source code with
links to other repositories, we can build a dictionary of tags and their source
locations. A simple solution would be to create a page that contains tag names
and links to the source locations of their definitions.

**[TRC-UNIQ.1::DUPS.1]** Again, when collecting the tags in
**TRC-GRAPH.1::BUILD.1**, we can check that every tag is defined only once. The
continuous integration tool should report the violations of **TRC-UNIQ.1**.

**[TRC-UNIQ.1::BRANCHES.1]** The approach of **TRC-UNIQ.1::DUPS.1** will report
false positives, if a git repository contains multiple branches. Indeed,
multiple branches may contain the definitions of the same tag that is defined
in a common git commit.  In this case, the tag analyser should analyse the
commit history. By doing so the analyser can test, whether the potential
duplicates belong to the same commit, and thus are uniquely defined.
