# IBC Connection


## 1. Problem Statement / Outside view

IBC connection is a protocol that ensures establishment of connection
between two modules. At the end of the connection handshake at both
modules there is common view on the corresponding data structures.

One of the party (called sometimes source or module B) is triggering
connection establishment and the other module follows.

Properties that should hold:

- data structures have consistent view at the end of the handshake
- protocol eventually terminates


#### Expected Expertise

- **lead:** Adi
- **input/feedback:** IBC team

#### Artifacts

  - High level English specification of the problem:
    ibc-rs/docs/spec/connection/L1_2.md

#### Verification, Validation, or Proof Obligation

- N.A.

### 2. Protocol Specification / Protocol view

#### 2.1 System Model Specification

##### Expected Expertise

  - **lead:** Adi
  - **input/feedback:** IBC team

##### Artifacts

  - High-level English specification:
    ibc-rs/docs/spec/connection/L1_2.md

##### Verification, Validation, or Proof Obligation

  - Informal check that the problem refinement in the given model
  corresponds to the top level problem definition.

#### 2.2 Algorithm (Protocol) Specification

##### Expected Expertise

  - **lead:** Adi, Zarko
  - **input/feedback:** IBC team

##### Artifacts

  - English description of the protocol: ibc-rs/docs/spec/connection/L1_2.md
  - TLA+ specification of the protocol: ibc-rs/docs/spec/connection/L2.tla
  - TLA+ specification of properties as invariants and temporal properties:
    ibc-rs/docs/spec/connection/L2.tla

##### Verification, Validation, or Proof Obligation

  - Check that the protocol satisfies the invariants and temporal logic
  properties


### 3. Single Node View

#### Expected Expertise

- **lead: Ilina
- **input/feedback:** IBC team

#### Artifacts

- Code of the IBC connection module (currently Go implementation)
- TLA+ specification of the IBC connection module (multi connection support,
data store, light client management, error handling, etc):
ibc-rs/docs/spec/connection/L3.tla
- TLA+ specification of properties introduced at the
implementation level (for example concurrency related) expressed as
invariants and temporal properties:
ibc-rs/docs/spec/connection/L3.tla

#### Verification, Validation, or Proof Obligation

- Check that the specification satisfies the invariants and temporal logic
  properties
- TLA+ reductions / refinement mappings with respect to the high level
specification
- Check that code passes tests that corresponds to (automatically) generated
abstract execution scenarios.
