---
title: "Benchmarking Methodology for Network Management AI Agents on Brownfield Networks"
abbrev: "NMAA Benchmarking Methodology"
category: info

docname: draft-salsano-bpm-nmaa-benchmarking-methodology-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Benchmarking & Performance Measurements"
keyword:
 - AI agents
 - network management
 - benchmarking
 - collateral damage
venue:
  group: "Benchmarking & Performance Measurements"
  type: "Working Group"
  mail: "bpm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/bpm/"
  github: "netgroup/draft-nmaa-benchmarking-methodology"
  latest: "https://netgroup.github.io/draft-nmaa-benchmarking-methodology/draft-salsano-bpm-nmaa-benchmarking-methodology.html"

author:
 -
    fullname: Stefano Salsano
    organization: University of Rome Tor Vergata / CNIT
    email: stefano.salsano@uniroma2.it
 -
    fullname: Luis M. Contreras
    organization: Telefonica (to be confirmed)
    email: luismiguel.contrerasmurillo@telefonica.com
 -
    fullname: Mohammad-Hadi Hossaini
    organization: STORDIS (to be confirmed)

normative:

informative:
  RFC1242:
  RFC2544:
  RFC6815:
  I-D.contreras-bmwg-ai-agent-benchmarking:
  I-D.cui-nmrg-llm-benchmark:
  I-D.gaikwad-llm-benchmarking-terminology:
  I-D.gaikwad-llm-benchmarking-methodology:
  I-D.gaikwad-llm-benchmarking-profiles:
  FRAMEWORK:
    title: "A Framework for Controlled Benchmarking of Network Management AI Agents on Brownfield Networks"
    author:
      - name: Stefano Salsano
      - name: Wim Henderickx
      - name: Andreas Kassler
    date: false
    seriesinfo:
      Internet-Draft: draft-salsano-nmrg-nmaa-benchmarking-framework-latest
  ANIGAMUT:
    title: "ANI-Gamut: Benchmarking Agent Reliability across the Gamut of Agent-Network Interface Abstractions"
    author:
      - name: Lorenzo Bracciale
      - name: Pierpaolo Loreti
      - name: Andrea Mayer
      - name: Stefano Salsano
      - name: Wim Henderickx
    date: 2026
    refcontent: "22nd International Conference on Network and Service Management (CNSM)"
  WILSON:
    title: "Probable Inference, the Law of Succession, and Statistical Inference"
    author:
      - name: Edwin B. Wilson
    date: 1927
    refcontent: "Journal of the American Statistical Association, 22(158)"
  CLOPPER-PEARSON:
    title: "The Use of Confidence or Fiducial Limits Illustrated in the Case of the Binomial"
    author:
      - name: C. J. Clopper
      - name: E. S. Pearson
    date: 1934
    refcontent: "Biometrika, 26(4)"
  MCNEMAR:
    title: "Note on the Sampling Error of the Difference between Correlated Proportions or Percentages"
    author:
      - name: Quinn McNemar
    date: 1947
    refcontent: "Psychometrika, 12(2)"
  RULE-OF-THREE:
    title: "If Nothing Goes Wrong, Is Everything All Right? Interpreting Zero Numerators"
    author:
      - name: James A. Hanley
      - name: Abby Lippman-Hand
    date: 1983
    refcontent: "JAMA, 249(13)"

--- abstract

This document specifies a benchmarking methodology for Network
Management AI Agent (NMAA) systems, software systems that receive an
operator requirement and act on a network to satisfy it using AI models.
It defines the test setup (system under test boundary, brownfield
substrate, scenario generator, out-of-band observation, fault
injection), the test procedures for comparing settings of one
controlled factor at a time, the metrics (functional success,
damage-free run rate, cost, recovery), the statistical treatment
required for near-zero damage probabilities, and the reporting
requirements. It complements the conceptual framework described in a
companion research document and existing work on task-oriented
metrics for AI agents in network operations.

--- middle

# Introduction {#intro}

Benchmarking documents in the tradition of {{RFC1242}} and {{RFC2544}}
give implementers and operators a common way to measure and compare
devices under controlled, reproducible conditions. Network Management
AI Agent (NMAA) systems, which configure and operate networks using AI
models, need the same discipline, with two differences. First, the
output of the system under test is a change to a network, and its most
important failure mode, collateral damage to services that were
already running, is only observable on a network that carries such
services. Second, the system under test is stochastic: the same input
can produce different actions, so results are proportions with
uncertainty, and the regime of interest (damage probabilities near
zero) requires appropriate statistics.

This document specifies how to run and report such benchmarks. The
underlying model, the controlled factors and the rationale are
described in {{FRAMEWORK}}. Task-oriented metrics for AI agents in
network operations are proposed in
{{I-D.contreras-bmwg-ai-agent-benchmarking}}; this document reuses them
where applicable and adds the test setup, the collateral-damage
metric, the paired procedures and the statistics. Terminology for the
system under test boundary follows
{{I-D.gaikwad-llm-benchmarking-terminology}}.

## Scope

In scope: configuration and change tasks performed by an NMAA system on
an emulated or laboratory network; comparison of settings of one
controlled factor (model, abstraction level of the agent-network
interface, agent mode, task class) under a matched budget. Existing
task sets, such as those of {{I-D.cui-nmrg-llm-benchmark}}, can be
run under this methodology once placed on a brownfield substrate.

Out of scope for this version: troubleshooting and root-cause tasks
(TODO: whether to include them), adversarial testing of agents, and
performance of the inference serving layer (covered by
{{I-D.gaikwad-llm-benchmarking-methodology}}).

As for the benchmarking documents of the former BMWG {{RFC6815}}, the procedures are intended for
isolated laboratory environments only.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

The terms NMAA system, Human-Agent Interface (HAI), Agent-Network
Interface (ANI), SUT boundary, brownfield, service instance, service
class, baseline invariant, collateral damage, task class, task variant,
controlled factor, stress parameter, run and campaign are defined in
{{FRAMEWORK}}. This document adds:

Cell:
: The set of runs sharing one setting of every controlled factor and
  stress parameter.

Paired runs:
: Runs of different settings of the factor under study on the same task
  variant (same request, same generated brownfield, same fault seed).

Outcome label:
: One of clean success, dangerous success, clean failure, destructive
  failure, assigned to each run ({{metrics}}).

TODO: align with the terms of {{I-D.contreras-bmwg-ai-agent-benchmarking}}
(Agent, Task, Episode, Tool Interaction, Success, Failure).


# Test Setup {#setup}

## SUT Boundary

The SUT is the NMAA system between the HAI and the ANI, including the
model(s), the orchestration logic and the ANI adapter. The substrate,
generator, fault injector and observer are outside the SUT. The tester
MUST declare the SUT boundary and MUST NOT compare results obtained at
different boundaries.

## Brownfield Substrate

TODO: minimum requirements: number of coexisting service classes,
shared resources between classes, independence of state density from
topology size; supported device types and heterogeneity.

## Scenario Generator and Oracle

TODO: seeded generation of the initial state from a manifest;
co-emitted baseline invariants; requirement that every pre-existing
service passes its invariants before a run; placement of the task
target in the populated context.

## Fault Injection

TODO: admissible primitives at node and link level, injected below the
ANI identically for every setting; seeded and replayable.

## Observation

Success and safety MUST be assessed out of band by the test
environment, through a channel that does not depend on the setting
under study.

## Budget

TODO: execution budget (model, system prompt, agent mode, step and token
caps, retry policy, observation interface) and development budget
(no setting-specific tuning) to be held constant.


# Test Procedures {#procedures}

TODO: one procedure per controlled factor, each with Objective, Setup
parameters, Procedure, Measurements and Reporting format, in the style
of {{RFC2544}}:

* Interface level comparison (ANI level varied).
* Model comparison (model varied).
* Agent mode comparison.
* Task class comparison.
* Stress sweeps (state density, fault rate) applied to any of the above.
* Replay of a campaign by a third party from the reported seeds.


# Metrics {#metrics}

## Functional Success

Definition: a run is a functional success if the requested service
passes all its task test cases at the end of the run.

TODO: hard (structural) and soft (semantic) success, aligned with
{{I-D.gaikwad-llm-benchmarking-profiles}}.

## Damage-Free Run Rate

Definition: a run is damage-free if, after the run, every baseline
invariant of every pre-existing service instance and every global
invariant still holds. The damage-free run rate of a cell is the
fraction of its runs that are damage-free.

Discussion: the unit of damage is the service instance; an instance
is damaged if any of its invariants fails, counted once. The per-run
outcome is binary because collateral damage is a zero-tolerance
property in operations. The magnitude of damage (fraction of damaged
instances per service class) is a secondary, diagnostic measure.

## Outcome Labels

Each run is labeled clean success (functional success, damage-free),
dangerous success (functional success, not damage-free), clean failure
(no functional success, damage-free) or destructive failure (neither).

## Cost

TODO: tokens, model calls, steps, wall-clock time; relation to the
efficiency metrics of {{I-D.contreras-bmwg-ai-agent-benchmarking}}.

## Recovery

TODO: fraction of fault-hit runs that end damage-free and functionally
correct.


# Statistical Treatment {#statistics}

TODO, outline:

* Proportions are reported with interval estimates valid near 0 and 1
  (Wilson {{WILSON}} or Clopper-Pearson {{CLOPPER-PEARSON}}); the normal
  approximation MUST NOT be used near 0.
* The number of runs per cell is sized from the claim: zero damaged runs
  out of S bound the damage probability at about 3/S at 95% confidence
  {{RULE-OF-THREE}}.
* Paired settings are compared with McNemar's test on discordant pairs
  {{MCNEMAR}}.
* Minimum numbers of runs for comparative claims.


# Reporting Requirements {#reporting}

TODO: SUT description (model identity and version, inference settings,
prompts, agent mode, ANI level and adapter version, guardrails); budget;
substrate and generator version; manifests and seeds; fault seeds;
per-cell counts, rates and intervals; outcome labels; enough
information for replay.


# Security Considerations {#security}

The procedures grant an AI agent the ability to change network
configuration. Tests MUST be run on isolated laboratory networks
{{RFC6815}}. Published results characterize a SUT on a given substrate
and budget and do not establish fitness for production.

TODO: adversarial inputs out of scope; benchmark gaming through
setting-specific tuning.


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Example: ANI-Gamut {#example}

TODO: mapping of the procedures onto the ANI-Gamut implementation
{{ANIGAMUT}}, and of its pilot results onto the reporting format.


# Acknowledgments
{:numbered="false"}

TODO acknowledge.
