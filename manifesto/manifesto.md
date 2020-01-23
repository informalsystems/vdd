These are the points that came out the planning sessions:

- What are responsibilities of specs?
- What are features of impl the spec should address? What level of details
    - High level protocol spec
    - Low level architecture spec
- English spec vs TLA+ spec
- Verification log/reporting/artifacts
- How does all this inform / synchronize with development ?
- Template for high and low level specs


# Verification-Driven Development: An Informal Manifesto

**Abstract.** Software bugs can be increasingly costly, e.g., blockchain
technology. At the same time the software is quite complex. For instance,
all-in-all the Tendermint protocols are written in x lines of Golang code.

Computer-aided verification is impractical, either because completely automated
verification methods (such as model checking) hit theoretical limits and
undecidability results, or because methods based on interactive theorem provers
require a huge amount of manual work and understanding of the protocols. Part of
the problem is that verification is often regarded as a task that happens "after
the fact", that is, when the software is written.

We are in the process of developing an approach where verification and software
design and development go hand-in-hand. We start by informal (English)
specifications that are the result of a collaboration of protocol designers,
software engineers, and verification engineers. Informal specifications contain
the information both to verify and to implement protocols. Then verification and
implementation proceed in parallel and inform each others.

We will discuss our ideas by using the Tendermint lite client as an example.

## Introduction

## Method at a glance: The Lite Client example

<< Discuss what documents there are, how they were written, discussed, re-written,
finalized. >>

<< links to documents >>

<< Discuss what validation and verification efforts have been done/planned.

## Development Steps and Documents

- high level specs
- low level spec (towards implementation)
- TLA+
- Rust

<< mapping specs to templates >>

### Testing

### Verification

<< Verification log/reporting/artifacts >>


## Lessons learned
