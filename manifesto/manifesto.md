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

### 1. Problem Statement:

#### Points addressed

  - The problem that we would like to solve, the involved parties
  - probably in a sequential view:
    e.g.,

        * the blockchain is a growing list,
        * a liteclient is just a node which gets an integer h as input.
        * The goal is to return the data that is stored at height h of the block chain

#### Expected Expertise

- **lead:** distributed algorithm designer
- **input/feedback:**


  << this might be super abstract, and only make sense in combination with the Protocol in the beginning. >>

#### Artefacts

  - Section 1 of high-level English spec.
  - TLA+ Spec (in case we want to do TLA+ reductions)

#### Verification, Validation, or Proof Obligation

#### Versioning

- v0.1 first draft
- v0.2 After feedback from point 2

<< figure that one out

### 2. Protocol Specification

#### Points addressed

  - first level of solution
  - introduce the full-blown distribution: e.g.;

        * the blockchain is implemented by a set of validators and other full nodes
        * validators may be Byzantine
        * there is a node running the lite client
        * the lite client and the full nodes communicate by message passing
        * asynchrony, time...

  - refinement of problem statement: e.g.;

        * in case of at most 1/3 faulty validators safety property A and liveness B
        * in case of between 1/3 and 2/3 then safety property A' and liveness B'
        * if the liteclient is connected to a correct full node and communication is
           timely, then C and D

#### Expected Expertise

  - **lead:** distributed algorithm designer
  - **input/feedback:** distributed systems engineers

#### Artefacts

  - Section 2 of high-level English spec.
  - TLA+ Spec (in case we want to do TLA+ reductions)
  - refined Problem Statement

#### Verification, Validation, or Proof Obligation

  - Simulation relation. How we map events/executions in a faulty and distributed environment to the ideal specification from Point 1.

#### Versioning

### 3. Implementation Specification


#### Points addressed

 - API
 - Error Handling
 - Tests that abstract the environment

#### Expected Expertise
- **lead:** software engineer, adversarial engineer
- **input/feedback:**

#### Artefacts
#### Verification, Validation, or Proof Obligation
#### Versioning


### 4. Implementation
#### Points addressed
#### Expected Expertise
- **lead:** software engineer
- **input/feedback:**

#### Artefacts
#### Verification, Validation, or Proof Obligation
#### Versioning


### 5. Optimized Implementation
#### Points addressed
#### Expected Expertise
- **lead:** software engineer
- **input/feedback:**

#### Artefacts
#### Verification, Validation, or Proof Obligation
#### Versioning




## Lessons learned
