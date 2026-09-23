# Case Study: Letting an AI Agent Near Production Data — Safely

**Rasmi Murugavel · Data Quality & AI Governance · VectorCognitive**
*Self-directed build. I defined the problem and governance approach and directed the build and documentation with Claude Code (AI-assisted development).*

---

## 1. Problem

Companies want AI working on their data, but two things block them.

**The data isn't ready.** Gartner found that 63% of organizations don't have, or aren't sure they have, the right data management practices for AI, and predicts that through 2026 organizations will abandon 60% of AI projects that lack AI-ready data. Missing values, duplicate records, broken table links, stale data and personal information in the wrong columns usually get found the expensive way: by an auditor, a regulator or a downstream failure, months later.

**Manual data quality review doesn't scale.** An analyst can audit a handful of tables a day, and rule-based tools only catch what someone remembered to write a rule for.

An AI agent could audit continuously. But in 15 years of enterprise data quality work, I learned the question leadership actually asks: *"How do I know it didn't do something it shouldn't have?"*

## 2. Risk

The design started from a list of what could go wrong if an AI agent is pointed at a production database:

| Risk | Impact |
|---|---|
| The agent writes, updates or deletes data | Data loss, corrupted records |
| Customer personal data is sent to an outside AI provider | Privacy breach, regulatory exposure |
| The agent reports findings it made up, or misses real ones | Wrong decisions, false confidence |
| No record of what the agent did | Findings can't be defended in an audit |
| A prompt or model change silently makes it worse | Quality degrades with no one noticing |

## 3. Controls

The design principle: **the AI decides what to check, but every safety rule lives in code it cannot override** — never in a prompt it could ignore.

| Risk | Control |
|---|---|
| Writes data | Read-only enforced twice: a `SELECT`-only database role, and a code-level check that rejects any other statement even if the role is misconfigured |
| Leaks personal data | PII detected by column name *and* by value pattern, then masked in the tool layer before any result reaches the model |
| Makes things up | Final output must match a strict schema; if the agent runs out of budget it must list what it didn't audit instead of guessing |
| No audit trail | Append-only log of every query, row count and duration, written before the agent sees the result |
| Silent regression | Every release is scored against test databases with known answers. Recall ≥ 85%, precision ≥ 70%, severity accuracy ≥ 75%, and **zero** PII leaks, or the release is blocked. |

The project follows a full enterprise lifecycle, the way I worked on enterprise releases: 10 business requirements → 32 functional and non-functional requirements → 59 test cases → a traceability matrix linking every requirement to a test.

## 4. Proof

**What testing found.** Running the system against a live database, not just reading the code, surfaced three real defects:

- **A SQL injection path (critical).** A crafted column name slipped a harmful command past the read-only check, because the statement still *looked* like a harmless `SELECT`. Fixed and regression-tested. Lesson: *looks read-only* and *is provably read-only* are different claims.
- **A silent failure under real permissions (critical).** Key detection returned nothing when run as the real least-privilege account, because the database only shows constraint details to a table's owner. It passed every test that wasn't run with production permissions.
- **A live-database-only failure (high).** A query parameter that only broke against a real connection.

**What's verified.** The read-only guard against 7 attack and allow cases, PII masking, the audit log, the release gate (a good run passes; a bad run with a PII leak fails on all four criteria), and live integration tests under the real least-privilege role.

**What's not done yet — stated openly.** An end-to-end evaluation against the live model, and UAT sign-off. Documenting gaps honestly is itself a governance control.

## 5. What this shows

- **AI-ready data is a data quality problem first.** The agent scores the same five dimensions I've used for years: completeness, validity, uniqueness, consistency, timeliness.
- **Governance is designed in, not added later.** Access, privacy, logging and evaluation were requirements before any code existed.
- **Proof beats claims.** The most important defects were found only by testing under real conditions — the same discipline that kept my enterprise releases clean.

**Repo:** [Data Quality & Governance Agent](../README.md) · **Contact:** [linkedin.com/in/mrasmi](https://www.linkedin.com/in/mrasmi)
