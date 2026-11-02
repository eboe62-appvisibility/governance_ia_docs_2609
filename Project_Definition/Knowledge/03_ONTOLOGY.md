==========================================
03_ONTOLOGY.md
Project Ontology
Version: 1.0 (Draft)
Authority: High

Depends on:
    00_PROJECT_VISION.md
    01_KNOWLEDGE_MATURITY.md
    02_MASTER_ARCHITECTURE.md

¿Qué existe dentro del dominio?
==========================================

# PURPOSE

This document defines the conceptual ontology of the project.

Its objective is to describe the domain represented by the repository, independently of any implementation technology.

The ontology establishes:
- which entities may exist;
- which properties they may have;
- which relationships are valid;
- which semantic constraints shall be respected.

It does NOT describe:
- repository structure;
- software implementation;
- graph databases;
- AI models;
- storage technologies.


==========================================
DESIGN PRINCIPLES
==========================================

The ontology shall represent the academic domain.
The ontology shall remain independent from implementation technologies.
The ontology shall evolve only when the conceptual domain evolves.
Implementation changes SHALL NOT require ontology changes.
Every entity shall have a unique semantic meaning.
Duplicate concepts are forbidden.
Relationships shall always have explicit meaning.

==========================================
CORE CONCEPTS
==========================================

Everything represented inside the project is an Entity.
Entities are connected through Relationships.
Entities and Relationships together form the Knowledge Graph.
Every entity belongs to exactly one Entity Type.
Every relationship belongs to exactly one Relationship Type.

==========================================
ENTITY TYPES
==========================================

The ontology recognizes the following high-level entity families.

ACADEMIC
Degree
Course
Subject
Professor
Student
Research
TFG
Exam
Assignment

LANGUAGE
Language
Writing System
Character
Radical
Word
Expression
Grammar Pattern
Pronunciation
HSK Level
Translation

GEOGRAPHY
Country
Province
Municipality
City
District
Village
Mountain
River
Lake
Island
Region
Route
Location

HISTORY
Dynasty
Historical Period
Historical Event
Historical Figure
Battle
Treaty

POLITICS
Government
Institution
Law
Political Organization
Policy

ECONOMICS
Economic Sector
Company
Industry
Trade Route
Economic Indicator

ENGINEERING
Infrastructure
Bridge
Tunnel
Dam
Railway
Airport
Port
Power Plant
Project

CULTURE
Religion
Philosophy
Literature
Book
Author
Artwork
Festival
Tradition

SOCIETY
Ethnic Group
Population
Community
Social Phenomenon
Demographic Indicator

ECOLOGY
Ecosystem
Species
Conservation Status
Pollution Source
Environmental Policy

EDUCATION
Educational System
School
Examination
Curriculum
Student Population

PERSONAL INTERESTS
Mountaineering
Hiking
Megaprojects
Technology
Travel
Anthropology

KNOWLEDGE
Concept
Source
Reference
Document
Image
Map
Dataset
Observation
Decision
Vocabulary Item
Skill


==========================================
RELATIONSHIP TO ADR-0033 DOMAIN VOCABULARY
==========================================

Entity Types (above) and the `Domain` vocabulary defined in `docs/decisiones/ADR-0033-Taxonomia_ADRs.md` § Domain answer different questions and operate at different levels.

Entity Types classify individual entities within the Knowledge Graph (e.g. a specific bridge, a specific dynasty, a specific book).

`Domain` classifies an entire Knowledge Asset, Relationship Record, or ADR by its overall subject-matter area.

A Knowledge Asset's `Domain` is not required to match 1:1 with the Entity Types it may reference or eventually decompose into.

Some terms exist as both an Entity Type nested inside a family (e.g. `Infrastructure` inside `ENGINEERING`, `Literature` inside `CULTURE`) and as an independent `Domain` term. These are two distinct concepts sharing a name — disambiguate by context: Entity Type family membership versus a `Domain:` field value.

==========================================
ENTITY PROPERTIES
==========================================

Every entity may define one or more properties.

Typical properties include:
Identifier
Canonical Name
Alternative Names
Description
Language
Category
Creation Date
Revision Date
Geographical Coordinates
Historical Period
Authority Level
Evidence Level
Status
Tags
References
Notes


==========================================
RELATIONSHIP TYPES
==========================================

Relationships define semantic meaning.

Examples:
belongs_to
contains
located_in
part_of
built_by
created_by
written_by
studied_in
teaches
uses
references
derived_from
depends_on
related_to
translated_as
pronounced_as
precedes
follows
influences
conflicts_with
requires
performed_by
supports
connects_to

==========================================
ONTOLOGICAL RULES
==========================================

Relationships shall always connect compatible entities.

Examples

City

    located_in

Province

------------------------------------------
Province
    belongs_to
Country

------------------------------------------
Bridge
    located_in
City

------------------------------------------
Character
    belongs_to
Writing System

------------------------------------------
Word
    composed_of
Character

------------------------------------------
Research
    references
Source

------------------------------------------
TFG
    uses
Research

==========================================
SEMANTIC CONSTRAINTS
==========================================

Every entity shall have one canonical definition.
Relationships shall never duplicate existing semantics.
Circular relationships should be avoided unless they represent real-world dependencies.
Ambiguous entities shall be explicitly disambiguated.
Orphan entities should not exist.
Every entity should participate in at least one meaningful relationship.

==========================================
KNOWLEDGE EVOLUTION
==========================================

Entities may evolve.
Relationships may evolve.
Definitions may evolve.
Semantic identity shall never change.
If the meaning changes significantly, a new entity shall be created.

==========================================
SPECIAL CONSIDERATIONS
==========================================

The entity families and terms listed under ENTITY TYPES instantiate the ontology for the project's current domain (Chinese Studies) and the personal profile of its current user. In particular:

- HSK Level (under LANGUAGE) is the specific proficiency scale for Mandarin Chinese; a project about a different language should use that language's own equivalent scale.
- PERSONAL INTERESTS (Mountaineering, Hiking, Megaprojects, Technology, Travel, Anthropology) reflects the current user's own motivations, not a structural requirement of the ontology.

A future project of a similar nature should retain the general structure — entity families grouped by academic/thematic domain, plus one family for the user's personal interests — while replacing these specific instantiations with its own.

==========================================
AI RESPONSIBILITIES
==========================================

When processing information, AI assistants shall:

identify entities;
identify relationships;
detect duplicated entities;
detect semantic inconsistencies;
suggest missing relationships;
preserve canonical definitions;
respect ontological constraints.

==========================================
IMPLEMENTATION INDEPENDENCE
==========================================

This ontology is implementation independent.

It shall remain valid regardless of whether the Knowledge Graph is implemented using:

- Understand Anything
- Neo4j
- RDF / OWL
- Property Graphs
- GraphQL
- future technologies
Implementation technologies may evolve.

The ontology shall remain stable.

==========================================
ARCHITECTURAL RESPONSIBILITY
==========================================

This document defines the conceptual model of the project.

It does not define:
- governance;
- AI behaviour;
- repository structure;
- software architecture;
- implementation details.
Those responsibilities belong to their respective documents.
