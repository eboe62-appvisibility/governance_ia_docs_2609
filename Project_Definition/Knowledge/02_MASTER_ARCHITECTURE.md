==========================================
02_MASTER_ARCHITECTURE.md
Version: 2026.07
Status: DRAFT
Authority: High

Depends on:
    00_PROJECT_VISION.md
    01_KNOWLEDGE_MATURITY.md

How is the system built?
==========================================

==========================================
PURPOSE
==========================================

This document defines the overall architecture of the project.

Its purpose is to describe how the system components are organized, how they interact and what responsibilities each component assumes.

It does not define operational rules.
It does not contain academic knowledge.
It does not define procedures.

It is the architectural blueprint of the system.

==========================================
DESIGN PRINCIPLES
==========================================

The project is built according to the following principles:

- simplicity;
- modularity;
- reusability;
- traceability;
- scalability;
- AI-platform independence;
- continuous evolution;
- minimum necessary governance.

==========================================
SYSTEM LAYERS
==========================================

                     User
                       │
                       ▼
               AI_ENTRYPOINT
          (Bootstrap + Orchestrator)
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
 Project          Knowledge         AI Skills
 Definition        Provider
      │                │                │
      └────────────────┼────────────────┘
                       ▼
              Knowledge Repository
                       │
                       ▼
               Project Deliverables

==========================================
COMPONENTS
==========================================

1. PROJECT DEFINITION

Defines the identity of the project.
Expected to remain stable over time.

------------------------------------------
2. GOVERNANCE

Defines how AI assistants shall operate.
Does not contain academic knowledge.

------------------------------------------
3. KNOWLEDGE PROVIDER

Provides uniform access to the structured knowledge of the project.
Acts as the abstraction layer between AI assistants and the underlying Knowledge Graph implementation.
Multiple implementations may coexist.

Examples include:
- Understand Anything;
- Neo4j;
- RDF/OWL;
- future technologies.

------------------------------------------
4. KNOWLEDGE REPOSITORY

Contains the project's human-readable Knowledge Assets and supporting documentation.

It represents the physical repository used to store and organize information.

It is not the source of truth.

The Knowledge Base comprises all knowledge managed by the project.

The Knowledge Graph constitutes the canonical structured representation of that knowledge.

The repository is therefore one representation of the Knowledge Base, but not the Knowledge Base itself.

------------------------------------------
5. AI SKILLS

AI Skills are specialised capabilities activated according to the current task, context and available evidence.

A Skill represents domain expertise rather than a fixed execution role.

Skills do not contain permanent knowledge.

They operate using the Knowledge Repository together with the Knowledge Graph through the Knowledge Provider.

------------------------------------------
6. PROJECT DELIVERABLES

Outputs produced by the system.

Examples include:
- notes;
- concept maps;
- exercises;
- vocabulary collections;
- research projects;
- Bachelor's Thesis (TFG);
- presentations;
- reports.

==========================================
INFORMATION FLOW
==========================================

New Information
↓
Capture
↓
Classification
↓
Validation
↓
Integration into the Knowledge Base
↓
Knowledge Provider synchronization
↓
Relationship with existing Knowledge Assets
↓
Available to AI Assistants
↓
Reuse

==========================================
ARCHITECTURAL RESPONSIBILITIES
==========================================

PROJECT VISION

Defines why the project exists.

------------------------------------------
KNOWLEDGE MATURITY

Defines how project knowledge evolves.

------------------------------------------
MASTER ARCHITECTURE

Defines how the system is organized.

------------------------------------------
ONTOLOGY

Defines the conceptual domain.

------------------------------------------
SOURCE MODEL

Defines where knowledge originates.

------------------------------------------
EVIDENCE CONFIDENCE

Defines the confidence assigned to knowledge claims.

------------------------------------------
LEARNING MODEL

Defines how learning progresses.

------------------------------------------
GOVERNANCE

Defines how AI assistants shall behave.

==========================================
ROLE OF AI_ENTRYPOINT
==========================================

AI_ENTRYPOINT is the bootstrap component of the system.

It does not contain academic knowledge.
It does not implement business logic.
It does not perform analysis.
Its sole responsibility is to initialize the execution environment and transfer control to the appropriate components.

==========================================
FUTURE EVOLUTION
==========================================

The architecture is designed to progressively incorporate:

- Knowledge Graph technologies;
- semantic retrieval;
- persistent memory;
- multiple AI assistants;
- future language models;
- document automation.

These evolutions shall not require changes to the project's conceptual
architecture.
