# Learner Notes — Integration 9A: SPARQL Query Suite

---

## Q1 — Authors at NeurIPS

**Intent:** Find every distinct author who has at least one paper published at the NeurIPS venue.

**SPARQL forms used:** `SELECT DISTINCT`, `WHERE`, `;` property shorthand.

**Result:** 17 authors.
- author000
- author001
- author100
- author004
- author111
- *(12 more …)*

---

## Q2 — Papers per Topic

**Intent:** For each research topic in the graph, count how many papers are tagged with it, to see which topics dominate the dataset.

**SPARQL forms used:** `SELECT`, `WHERE`, `GROUP BY`, `COUNT(?paper) AS ?n`, `ORDER BY DESC`.

**Result:** 24 topics.
| Topic | Papers |
|---|---|
| topic_self-supervised | 5 |
| topic_summarization | 3 |
| topic_interpretability | 2 |
| topic_recommender-systems | 2 |
| topic_attention | 1 |
| *(19 more …)* | |

---

## Q3 — Canonical Coauthor Pairs

**Intent:** List every unordered pair of authors {a, b} who share at least one paper, with no duplicates — suitable for building a coauthorship network.

**SPARQL forms used:** `SELECT DISTINCT`, `WHERE`, `FILTER` (two conditions).

**Key design decisions:**
- `SELECT DISTINCT` collapses multiple shared papers down to one row per pair.
- `FILTER (str(?a) < str(?b))` enforces canonical ordering so each unordered pair appears exactly once (no (a,b) and (b,a) duplicates).

**Result:** 215 distinct coauthor pairs.
- author000 – author001
- author000 – author100
- author000 – author004
- author000 – author111
- author000 – author039
- *(210 more …)*

---

## Q4 — Papers with Optional DOI

**Intent:** List all papers alongside their DOI identifier, while keeping papers that have no DOI in the results (shown as unbound/null).

**SPARQL forms used:** `SELECT`, `WHERE`, `OPTIONAL`.

**Why OPTIONAL matters:** Placing `:doi` inside `OPTIONAL { }` performs a left outer join — all 80 papers appear regardless of whether they have a DOI triple. Moving `:doi` into the main `WHERE` block would silently drop every paper without a DOI.

**Result:** 80 rows (all papers).
| Paper | DOI |
|---|---|
| paper000 | 10.1000/p000 |
| paper001 | *(unbound)* |
| paper002 | *(unbound)* |
| paper003 | 10.1000/p003 |
| paper004 | *(unbound)* |
| *(75 more …)* | |

---

## Q5 — ASK: Any Author with > 10 Papers?

**Intent:** Return `true` if the dataset contains at least one author who has published more than 10 papers; otherwise return `false`.

**SPARQL forms used:** `ASK`, sub-`SELECT`, `GROUP BY`, `COUNT`, `HAVING`.

**Result:**
```
true
```
At least one author in the ~120-author dataset has more than 10 papers among the 80 in the graph.

---

## Q6 — CONSTRUCT: 2023 Paper–Author Graph

**Intent:** Build a new RDF mini-graph containing only the authorship triples for papers published in 2023 — useful for exporting a year-scoped slice of the data.

**SPARQL forms used:** `CONSTRUCT`, `WHERE`, `:year` literal match.

**Result:** 25 triples of the form `?paper :authoredBy ?author`.

Sample triples:
```
<paper042>  :authoredBy  <author003> .
<paper057>  :authoredBy  <author011> .
<paper071>  :authoredBy  <author000> .
```

---

## Q7 — Top 5 Most-Cited Papers

**Intent:** Identify the five papers with the highest citation counts using the `:citationCount` integer literal stored on each paper.

**SPARQL forms used:** `SELECT`, `WHERE`, `ORDER BY DESC(?cc)`, `LIMIT 5`.

**Important:** The dataset stores citations as a single integer literal per paper (`:citationCount`). There are **no** `:cites` edges in this dataset — aggregating over inverse edges returns zero rows.

**Result:** 5 papers.
| Paper | Citation Count |
|---|---|
| paper063 | 485 |
| paper043 | 475 |
| paper004 | 473 |
| paper007 | 470 |
| paper048 | 470 |

---

## Q8 — Authors Named "Hinton"

**Intent:** Find all author resources whose name is recorded as "Hinton" under either their primary SKOS label (`skos:prefLabel`) or an alternate label (`skos:altLabel`) — the SKOS disambiguation pattern from reading §7.

**SPARQL forms used:** `SELECT DISTINCT`, `WHERE`, `FILTER` with `||` (logical OR).

**Why both labels?** An author's canonical name might differ from "Hinton" (e.g. "Geoffrey Hinton") while "Hinton" appears as an `altLabel`, or vice versa. Checking only `prefLabel` would silently miss the other case — the autograder specifically tests for this.

**Result:** 2 authors.
- author000
- author007