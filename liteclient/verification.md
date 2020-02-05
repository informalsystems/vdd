*** This is the beginning of an unfinished draft. Don't continue reading! ***

# << Name of Component >>

<< Rough outline of what the component is doing and why. 2-3 paragraphs >>

# Part I - Outside view

## Context of this document

<< mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc. >>

should give the reader the understanding in what environment this component
will be used.

## Informal Problem statement

for the general audience


## Sequential Problem statement


# Part II - Protocol view

## Environment/Assumptions/Incentives

<< Introduce distributed aspects >>

<< Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly >>

## Problem Statement

<< safety specifications / invariants in English >>

<< liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually. >>

should have clear formalization in temporal logic.

## Definitions

In this section we become more concrete, with basic data types,

some math that allows to write specifications and pseudo code solution below.
Some variables, etc.

## Solution

<< Basic data structures. Simplified, so that we can focus on the distributed
algorithm here. If existing: link to Tendermint data structures, and mentioned
if details were omitted. >>

<< Pseudo code of the solution >>


## Correctness arguments

<< Proof sketches of why we believe the solution satisfies the specifications.
Possibly giving inductive invariants that can be used to prove the specifications >>
<<Link to Part I>>
