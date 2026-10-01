# CiPD Chatbot: Production Product Analysis

Audit date: 1 October 2026

## Executive recommendation

Build a **CiPD Knowledge and Discovery Assistant** for the live website.

The product should stay narrow:

1. answer verified questions about CiPD and iPD-CP from public sources;
2. treat deadlines, fees, grants, eligibility, and contacts as structured, versioned facts;
3. help visitors discover connections among projects, faculty, guest faculty, mentors, technologies, and events;
4. show the professor which visitor questions the website does not currently answer;
5. provide a separate authenticated search experience for approved internal content in a later production phase.

This is more useful than a general campus chatbot because it addresses CiPD's actual information structure and produces measurable improvements on the live website.

## Live website audit

### Public information inventory

The live website currently contains:

- a centre overview and five homepage FAQs;
- a detailed iPD-CP programme page covering curriculum, eligibility, fees, scholarships, fellowships, grants, dates, and application calls to action;
- ten published projects with categories, summaries, features, technologies, status, and student teams;
- six core faculty and twenty-seven guest faculty profiles;
- four published events, with one currently upcoming;
- blogs, seminar and webinar information;
- separate Connect and Share an Idea forms;
- older and newer programme brochures with cohort-specific details.

The content is valuable, but users must already know which page contains the answer.

### Observed problems the chatbot can solve

#### 1. Current facts require date and cohort awareness

The current iPD-CP page presents the January 2027 cohort and the regular application deadline of 30 October 2026. The associated event description also mentions the completed early deadline of 27 September and the final window of 30 November. A simple semantic search could retrieve the early date without explaining that it has passed.

Programme brochures from different cohorts also contain different application fees, scholarship rules, payment terms, and support amounts. Retrieval based only on textual similarity could combine incompatible years.

**Required response:** store important facts with `cohort`, `valid_from`, `valid_until`, `status`, and `source_chunk_id`. Resolve them against the current date before generation.

#### 2. Discovery is divided across separate pages

Projects can be searched and filtered, but the public site does not provide one search across projects, people, technologies, and events. Faculty and guest faculty are presented as lists, while project pages contain their own technology and team information.

**Required response:** use shared entity metadata and relationship records so a visitor can ask:

- “Who works on embedded systems?”
- “Show health-tech projects using sensors.”
- “Which faculty or guest faculty are relevant to product design?”
- “Which events relate to IoT or hardware prototyping?”
- “I have an agriculture sensing idea; which projects and people should I read about?”

#### 3. Published mentor data is not ready for mentor matching

The public mentors dataset currently has no published records, and all ten project records have empty mentor relationships. The assistant must not invent mentor links from job titles or guest-faculty descriptions.

**Required response:** show mentor discovery only when approved mentor profiles and relationships are published. Until then, return faculty, guest faculty, projects, and events, and state that a verified mentor match is unavailable.

#### 4. People metadata is uneven

The public faculty dataset contains thirty-three people, but twenty-five records currently lack a biography and twenty-seven lack a LinkedIn link. Titles are useful, but they do not always provide enough evidence for precise expertise matching.

**Required response:** separate verified expertise tags from inferred text matches. Label an inferred match as “based on the published profile description,” and allow administrators to add approved expertise tags later.

#### 5. The FAQ surface is too small for the audience range

The homepage has five FAQs, while the website serves final-year students, graduates, founders, working professionals, IIIT Delhi members, companies, mentors, event hosts, and people proposing ideas.

**Required response:** collect privacy-safe question outcomes and build an FAQ gap report. The report should show high-frequency unanswered topics, poor-result searches, and repeated handoffs. It should recommend topics for the professor or authorised team to address; it must not publish content automatically.

#### 6. Connect and Idea are separate but their boundary is unclear

Connect supports event hosting, team enquiries, project ideas, partnerships, and other messages. The Idea page separately accepts concepts at idea, sketch, prototype, or shipped stages. A visitor may not know which route to use.

**Required response:** the chatbot should ask one short routing question and send the visitor to the correct existing form with a short explanation. It should never submit a form without the user.

## Target audiences and jobs to be done

### Prospective applicant outside IIIT Delhi

Needs to understand the programme, eligibility, curriculum, schedule, cost, support, deadlines, and application route without interpreting multiple pages and cohort documents.

Successful outcome: a cited answer using the current cohort, followed by the official application or contact route.

### IIIT Delhi student or researcher

Needs to discover relevant projects, technologies, faculty, guest experts, and events, or decide where to take an idea.

Successful outcome: a small set of relevant, explainable matches with direct profile or project links.

### Founder, company, event host, or external collaborator

Needs to understand CiPD capabilities, identify relevant work or expertise, and reach the correct collaboration route.

Successful outcome: relevant public evidence followed by the Connect or Idea form.

### Professor or authorised CiPD team member

Needs to see which questions are common, which are not answered, which sources were used, and where published metadata is too weak for reliable answers.

Successful outcome: an FAQ gap and content-quality report based on anonymous usage data.

## Product scope

### Core module 1: governed content database

Use PostgreSQL and pgvector for documents, versions, chunks, embeddings, structured facts, question outcomes, visitor events, and reviewed drafts.

Every document and chunk carries one of three visibility labels:

- `public`: available to the website assistant;
- `internal`: available only to authenticated, authorised users;
- `confidential`: excluded from ordinary assistant retrieval and protected by a default-deny database policy.

The public assistant always applies `visibility = 'public'` before vector or keyword search.

### Core module 2: validated ingestion

Ingest approved CiPD pages and brochures. Split documents at section boundaries, preserve heading paths and page numbers, and link every chunk to an immutable source version.

Validate extracted title, author, year, cohort, page type, publication date, and visibility against a fixed schema. Invalid or low-confidence extractions enter a review queue instead of the searchable index.

### Core module 3: Ask CiPD

Answer from retrieved public paragraphs and structured facts. Put citations beside supported claims. For important facts, show the source and last checked date.

If the sources are absent, stale, or insufficient, say so and provide the relevant official route. Do not guess admissions decisions, scholarship eligibility, project availability, or mentor relationships.

### Selected addition 1: structured important facts

Create typed records for:

- cohort start and end dates;
- application windows and their current status;
- programme and application fees;
- scholarship rules;
- fellowships and grants;
- eligibility categories;
- official email, phone, location, application, Connect, and Idea links.

Each value must link to its supporting source chunk and cohort. Structured facts take priority over unstructured retrieval for these questions.

### Selected addition 2: entity discovery

Model the following public entities:

- `project`;
- `person` with a role such as core faculty, guest faculty, or approved mentor;
- `technology`;
- `category`;
- `event`;
- `programme_topic`.

Store explicit relationships such as:

- project uses technology;
- project belongs to category;
- person has verified expertise;
- person mentors project;
- event covers technology or programme topic;
- person appears at event.

Begin with PostgreSQL relationship tables. A separate graph database is unnecessary for the current data volume.

### Selected addition 3: FAQ gap and visitor-question analytics

Log anonymous, minimal events:

- question topic and audience choice;
- answered, refused, or handed off outcome;
- cited source IDs;
- retrieval score band;
- latency;
- optional helpful or not-helpful feedback.

Do not store passwords, application submissions, private form content, or unnecessary personal identifiers. Define a retention period for raw question text and retain aggregated metrics longer.

The dashboard should answer:

- Which public questions are most common?
- Which topics produce no reliable answer?
- Which pages resolve questions successfully?
- Which source records are stale, sparse, or never retrieved?
- Which visitor groups require different starting prompts?

### Selected addition 4: role-aware internal staff search

Treat this as a separate authenticated interface, not a mode the public user can activate. Apply PostgreSQL roles and row-level security before retrieval.

Release it only after CiPD defines:

- who may access internal documents;
- which source collections are approved;
- retention and audit requirements;
- how access is removed when a role changes.

## User experience design

### Entry experience

Do not force account creation or a long questionnaire. Display three optional starting paths:

- **Explore the programme**
- **Find projects and people**
- **Collaborate or share an idea**

Below them, keep a normal question box so experienced visitors can type immediately.

### Answer layout

Use the same predictable structure:

1. direct answer in one or two sentences;
2. important current fact card when the question concerns dates, cost, support, eligibility, or contacts;
3. two to four supporting details;
4. inline source citations and last checked date;
5. one relevant next action.

### Discovery result layout

Return no more than three initial matches. Each match should include:

- entity type;
- name;
- why it matches;
- verified tags;
- relevant project, profile, or event link;
- an “inferred from profile text” label when the relationship is not explicit.

### Handoff rules

- Admissions or programme questions with insufficient evidence: official CiPD contact.
- Collaboration, event hosting, or partnership: Connect form.
- New product or research concept: Idea form.
- Internal-document request from a public session: explain that the public assistant cannot access internal content.

## Retrieval design

1. Classify the request as current fact, discovery, general explanation, navigation, or unsupported/private.
2. Apply the public access policy.
3. For current facts, query structured records first and retrieve the linked evidence.
4. For discovery, retrieve entities and relationships, then use semantic search on supporting descriptions.
5. For general explanation, use hybrid vector and PostgreSQL full-text retrieval.
6. Rerank the candidates and require a minimum evidence score.
7. Generate only from selected evidence and attach sentence-level citations.
8. Refuse or hand off when the evidence threshold is not met.

For the current corpus size, begin with exact pgvector search plus PostgreSQL full-text search. Introduce HNSW only after measurement shows a latency need.

## Evaluation plan

Create a professor-reviewed benchmark of at least 180 questions:

| Segment | Minimum questions |
|---|---:|
| Programme, eligibility, curriculum, and application | 40 |
| Deadlines, fees, scholarships, fellowships, and grants | 35 |
| Projects, technologies, categories, and teams | 35 |
| Faculty, guest faculty, mentors, and expertise | 30 |
| Events, collaboration, Idea, Connect, and contacts | 20 |
| Ambiguous, outdated, private, adversarial, and unanswerable | 20 |

Include questions from college members and outside visitors, short keyword queries, full natural-language questions, spelling mistakes, and follow-up questions.

### Production success measures

- 100% correct cohort selection for reviewed deadline, fee, support, and eligibility questions;
- at least 95% citation correctness and citation coverage;
- zero public retrieval of internal or confidential chunks;
- at least 85% successful top-three discovery matches on professor-reviewed cases;
- at least 90% correct routing to Apply, Connect, Idea, event, project, or contact destinations;
- complete common answers within five seconds at the 95th percentile;
- measurable reduction in repeated unanswered question clusters after website content is improved.

## Demonstration scenarios

### Scenario 1: current application information

Question: “Can I still apply for the January 2027 cohort and what will it cost?”

The assistant resolves today's date against structured application windows, identifies the regular round, provides the current fee and support information, cites the current cohort sources, and links to Apply.

### Scenario 2: external collaborator discovery

Question: “We work on agriculture sensing. Which CiPD work and people should we explore?”

The assistant returns relevant Agri Tech and IoT projects, public people matches only when supported by profile evidence, related events when available, and the Connect route.

### Scenario 3: IIIT Delhi student discovery

Question: “I want to build an embedded healthcare device. What should I look at?”

The assistant connects Health Tech projects, sensor and embedded-system technologies, relevant public faculty or guest profiles, and programme or event resources.

### Scenario 4: missing mentor evidence

Question: “Who is the verified mentor for ChillPod?”

Because the published project currently has no mentor relationship, the assistant says it cannot verify a mentor and points to the project page or official contact. It does not infer one.

### Scenario 5: professor's FAQ gap view

The dashboard shows that many users ask about application documents, attendance requirements, or project participation but the indexed public sources do not contain verified answers. The professor receives an evidence-based content backlog rather than automatically generated public claims.

## Explicit non-goals

- a general IIIT Delhi campus assistant;
- admission, scholarship, grant, or funding decisions;
- autonomous form submission;
- automatic publishing to the live website;
- unrestricted web search as an answer source;
- inferred private contact details;
- mentor recommendations without published mentor evidence;
- indexing application submissions, private emails, or credentials.

## Delivery sequence

### Release 1: verified public assistant

- public-page and brochure ingestion;
- versioned PostgreSQL and pgvector store;
- structured important facts;
- cited Ask CiPD responses;
- refusal and routing to existing public forms;
- reviewed evaluation set.

### Release 2: CiPD discovery and analytics

- project, person, technology, category, and event relationships;
- public discovery results with match explanations;
- anonymous question outcomes and helpfulness feedback;
- professor-facing FAQ gap report.

### Release 3: governed internal search

- authenticated internal interface;
- approved internal collections;
- database-enforced role policies;
- audit and retention controls.

## Sources reviewed

- [CiPD homepage](https://cipd.iiitd.ac.in/)
- [iPD-CP programme](https://cipd.iiitd.ac.in/ipd-cp)
- [Projects](https://cipd.iiitd.ac.in/projects)
- [Faculty and guest faculty](https://cipd.iiitd.ac.in/faculty)
- [Events](https://cipd.iiitd.ac.in/events)
- [Blogs](https://cipd.iiitd.ac.in/blogs)
- [Connect](https://cipd.iiitd.ac.in/connect)
- [Share an Idea](https://cipd.iiitd.ac.in/ideas)
- [January–June 2026 brochure](https://cipd.iiitd.ac.in/wp-content/uploads/2025/12/Brochure_iPD-CP-2026-3.pdf)
- [Earlier iPD-CP brochure](https://cipd.iiitd.ac.in/wp-content/uploads/2025/04/iPD-CP-Brochure_V2.pdf)

