==========================================
05_EVIDENCE_CONFIDENCE.md
Evidence Evaluation Model
Version: 1.0 (Draft)
Authority: High

Depends on:
    00_PROJECT_VISION.md
    01_KNOWLEDGE_MATURITY.md
    02_MASTER_ARCHITECTURE.md
    03_ONTOLOGY.md
    04_SOURCE_MODEL.md

¿Con qué confianza aceptamos una afirmación?
==========================================

# PURPOSE

This document defines how evidence is evaluated before knowledge becomes part of the project.

Its objective is to determine the confidence that can be assigned to a statement based on the available evidence.

This document evaluates claims.

It does NOT evaluate sources.

Source evaluation is defined in
04_SOURCE_MODEL.md.

==========================================
RESPONSIBILITIES
==========================================

This document defines:

- evidence concepts;
- evidence levels;
- evidence assessment criteria;
- confidence assignment;
- evidence lifecycle;
- AI responsibilities.
It does NOT define:

- source quality;
- repository structure;
- ontology;
- AI governance.

==========================================
FUNDAMENTAL PRINCIPLES
==========================================

Evidence is evaluated independently of its presentation.
Confidence shall always be proportional to the available evidence.
Absence of evidence shall never be interpreted as evidence of absence.
Contradictory evidence shall be preserved until resolved.
Every important claim should indicate its confidence level.
Confidence may evolve as new evidence becomes available.

==========================================
EVIDENCE TYPES
==========================================

Typical evidence includes:

Direct observation
Official statistics
Academic research
Experimental results
Historical documentation
Field observations
Maps
Photographs
Audio recordings
Interviews
Independent confirmations
Reproducible calculations
Validated translations

==========================================
EVIDENCE DIMENSIONS
==========================================

Evidence should be evaluated using several independent dimensions.

Consistency
Agreement between independent evidence.

------------------------------------------
Independence
Evidence obtained through independent origins.

------------------------------------------
Reproducibility
Whether the result can be independently reproduced.

------------------------------------------
Completeness
Coverage of the investigated subject.

------------------------------------------
Corroboration
Support from multiple independent observations.

------------------------------------------
Context
Suitability for the current question.

------------------------------------------
Uncertainty
Known limitations affecting interpretation.


==========================================
CONFIDENCE LEVELS
==========================================

Level 0
Unverified
Initial hypothesis or idea.
No supporting evidence.

------------------------------------------
Level 1
Preliminary
Limited supporting evidence.
Further verification required.

------------------------------------------
Level 2
Supported
Evidence exists and appears consistent.
Independent verification recommended.

------------------------------------------
Level 3
Well Supported
Multiple independent evidence sources agree.
No significant unresolved contradictions.

------------------------------------------
Level 4
Established Knowledge
Strong and consistent evidence.
Widely accepted within the corresponding academic discipline.
May still evolve if future evidence justifies revision.


==========================================
CONTRADICTORY EVIDENCE
==========================================

Contradictory evidence shall never be discarded solely because it conflicts with existing knowledge.

Contradictions shall be:

identified;
documented;
linked to their supporting evidence;

reviewed when additional information becomes available.

The existence of disagreement shall be explicitly represented.

==========================================
EVIDENCE EVOLUTION
==========================================

Evidence is dynamic.

New observations may:
increase confidence;
reduce confidence;
confirm existing knowledge;
replace obsolete conclusions;
create alternative hypotheses.

Knowledge shall remain open to revision.

==========================================
EVIDENCE TRACEABILITY
==========================================

Every evidence element should preserve its origin.

Preferred chain:

Claim
↓
Supporting Evidence
↓
Referenced Sources
↓
Original Sources

The complete reasoning path should remain reconstructable whenever possible.

==========================================
AI RESPONSIBILITIES
==========================================

AI assistants shall:

evaluate available evidence;
distinguish evidence from opinion;
identify unsupported claims;
assign an explicit confidence level whenever appropriate;
preserve contradictory evidence;
recommend additional verification when confidence is low;
avoid presenting uncertain information as established knowledge.

==========================================
SPECIAL CONSIDERATIONS
==========================================

Historical subjects may legitimately contain multiple competing interpretations.
Linguistic interpretations may vary according to academic traditions.
Field observations shall always be identified as observations.
Personal experience may enrich understanding but shall not automatically be treated as established evidence.
Engineering case studies should distinguish measured facts from engineering interpretation.


==========================================
ARCHITECTURAL CONSTRAINTS
==========================================

Evidence evaluation shall remain independent from:

AI platform;
Knowledge Graph implementation;
database technology;
repository organization;

future software evolution.


==========================================
FUNDAMENTAL RULE
==========================================

Knowledge Assets should evolve towards higher confidence through continuous verification.
The objective is not to eliminate uncertainty.
The objective is to represent uncertainty honestly and consistently.
