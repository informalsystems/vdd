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

We are developing an approach where verification and software
design and development go hand-in-hand. Our goal is to improve software 
engineering practices by relying on formal methods. Our approach focuses on
fault-tolerant distributed systems. 


## Introduction

- because fault-tolerance, we have to deal with time and faults


## Method at a glance: The Lite Client example

<< Discuss what documents there are, how they were written, discussed, re-written,
finalized. >>

<< links to documents >>

<< Discuss what validation and verification efforts have been done/planned.

## Development Steps and Documents

1. Idea:

  - The problem that we would like to solve, the involved parties
  - probably in a sequential view:
    e.g., 
        * the blockchain is a growing list, 
        * a liteclient is just a node which gets an integer h as input. 
        * The goal is to return the data that is stored at height h of the block chain
  
2. Protocol:

  - first level of solution
  - introduce the full-blown distribution: e.g.; 
        * the blockchain is implemented by validators (full nodes)
        * validators may be Byzantine
        * there is a node running the lite client
        * the lite client and the full nodes communicate by message passing
        * asynchrony, time...

- high level specs
- low level spec (towards implementation)
- TLA+
- Rust

<< mapping specs to templates >>

### Testing

### Verification

<< Verification log/reporting/artifacts >>


## Lessons learned
