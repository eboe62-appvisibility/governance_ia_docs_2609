==========================================
04_SOURCE_MODEL.md
Source Evaluation Model
Version: 1.0 (Draft)
Authority: High

Depends on:
    00_PROJECT_VISION.md
    01_KNOWLEDGE_MATURITY.md
    02_MASTER_ARCHITECTURE.md
    03_ONTOLOGY.md

¿De dónde procede la información?
==========================================

# PURPOSE

This document defines how knowledge sources are identified, classified, evaluated and referenced within the project.

Its purpose is not to determine whether a statement is true.

Its purpose is to establish a consistent model for evaluating the origin and quality of information before it becomes part of the project knowledge.

Evidence assessment is defined separately in
05_EVIDENCE_CONFIDENCE.md.


==========================================
RESPONSIBILITIES
==========================================

This document defines:

- source categories;
- source attributes;
- source evaluation criteria;
- source traceability;
- source lifecycle;
- AI responsibilities regarding sources.

It does NOT define:

- evidence levels;
- AI behaviour;
- ontology;
- repository structure.

==========================================
SOURCE PRINCIPLES
==========================================

Every knowledge element shall reference its origin whenever possible.
Every source shall be traceable.
Sources shall be evaluated individually.
Source category alone shall never determine reliability.
Original sources shall be preferred whenever reasonably available.
Derived Knowledge Assets shall preserve references to their originating sources.

==========================================
SOURCE CATEGORIES
==========================================

The ontology recognizes the following source categories.


PRIMARY SOURCES
Official documents
Government publications
Legal texts
Official statistics
Original literary works
Original historical documents
Direct field observations
Interviews
Research datasets
Academic lectures

SECONDARY SOURCES
Academic books
Peer-reviewed papers
University teaching material
Professional reports
Technical documentation
Conference proceedings

TERTIARY SOURCES
Encyclopedias
Reference manuals
Educational websites
Study guides
Review articles

DERIVED SOURCES
AI-generated summaries
Personal notes
Flashcards
Knowledge maps
Concept summaries
Research synthesis

==========================================
SOURCE ATTRIBUTES
==========================================

Each source may define:

Identifier
Title
Author
Publisher
Publication date
Language
Category
Document type
Version
Origin
Access date
Repository location
Citation
Notes


==========================================
SOURCE EVALUATION DIMENSIONS
==========================================

Each source shall be evaluated independently.
Typical evaluation dimensions include:
Authority
Expertise of the source author.

------------------------------------------
Evidence
Degree to which claims are supported.

------------------------------------------
Traceability
Ability to verify the original source.

------------------------------------------
Recency
Appropriateness of publication date for the subject.

------------------------------------------
Scope
Relevance to the current research question.

------------------------------------------
Completeness
Coverage of the represented topic.

------------------------------------------
Consistency
Agreement with other reliable sources.

==========================================
TRACEABILITY
==========================================

Knowledge shall always preserve traceability.

Preferred chain:

Knowledge
↓
Derived Information
↓
Referenced Source
↓
Original Source

The traceability chain should remain recoverable whenever possible.

==========================================
SOURCE RELATIONSHIPS
==========================================

Typical relationships include:

references
derived_from
summarizes
translates
quotes
extends
updates
contradicts
supports
supersedes

==========================================
SOURCE LIFECYCLE
==========================================

Sources evolve during the project.

Candidate Source
↓
Collected
↓
Classified
↓
Evaluated
↓
Referenced
↓
Archived

Sources are never deleted solely because they become obsolete.

Their status shall be updated instead.


==========================================
SPECIAL CONSIDERATIONS
==========================================

Chinese-language sources should be preferred when they provide information not available through equivalent translations.

Whenever possible, Chinese terminology shall be preserved together with its Pinyin and Spanish translation.

Personal observations shall always be identified as observations and shall never be confused with academic evidence.

Field observations obtained during travel or research activities (including Usera or future stays in China) constitute valid project sources when explicitly identified.

When a source has a PDF+Markdown pair, the PDF is a visual/graphical rendering of the transcribed Markdown, never the authoritative artifact for project knowledge. PDFs are intentionally excluded from version control (`*.pdf` in `.gitignore`); the Markdown transcription is the tracked, diffed, merged, and citable artifact (see ADR-1103).


==========================================
AI RESPONSIBILITIES
==========================================

AI assistants shall:
identify cited sources;
preserve traceability;
distinguish primary and derived knowledge;
recommend stronger sources when appropriate;
identify missing references;
detect duplicated references;
warn when source quality is uncertain;
avoid presenting derived knowledge as original evidence.

==========================================
ARCHITECTURAL CONSTRAINTS
==========================================

Source evaluation shall remain independent from implementation technology.
The model shall remain valid regardless of:
Knowledge Graph implementation;
AI platform;
database technology;
citation manager;
repository structure.

==========================================
FUNDAMENTAL RULE
==========================================

Knowledge without traceable sources may be useful for exploration, but shall not become consolidated project knowledge until its origin has been sufficiently identified and documented.
