# Traceability

**Informal Systems**

**Version 2**

**Abstract**. This document contains a proposal on implementing
traceability in the process of verification-driven development. Software
developers are familiar with [requirements traceability], which can be
understood as "the ability to describe and follow the life of a
requirement in both a forwards and backwards direction"
\[[Wikipedia][requirements traceability]\]. The [VDD Manifesto] defines
several layers of specifications that have to be linked with the
implementation, unit tests, architectural proposals, etc. We propose a
concrete and lightweight approach that will allow us to implement
traceability without using heavy and expensive tooling. This approach
can be automated with plain text tools, though it can be also
implemented manually.

## 1. Process artifacts

For every distributed protocol, [VDD Manifesto] defines the following
artifacts (this list is incomplete):

-   High level English specification of the problem solved by the
    protocol. For instance, a specification of a sequential algorithm.
-   System model specification in English.
-   Protocol specification in English.
-   TLA+ specification of the protocol.
-   TLA+ specification of the expected behavior (invariants and temporal
    properties).
-   Implementation in a language of choice (Golang or Rust).
-   TLA+ specification of the state machines that correspond to the
    implementation code.
-   TLA+ specification of the invariants and temporal properties at the
    implementation level.
-   Abstract test scenarios.

It is clear that maintaining cross references between the logical units
(e.g., paragraphs of text, TLA+ operators, Rust functions) in all of
these artifacts is tedious and error prone. Maintaining github links is
a fragile solution, as the links expire, get moved across different
branches, etc.

## 2. Logical units

Before we discuss, the implementation of the traceability properties, we
classify logical units that should be tagged. These are:

-   **Requirements in English**. We should tag every paragraph (or
    several adjacent paragraphs) that defines a single requirement. We
    do not have to tag the text that provides the reader with the
    motivation, explanation, examples, etc.

-   **TLA+ operators**. We should tag the top-level operators, that is,
    the operators that do not have parents in the call graph.

-   **Implementation methods**. We should tag the principle functions
    and data structures. The main point is not to tag every single piece
    of code, but to tag the code that implements the requirements at
    higher levels. At the moment, it is not reasonable to pursue
    complete code coverage with tags.

-   **Unit tests**. We should tag every unit test, as they are the main
    proof that the requirements have been implemented.

## 3. Traceability properties

We need a solution that satisfies the following properties:

<span id="TRC-TAG.1">|TRC-TAG.1|</span>
:   Tagging a logical unit should be easy.

<span id="TRC-REF.1">|TRC-REF.1|</span>
:   Referencing a logical unit should be easy.

<span id="TRC-IMPL.1">|TRC-IMPL.1|</span>
:   Marking that one logical unit implements another one should be easy.

<span id="TRC-REV.1">|TRC-REV.1|</span>
:   Tagging revised logical units should be easy.

<span id="TRC-GRAPH.1">|TRC-GRAPH.1|</span>
:   There should be a way to automatically construct the traceability
    graph, that is, the graph connecting the logical units via tag
    references.

<span id="TRC-MISS.1">|TRC-MISS.1|</span>
:   There should be a way to automatically identify childless and
    orphaned logical units.

<span id="TRC-GITHUB-REF.1">|TRC-GITHUB-REF.1|</span>
:   The GitHub references should be updated automatically.

<span id="TRC-UNIQ.1">|TRC-UNIQ.1|</span>
:   Every identifier should be declared only once.

## 4. Tag syntax

### 4.1. Naming scheme

<span id="TRC-TAG.1::SYNTAX.1">|TRC-TAG.1::SYNTAX.1|</span>
:   We propose a simple naming scheme for tags. We start with the tags
    for top-level requirements and then proceed with the tags of the
    logical units that implement lower-level requirements.

**Zero-level requirement tags.** These are the tags of the requirements
that do not implement any requirement but are the zero-level
requirements themselves. These tags have the form `<NAME>.<REVISION>`.
The component `NAME` is a sequence of capital English letters and
(arabic) digits, possibly separated with a hyphen (-). The component
`<REVISION>` is a sequence of digits, starting with a non-zero digit.

Example: the above-defined tag **TRC-TAG.1** is a zero-level tag, whose
`<NAME>` component is `TRC-TAG`, whereas the `<REVISION>` component is
`1`.

**Non-zero-level requirement tags.** These are the tags of the logical
units that implement the higher-level requirements. For instance, these
may be the tags of English paragraphs that define requirements of lower
levels, of TLA+ operators, and of Rust methods. A tag of level `k` is of
the form `<PARENT>::<NAME>.<REVISION>`. The component `<PARENT>` is a
tag of level `k-1`, while `<NAME>` and `<REVISION>` are defined exactly
as in the case of zero-level tags.

Example: the above-defined tag **TRC-TAG.1::SYNTAX.1** is a tag of level
1.

Remark: We choose `::` as a tag separator, as it is familiar to Rust and
C++ programmers. There is no danger of confusing a Rust package name
with a tag, as the tags always come with revision numbers.

<span id="TRC-TAG.1::SYNTAX.1::SYNONYMY.1">|TRC-TAG.1::SYNTAX.1::SYNONYMY.1|</span> \|TRC-IMPL.1::SYNONYMY.1\|
:   Non-zero-level requirement tags (described above) record the
    "ancestry" of the higher level units they implement. When a logical
    unit implements multiple units from a higher level, it has multiple
    parents, multiple lines of ancestry, and, consequently, multiple
    valid tags. In such cases, we end up with two or more tags which
    refer to the same logical unit. These tags are *synonymous* (in
    terms of reference, but not sense) so we refer to this as "tag
    synonymy".

A logical unit is tagged with synonymous tags by separating the synonyms
with spaces. Given a unit designated by the leaf tag `T` that implements
`n` ancestor units with paths `path[0]...path[n]`, the unit is tagged
with `|path[0]::T| |path[1]::T| ... |path[n]::T|`.

Example: The previous logical unit implements both [TRC-TAG.1::SYNTAX.1]
and [TRC-IMPL.1]. As a result, it is tagged with both
[TRC-TAG.1::SYNTAX.1::SYNONYMY.1] and
[TRC-IMPL.1::SYNONYMY.1][TRC-TAG.1::SYNTAX.1::SYNONYMY.1], and so these
two tags are synonymous. However, all its tags end with the single leaf
tag `SYNONYMY.1`

\|{TRC-TAG.1::SYNTAX.1,TRC-IMPL.1}::SYNONYMY.1::BRACE-EXPANSION.1\|
:   [brace expansion] is used for a concise designation of synonyms:
    ancestor tag paths are separated by commas and surrounded in curly
    braces. Given a unit with the leaf tag `T` that implements `n`
    ancestor units with paths `path[0]...path[n]`, the unit is tagged
    with `|{path[0],path[1],...,path[n]}::T|`. This is equivalent to the
    concatenation of all tag synonyms, as specified in
    [TRC-TAG.1::SYNTAX.1::SYNONYMY.1].

Example: The previous unit implements both
[TRC-TAG.1::SYNTAX.1::SYNONYMY.1] and
[TRC-IMPL.1::SYNONYMY.1][TRC-TAG.1::SYNTAX.1::SYNONYMY.1], and has the
leaf tag `BRACE-EXPANSION.1`. It is tagged using the brace expansion
syntax
`|{TRC-TAG.1::SYNTAX.1,TRC-IMPL.1}::SYNONYMY.1::BRACE-EXPANSION.1|`,
which is equivalent to the sequence of tags
`|TRC-TAG.1::SYNTAX.1::SYNONYMY.1::BRACE-EXPANSION.1| |TRC-IMPL.1::SYNONYMY.1::BRACE-EXPANSION.1|`.

### 4.2. Tagging logical units

<span id="TRC-TAG.1::DEF.2">|TRC-TAG.1::DEF.2|</span>
:   A logical unit should be tagged with a tag according to where the
    tag is being created. If the tag is created in a specification,
    assuming that the specification is written in Markdown format, the
    logical unit must be tagged using [PHP Markdown Extra's Definition
    List format] and must be surrounded by pipe symbols (`|`).

Example:

``` {.sourceCode .markdown}
|TRC-TAG.1|
: This is the logical unit to which [TRC-TAG.1] refers. Text content related
  to the logical unit can span multiple lines.

  Text content related to this logical unit can also span multiple
  paragraphs, as long as those paragraphs are indented to align with the text
  of the first line of the logical unit following the colon (:).
```

Note that several lower-level requirements may implement the same
higher-level requirement. For instance, the requirements labelled with
[TRC-TAG.1::SYNTAX.1] and [TRC-TAG.1::DEF.2] implement the requirement
[TRC-TAG.1].

### 4.3. Referring to logical units

<span id="TRC-REF.1::SYNTAX.2">|TRC-REF.1::SYNTAX.2|</span>
:   To refer to a tag TAG, one surrounds the tag name with square
    brackets, that is, `[TAG]`. The tag syntax is sufficiently unique to
    automatically identify a tag in text or in code.

Example: [TRC-REF.1::SYNTAX.2] is a reference to the tag for this
requirement.

### 4.4. Implementing logical units

<span id="TRC-IMPL.1::PREFIX.1">|TRC-IMPL.1::PREFIX.1|</span>
:   The fact that a logical unit implements another logical unit is
    reflected by the tag naming scheme. The name of a tag of level `k`
    has the form `<PARENT>::<NAME>.<REVISION>`. The name indicates that
    the logical unit labelled with the tag implements the logical unit
    that is labelled with the tag `<PARENT>`.

Example: the requirement described in this previous paragraph has the
tag [TRC-IMPL.1::PREFIX.1] that implements the requirement [TRC-IMPL.1].

### 4.5. Revising logical units

<span id="TRC-REV.1::INC.1">|TRC-REV.1::INC.1|</span>
:   Whenever the behavior of a logical unit has been changed -- this
    decision being done by the maintainer of the logical unit -- the tag
    revision should be incremented. The revisions of the parent tags
    should stay intact.

## 5. Automation

Tagging logical units requires manual effort from researchers,
verification engineers, and system engineers. As the tagging scheme is
simple and non-intrusive, we believe that the tagging process itself
does not require any automation. However, we indeed need automation to
keep the logical units and their tags consistent.

<span id="TRC-GRAPH.1::BUILD.1">|TRC-GRAPH.1::BUILD.1|</span>
:   From the conceptual point of view, it should be easy to construct
    the traceability graph. One has to check out all repositories that
    contain English specifications, TLA+ specifications, and the
    implementation code. Once it is done, the source files should be
    grepped for the regular expression that defines a tag, see
    [TRC-TAG.1::DEF.2] and [TRC-TAG.1::SYNTAX.1]. Having extracted the
    tags, we can immediately build the graph, as the non-zero-level tags
    contain the names of the parent tags. (Due to that, the graph edges
    do not have to be built at all.)

Example. The figure below shows the traceability graph of the
requirements that are introduced in this document.

             .----> TRC-TAG.1 <--.               TRC-REF.1
             |                   |                   ^
             |                   |                   |
    TRC-TAG.1::SYNTAX.1   TRC-TAG.1::DEF.2    TRC-REF.1::SYNTAX.2

       .-> TRC-IMPL.1         .-> TRC-REV.1
       |                      |                     <etc.>
       |                      |
    TRC-IMPL.1::PREFIX.1    TRC-REV.1::INC.1

<span id="TRC-GRAPH.1::CI.1">|TRC-GRAPH.1::CI.1|</span>
:   Obviously, searching for the tags from scratch will take plenty of
    time. It is more reasonable to update the tags database when new
    commits arrive in a git repository. Such updates could be
    implemented with continuous integration tools.

<span id="TRC-MISS.1::ANALYSIS.1">|TRC-MISS.1::ANALYSIS.1|</span>
:   Once we have collected the database of tags (see
    [TRC-GRAPH.1::BUILD.1]), we can identify the tags that have the
    following properties:

1.  The tags of the form `PARENT::NAME.REVISION` that do not have the
    corresponding `PARENT` tag registered in the database (orphan tags).

2.  The tags of the form `PARENT` that do not have a single child tag
    `PARENT::NAME.REVISION` (childless tags).

The continuous integration tool should report on the orphan tags and
childless tags. If we do not like to see the leaf tags in the report, we
should introduce a notation for the implementation tags that are not
supposed to have children.

<span id="TRC-MISS.1::OUTDATED.1">|TRC-MISS.1::OUTDATED.1|</span>
:   The tag revisions may help us in finding outdated implementations.
    For instance, let us assume that a function `bar()` is labelled with
    the tag `FOO.1::BAR.1`, and `FOO` has been advanced to revision 2,
    that is, there is a requirement `FOO.2`, but no requirement `FOO.1`.
    In this case, the tool could report that the function `bar()`
    implements the outdated requirement `FOO.1`.

<span id="TRC-GITHUB-REF.1::IMPL.1">|TRC-GITHUB-REF.1::IMPL.1|</span>
:   When collecting the tags in the process of [TRC-GRAPH.1::BUILD.1],
    we can record the source location of every tag. Having the source
    locations, it is easy to replace a reference to every tag with a
    hyperlink (yeah!) to the source location.

<span id="TRC-GITHUB-REF.1::DICT.1">|TRC-GITHUB-REF.1::DICT.1|</span>
:   If we do not like to update the source code with links to other
    repositories, we can build a dictionary of tags and their source
    locations. A simple solution would be to create a page that contains
    tag names and links to the source locations of their definitions.

<span id="TRC-UNIQ.1::DUPS.1">|TRC-UNIQ.1::DUPS.1|</span>
:   Again, when collecting the tags in [TRC-GRAPH.1::BUILD.1], we can
    check that every tag is defined only once. The continuous
    integration tool should report the violations of [TRC-UNIQ.1].

<span id="TRC-REV.1::INC.1::TOOL.1">|TRC-REV.1::INC.1::TOOL.1|</span>
:   In order to manage change, the tool should check whether the content
    corresponding to a tag (English, TLA+, code) has changed since the
    last commit. The tool should suggest to update to a new version
    number. The user then decides whether a new version number is
    necessary. If the version number is updated, the tool should show
    all references to the old tag. For each of these, the user has to
    decide to update the references to the new version, or keep them to
    the old one. This functionality should be available to the user to
    check the current working branch, but the functionality should also
    be part of CI. Also if a new version is introduced, perhaps we can
    collect the old versions in a way that is easily accessible. The
    information about all tags is already in the repo, but we should
    make it easily accessible.

<span id="TRC-UNIQ.1::BRANCHES.1">|TRC-UNIQ.1::BRANCHES.1|</span>
:   The approach of [TRC-UNIQ.1::DUPS.1] will report false positives, if
    a git repository contains multiple branches. Indeed, multiple
    branches may contain the definitions of the same tag that is defined
    in a common git commit. In this case, the tag analyser should
    analyse the commit history. By doing so the analyser can test,
    whether the potential duplicates belong to the same commit, and thus
    are uniquely defined.

  [requirements traceability]: https://en.wikipedia.org/wiki/Requirements_traceability
  [VDD Manifesto]: https://github.com/informalsystems/VDD/blob/master/manifesto/manifesto.md
  [TRC-TAG.1::SYNTAX.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-TAG.1::SYNTAX.1
  [TRC-IMPL.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-IMPL.1
  [TRC-TAG.1::SYNTAX.1::SYNONYMY.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-TAG.1::SYNTAX.1::SYNONYMY.1
  [brace expansion]: www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html
  [PHP Markdown Extra's Definition List format]: https://michelf.ca/projects/php-markdown/extra/#def-list
  [TRC-TAG.1::DEF.2]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-TAG.1::DEF.2
  [TRC-TAG.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-TAG.1
  [TRC-REF.1::SYNTAX.2]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-REF.1::SYNTAX.2
  [TRC-IMPL.1::PREFIX.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-IMPL.1::PREFIX.1
  [TRC-GRAPH.1::BUILD.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-GRAPH.1::BUILD.1
  [TRC-UNIQ.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-UNIQ.1
  [TRC-UNIQ.1::DUPS.1]: https://github.com/informalsystems/vdd/blob/master/traceability/traceability.md#TRC-UNIQ.1::DUPS.1
