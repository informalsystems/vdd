# Informal Research Goals

## Why are we doing research / research goals

1. Get better decentralized infrastructures

   a. More correct and reliable
   
   b. More efficient
   
2. Formal specification that make precise 

   a. what the system is doing and 
   
   b. under what assumptions (make hidden assumptions explicit)
   
   c. what is the security model assumed and what are provided
   security guarantees
   
3. Precise specifications that allow implementations in other
   languages, by other teams
   
4. The system is designed to be run by others (validators). Thus we
need to make precise what is expected by a validator (what does it mean to execute the protocol correctly, e.g., if a single validator is implemented by some replication mechanism)
5. External visibility. Dissemination of our results at scientific venues.

## Research Areas
- **DA**: Distributed Algorithms (consensus, gossip, fork accountability, light client)
- **Verif**: Automated Verification (current codebase, research to be able to verify all core protocols or eventually the complete implementation)
- **MD**: Mechanism Design (do the protocols incentivize “correct behavior”)
- **DB**: Database Research (AVL trees, Merkle trees)
- **Sys**: Systems Research (new protocols, efficiency)
- **Impl**:Implementation (improve quality, testability, etc.)
- **Sec**: Security and Crypto

The Sec area is currently covered by ICF funding external projects at
Berkeley and Stanford. The other areas we will cover internally and
with collaboration. 

**TODO:** Questions for
Ethan: provide content on the Crypto research for Sec so that we have a complete picture here


## People

**TODO:** In the following I have added the people from the
Collaborations task (in the goal tracking notion). Should we also mention engineers / research
engineers here?

### Informal

| Name | Interests |
|-----|-------------------------|
| Zarko (ZM) | |
| Ethan (EB) | |
| Josef (JW) | DA, Verif |
| Igor (IK) | |
| Adi (AS) | |
| Romain (RR) ||
| Cezara (CD soon) | ||




### External Collaborators 

| Name | Position |  Interests | Supported by ICF | period |
|-----|-----------|------------|---------------|
| Daniel Cason (DC) | PostDoc Lugano  | |
| Jovan Komatovic  (JK) | PhD EPFL from July 1st | |
| Fernando Pedone (FP) | Prof. Lugano | |
| Gianmarco Fraccaroli | Master student at USI | |
| Sara Tucci? | CEA (game theory, mechanism design) | |
| Cezara Dragoi | CR, Inria Paris / ENS | Verif. | yes | 2019 - now |
| Patrizio Inzaghi | PhD student, Inria Paris | " | " | 2019 - now |
| Nathalie Bertrand | DR Inria Rennes | Verification of Randomized Distributed Algorithms | no | 2019 - now |
| Bastien Thomas | PhD Student Inria Rennes | "  | " |  2019 - now |
| Jure Kukovec (JK) | PhD Student TU Wien | (APALACHE model checker) | no |  2019 - now |
| Giuliono Losa | Galois | verif. | yes |  2020 - now ||


### Who works with whom


|    | ZM | EB | JW | IK | AS | RR |
|----|----|----|----|----|----|----|
| DC |    |    |    |    |    |    |
| CD |    |    | C1 |    |    |    |
| JK |    |    |    | C2 |    |    |
| GL |    |    |    | C3 |    |    ||


### C1

See T2 below.

### C2

See T3 below.



----




# Old task descriptions

**TODO:** This is what we planned at summer/fall 2019.

## Who does what in 2019?

| Nr. | Task | Area | Who (perhaps with task leader first) | Towards which goal |
|-----|------|------| -------------------------------------|--------------------|
| T1 | Light client | DA | Josef, Zarko, Igor | 1,2,3 |
| T2 | Async to Sync Verif | Verif | Josef, Zarko | 1a, 2 |
| T3 | Formalization and Verification in TLA+ | Verif | Igor, Josef, Zarko, | 1a,2,3,4 |
| T4 | Gossip | DA | Fernando, Zarko | 1, 2, 3 |
| T5 | Mechanism Design | MD | | |
| T6 | Fork accountability | DA | Zarko | 1,2,3 |
| T7 | Merkleized data store | DB | | | |







### T1. Light client specification

#### High-level objective.

The Light Client is a participate in the system that instead of learning all blocks of the block chain, selectively tries to obtain blocks (headers) that appear relevant to it. This is relevant for accessing blocks with mobile devices. Since this implies it does not locally have a complete list of proof that all the blocks were signed properly, the light client needs to obtain proof (trust) with few communication and local computation. Besides light-weight learning of specific blocks will also be crucial for Inter-Blockchain Communication IBC.
After we have a proper understanding of light client we will move
towards IBC.

#### State of the art.
We have an informal understanding of the specification (the problem to be solved by the light client) and the environment assumption (faults, timing, etc.) under which this should be solved.

#### Todo.

Formalize 

- Specifications - roughly 2 weeks 
- Assumptions  - roughly 2 weeks 
- A solution (a distributed algorithm) - roughly 2 weeks
- Automated test case generation (SMT) - roughly 4 weeks of work
- Automated Verification at protocol level - roughly 4 weeks of work (assuming Igor is holding gun :))
- Accompany engineers in the implementation, and use feedback to update specs, assumptions, and algorithm


### T2. Reducing verification of asynchronous protocols to synchronous
#### High-level objective.
We are convinced that communication closure is a design principle that (i) does not restrict the designer or implementer too much (if at all), and (ii) allows the most efficient reduction methods for verification of asynchronous protocols.

#### State of the art.
In the benign case we know how to do it based on communication closure (CAV 2019). Byzantine protocols (including Tendermint) are not communication-closed.

#### Todo.
For the Byzantine case:

- Reduction theorem for relaxed communication closure - 4 weeks
- Rewriting algorithm - 4 weeks
- Implementation of rewriting (current prototype works on a C fragment. Interfacing with Go, Rust) - lead Cezara (we will have regular discussions)
- Verification in Byzantine synchronous - lead Cezara (we will have regular discussions)
- Verification in ByMC by Igor

Two kinds of encodings to consider: (a) Control-flow version (b) upon version.

### T3. Formalization and Verification  in TLA+
#### High-level objective.
TLA+ is a formal language that seems to be a good candidate for communications between engineers, researchers, and verification engineers. Part of the project is to confirm its usefulness in a project. In addition to using the language as a means of communication, we develop/maintain the APALACHE model checker that will allow to deal with large scale distributed systems.
#### State of the art.
For verification there is TLC and the first release of APALACHE. Regarding formalization, the code base is currently refactored in the spirit of finite state machines which makes the gap towards formal modeling smaller. We use TLC to debug new TLA+ specifications. However, as the specifications get more detailed, TLC stops being useful, owing to the large state space to enumerate. This is where APALACHE kicks in. To make APALACHE a real tool, we are continuing its development together with TU Wien.
#### Todo.
- Specification of Fastsync -- together with Anca -- and generation of test cases. Refinement of the spec and the implementation, as soon as discrepancies are found. 2 more weeks in August.
- Model checking of Fastsync and identification of expected correctness properties. 1 week.
- Refactoring of APALACHE to improve its usability and speed -- together with Jure Kukovec. A long-term initiative, 1-2 days/week. 
- Preparing learning materials for the team to get accustomed with modeling in TLA+. 1-2 weeks.



### T4. Robust gossip protocols for decentralised applications 
#### High-level objective.
Tendermint is gossip based BFT SMR middleware. In the current design and implementation the consensus layer and the gossip layer are intertwined in a monolithic module. This makes understanding, testing and potential improvements very hard. It is also not clear what should be the responsibility of each layer (consensus and gossip). Furthermore, other Tendermint modules (such as Mempool, fast-sync, evidence reactor, etc) are also communicating by proprietary (every module has different protocol) gossip protocols, where all communication is at the network level multiplexed on top of the same TCP connections. 

The goal of this project is to separate the gossip layer from application modules (consensus, mempool, fast syncm etc), to formally specify the properties and responsibilities of each layer, and to come up with a unified gossip API that can be used by multiple application modules. 
#### State of the art.
- Current specification of the consensus protocol can be found here: https://arxiv.org/abs/1807.04938
- The current specification of the gossip and consensus module can be found here: https://docs.google.com/document/d/1gdyo2oqhrPUVRKVIks8jmKEwz_gP4KtORRY1wt47HTc/edit?ts=5cea8d62

#### Todo.

- Separate consensus and gossip logic by initially assuming very simple responsibility by the gossip layer (essentially just simple send and receive interface) - 1 week (review Daniel’s work)
- Introducing suitable abstractions at the gossip layer that allows application modules to express real time requirements and priorities on sending and reception of messages. - 2 weeks (to review and follow Daniel’s work)
- Identify optimal concurrency architecture that facilitates modularity, testability and understanding, but does not penalize performance too much. - 2 weeks (to review and follow Daniel’s work)

#### Remark.
There is a smaller gossip related project with Professor Pedone and
one of his master students with the goal to design more robust gossip
protocols (inspired by BAR Gossip protocol) for open systems where
total group membership is dynamic and not known by participants. This
project has the potential to grow into a bigger project. 

### T5. Mechanism design based analysis of Tendermint and Cosmos
#### High-level objective.
Open decentralised systems such as Tendermint and Cosmos pose a new challenge in algorithm design. As those kind of systems are not part of the same administration domain, traditional fault models that consider only correct and faulty processes seems inappropriate as network participants are driven by rational goals (for example, maximising profit). The goal of this project is analysing critical components of Tendermint and Cosmos SDK such as consensus and gossip protocols (on the Tendermint side) and staking, slashing and fees modules on the Cosmos side. 

#### State of the art.
There are few papers that have started this kind of analysis, for example:

- https://arxiv.org/abs/1902.07895
- http://cs-www.cs.yale.edu/homes/jf/FS.pdf
- http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.74.1028&rep=rep1&type=pdf
- https://www.cs.huji.ac.il/~dolev/pubs/leader_selfcopy.pdf

#### Todo.
Identify potential external collaborators that could help us with this project. Initial contact with Sara Tucci is made. 

### T6. Fork-accountability in Tendermint
#### High-level objective.
The goal of this project is to design an algorithm that allows identifying faulty processes in a presence of forks in Tendermint, i.e., in case the number of faulty processes is greater than ⅓ of the total voting power. This mechanism is an essential part of the security model as it disincentives nodes not to double vote (create forks) as it allows faulty nodes to be detected and punished. 
#### State of the art.
At this point, we are close to having a first draft of the theoretical foundation and the proof of concept implementation.
#### Todo.
- Formalise theory and propose an algorithm  - 3 weeks (for review)
- Plan an integration with Tendermint 

Remark: It is important to have a formal specification of Tendermint in the presence of forks. The first discussions on a light client spec showed that communication with the chain must be based on chain invariants. What this invariant is in the presence of forks, or what the guarantees should be is crucial also for the light client.

### T7. Design and implementation of Merkleized data store for Cosmos SDK
#### High-level objective.
Blockchain applications developed using Cosmos SDK (https://github.com/cosmos/cosmos-sdk) stores application state in a proprietary Merkleized data store (https://github.com/tendermint/iavl). The key value store that uses a Merkle tree is implemented as an application layer on top of normal key value database. This design seems fundamentally wrong and we are already having performance issues. Other blockchain projects are following the same pattern as there is no database with required design and semantics. The goal of this project is to design a Merkleized data store that would fulfill requirements (both semantic and performance wise) of blockchain applications. 
#### State of the art.
The area seems fairly unexplored. In the Wikipedia article on Merkle tree, there are some references to existing systems based on Merkle tree (https://en.wikipedia.org/wiki/Merkle_tree), where ZFS is probably the most stable and complete such system. 
#### Todo.
Identify collaborators from academia that can help us with this project. We have contacted Professor Manos Athanassoulis as he was referred to us as domain expert, and waiting for an answer. We should try to identify relevant researchers in Germany as various tree related research is common research theme there (domain expert said this).  

## Even Older List of Topics 

- Gossip layer
  - Formal specification of gossip layer
  - Define gossip API that will be exposed to several modules (Mempool, Consensus, StateSync) and that can take care about various concerns such as priority, resource usage, peer correctness and quality of service, attack resistance
  - BAR gossip - incentives based design tolerant to rational and Byzantine behaviour
  - Experimental evaluation of gossip layer
- Concurrency architecture of Tendermint 2.0
  - Develop concurrency architecture that will be understandable, performant and testable
  - Support for experimental modules
- Formal verification
  - Development of tools and methodology needed to automate design, implementation and verification processes
  - Formal verification of all core protocols
  - Test scenarios driven by verification tools
- Fork accountability
- Optimal state sync protocol
- Use of aggregate signatures in Tendermint 2.0
- Formal analysis of staking and slashing modules of Cosmos SDK
- IBC

