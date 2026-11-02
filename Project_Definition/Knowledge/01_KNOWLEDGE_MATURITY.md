==========================================
01_KNOWLEDGE_MATURITY.md
Version: 2026.07
Status: DRAFT
Authority: High

Depends on:
    00_PROJECT_VISION.md

How does knowledge mature?
==========================================

==========================================
PURPOSE
==========================================

This document defines the conceptual model used to organize, relate, validate and evolve all knowledge generated throughout the project.

The purpose of the project is not to store documents.

Its purpose is to build and maintain a Knowledge Graph that represents the project's knowledge in a coherent, verifiable, interconnected and reusable manner.

Documents constitute a human-readable representation of that knowledge, but they are not its source of truth.

==========================================
PRINCIPLES
==========================================

1. Knowledge has priority over documents.
Documents are merely a storage medium.
The true value resides in the relationships between concepts.

------------------------------------------
2. Every new knowledge asset shall be reusable.
Isolated information shall never be intentionally created.

------------------------------------------
3. Knowledge shall continuously evolve.
Review, refinement and extension are integral parts of the model.

------------------------------------------
4. Information shall never be unnecessarily duplicated.
Each concept shall have a single canonical definition.

Other knowledge assets shall reference that definition instead of duplicating it.

------------------------------------------
5. Every significant statement shall indicate its evidence confidence level.
Facts, hypotheses and opinions shall be explicitly distinguished.

------------------------------------------
6. The Knowledge Graph constitutes the canonical representation of project knowledge.
Every structured knowledge asset shall be related to one or more graph nodes.
Documents represent views of the knowledge, not the knowledge itself.

==========================================
KNOWLEDGE ASSET
==========================================

The fundamental unit of the system is the Knowledge Asset.
Each asset represents a potential node within the Knowledge Graph.
Relationships between assets define the conceptual structure of the project.
A Knowledge Asset represents one unique knowledge entity.

Examples include:

- a Chinese word;
- a Chinese character;
- a city;
- a province;
- a mountain;
- a dam;
- a historical figure;
- a company;
- a university;
- a law;
- a philosophical concept;
- an event;
- a literary work;
- a document;
- an academic subject;
- a research project;
- a bibliographic source.

==========================================
KNOWLEDGE ASSET ATTRIBUTES
==========================================

When applicable, a Knowledge Asset may include:

Identifier
Canonical Name
Category
Description
Context
Timeline
Location
Sources
Evidence Confidence Level
Relationships
Review Status
Notes
Creation Date
Last Revision Date

==========================================
RELATIONSHIPS
==========================================

Knowledge gains value through relationships.

Examples:

City
    belongs_to
Province

Province
    belongs_to
Historical Region

Engineering Structure
    located_in
Province

Historical Figure
    participated_in
Historical Event

Company
    built
Infrastructure

Word
    appears_in
Text

Text
    belongs_to
Academic Subject

Academic Subject
    belongs_to
Course

Course
    belongs_to
Degree Programme

Bachelor's Thesis (TFG)
    uses
Research

Research
    references
Source

==========================================
KNOWLEDGE GRAPH
==========================================

The Knowledge Graph provides the structured representation of the project's knowledge.

Its objectives are:
- eliminate duplication;
- enable semantic search;
- discover relationships;
- connect academic subjects;
- connect research activities;
- support AI reasoning.

The Knowledge Graph shall remain independent of any specific AI assistant.

==========================================
PRIMARY KNOWLEDGE DOMAINS
==========================================

Language
History
Geography
Politics
Economics
Society
Engineering
Technology
Art
Literature
Philosophy
Religion
Culture
Business
Universities
Historical Figures
Infrastructure
Travel
Megaprojects
Mountaineering
Bachelor's Thesis (TFG)
Research
Bibliography

==========================================
KNOWLEDGE LIFECYCLE
==========================================

Knowledge evolves through several stages.

IDEA
↓
CAPTURE
↓
VERIFICATION
↓
CONSOLIDATION
↓
APPLICATION
↓
REVISION
↓
ARCHIVE (if no longer relevant)

==========================================
KNOWLEDGE MATURITY LEVELS
==========================================

Level 0
Initial idea awaiting verification.

------------------------------------------
Level 1
Knowledge supported by a single identified source.

------------------------------------------
Level 2
Knowledge corroborated by multiple independent sources.

------------------------------------------
Level 3
Knowledge consolidated and semantically connected to other Knowledge Assets.

------------------------------------------
Level 4
Knowledge successfully applied in learning activities, research projects or the Bachelor's Thesis (TFG).

==========================================
KNOWLEDGE SOURCES
==========================================

Knowledge may originate from:

- academic subjects;
- professors;
- books;
- scientific papers;
- official documentation;
- conversations with native speakers;
- academic stays abroad;
- field observations in a locally relevant community;
- personal projects;
- university assignments;
- independent research.

==========================================
REUSABILITY
==========================================

A single Knowledge Asset may simultaneously support:

- examination preparation;
- vocabulary acquisition;
- translation tasks;
- concept maps;
- presentations;
- the Bachelor's Thesis (TFG);
- future learning activities.

==========================================
SPECIAL CONSIDERATIONS
==========================================

In the current project (Chinese Studies), academic stays abroad take place in China, and the locally relevant community for field observations is Usera (the Chinese community in Madrid).

A future project of a similar nature should identify, analogously, its own equivalent immersion stays and local-community observation context.

==========================================
ROLE OF AI
==========================================

AI assistants do not merely generate responses.

Their primary responsibility is to preserve the consistency of the project's knowledge model.

To achieve this, AI assistants shall:
- detect duplicated knowledge;
- identify contradictions;
- suggest semantic relationships;
- recommend improvements;
- identify knowledge gaps;
- detect obsolete knowledge;
- recommend further research;
- preserve the overall consistency of the Knowledge Base.

==========================================
FUNDAMENTAL RULE
==========================================

Before creating new knowledge, AI assistants shall first attempt to relate it to the existing Knowledge Base.

Creating isolated knowledge shall be considered an exceptional situation.

The permanent objective is to increase both the density and the quality of the project's knowledge network.
