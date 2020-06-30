# VDD Naming Convetions / Semantic Versioning

In order to facilitate tracability over several iterations of English
and TLA+ specifications, and to make clear to external readers what is
the state of a given document, we use the following convention:

## English Spec

As part of the version, each English specification should have one
of the following stages:

```
Draft --> Proposal --> Reviewed
             |
             |--> Published
```


- Draft should live in auxiliary branches
- Proposal should live in a PR
- Reviewed, Published, Withdrawn should live in the main
  (f.k.a. master) branch
- From any stage, if updates need to be made, we first transition back
  to Draft.
- There should be an additional stage `Withdrawn`
  
  
Whenever the stage changes for a English specification, the version
number should increase. Here an example trace

Draft.1 -> Proposal.2 -> Reviewed.3 -> Draft.4 -> Proposal.5 ->
Published.6


## TLA+ Spec

Draft --> Proposal --> Reviewed --> 
Validation Draft --> Validation Proposal --> Validated --> 
Verification Draft --> Verification Proposal --> Verified

**TODO** there should also be a withdrawn

- Draft, Proposal, Reviewed work similar to the English spec
- when simple invariants and type checks are added the spec should
  transition to Validation Draft, and when it passes model checking it
  should be called Validated
- if we go towards full verification (to be defined; parameterized?)
  we go towards verification draft, and when the model checker
  verifies it, it should move to verified.

- All drafts live in branches
- Proposals live in PRs
- Reviewed, Validated, Verified, live in the main branch
- There should be an additional stage `Withdrawn`
