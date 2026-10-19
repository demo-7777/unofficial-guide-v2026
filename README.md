# The Unofficial Guide — RAG Question-Answering System

The Unofficial Guide is a Python-based retrieval-augmented generation (RAG) application that answers questions about campus life using a collection of student life documents.

The system retrieves relevant document passages, applies a relevance threshold to filter unrelated questions, and generates answers grounded in the retrieved content. It also supports source attribution so users can identify the documents behind an answer.

## Key Features

- **Document processing:** Splits campus-life documents into paragraph-based chunks targeting approximately 500 characters.
- **Semantic retrieval:** Searches indexed document chunks for information relevant to a user's question.
- **Relevance filtering:** Uses a configurable distance threshold to reject questions outside the document collection.
- **Source attribution:** Supports citing source documents in generated answers; evaluation identified missing attribution in refusal responses.
- **Hybrid retrieval experiment:** Combines semantic similarity with BM25 keyword matching to evaluate whether retrieval quality improves.
- **Structured evaluation:** Measures answer retrieval, source attribution, out-of-scope rejection, chunk quality, and retrieval ranking.

## System Workflow

1. **Chunk documents:** Divide the source documents into passages while preserving paragraph boundaries.
2. **Index passages:** Prepare the chunks for semantic retrieval.
3. **Retrieve context:** Search for passages relevant to the user's question.
4. **Apply relevance filtering:** Reject queries when retrieved results fall outside the configured relevance threshold.
5. **Generate an answer:** Use retrieved passages to produce an answer with supporting source references when available.
6. **Evaluate results:** Test retrieval and response behavior against five defined criteria.

## Evaluation Results

The initial evaluation produced the following results per run across three runs:

| Criterion | Target | Observed result |
|---|---|---|
| Retrieved chunk contains the answer | 4/5 | 1/5 |
| Every answer names a source | 5/5 | 1/5 |
| Out-of-corpus questions are rejected | 4/5 | 5/5 |
| Sampled chunks contain sufficient context | 4/5 | 5/5 |
| Relevant source appears in the top three results | 5/5 | 2/5 |

The system successfully rejected all five out-of-scope questions and produced understandable document chunks in the sampled evaluation.

However, retrieval frequently returned passages that were related to the question's topic without containing the information needed to answer it. Source attribution also failed for several refusal responses.

To investigate the retrieval issue, I implemented an experimental hybrid search approach combining semantic similarity with BM25 keyword matching. This changed retrieval rankings but did not meaningfully improve the measured results.

### My Contributions

Developed as part of the CodePath Applied AI program using a provided starter repository.

- Modified the document chunking implementation to group complete paragraphs into passages targeting approximately 500 characters.
- Configured and tested a relevance threshold to distinguish campus-related questions from out-of-scope queries.
- Implemented an experimental hybrid retrieval approach combining semantic similarity with BM25 keyword matching.
- Defined evaluation questions, executed repeated tests, and analyzed failures in retrieval quality and source attribution.
- Compared retrieval behavior before and after introducing BM25, documenting that the change did not improve the measured results.

## Getting Started

See [RUNNING.md](RUNNING.md) for setup and execution instructions.

## Implementation and Evaluation Details

The sections below document the chunking strategy, sample outputs, relevance threshold, evaluation methodology, retrieval experiment, and remaining limitations.

---


## Chunking Strategy

**Chunk size: 500 characters**
**Overlap: No fixed overlap, chunks are grouped by paragraph boundaries instead.**

I chose this strategy because the campus_life documents are short posts, with most documents already fitting within a few hundred characters. The starter produced 88 documents and 88 chunks, which showed that the default 800-character splitter rarely split anything. I changed the strategy to group complete paragraphs together up to about 500 characters so each chunk stays focused while preserving complete ideas.

This produced 90 chunks instead of 88, and the sampled chunks were still understandable without needing neighboring chunks.

## Sample Chunks

======================================================================
Chunk 1  |  source: dining_the_ridgeway_cafe.txt#0  |  produced by: chunker.py::split_documents
======================================================================
The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00. The thing worth going for is the only place on campus with real espresso. The thing to know is that seating is tight; about 40 seats for a building of 900.

Hours are 7:00am to 4:00pm weekdays only. Costs declining balance only, no meal swipes.

======================================================================
Chunk 2  |  source: housing_morrow_house.txt#0  |  produced by: chunker.py::split_documents
======================================================================
Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.

Laundry costs $1.50 wash, $1.25 dry, coin or card. On noise: loud until about 1am on weekends, no enforced quiet hours.

## Sample Answer

**Question:**

"What are the wait times at The Ridgeway Café around 12:30?"

**Answer:**

"The wait times at The Ridgeway Café are 10 to 15 minutes at 12:30."

**Source:**

dining_the_ridgeway_cafe.txt

## Relevance Threshold

**My relevance cutoff:** 0.7

| Question | In corpus? | Best distance |
|---|---|---|
| What lunches do students mention ordering most often? | Yes | 0.6203 |
| What do students say about how difficult it is to find parking? | Yes | 0.5795 |
| What do students say about professors' office hour availability? | Yes | 0.5168 |
| What steps do students mention for changing roommates? | Yes | 0.5822 |
| What times do students say the gym is least crowded? | Yes | 0.5086 |
| What is the capital of Mongolia? | No | 0.8246 |
| How do I change the oil in a diesel engine? | No | 0.9340 |
| Who won the 1994 World Cup? | No | 0.8859 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| How do I write a for loop in Rust? | No | 0.8960 |

## How I Used AI

**1.**

I used AI to help review my five test questions and make them more specific and easier to check. The AI suggested clearer wording and more specific expected phrases. I then edited the questions myself so they matched the topics I wanted to test in the campus_life corpus.

**2.**

I used AI to help revise the chunking logic in chunker.py. It suggested grouping text by paragraph boundaries with a target size of about 500 characters instead of using the starter’s fixed 800-character windows. I added that approach, ran the index again, and checked five sample chunks to make sure they were still understandable on their own.

**3.**

I used AI to help review the test results and identify where the failures were happening in the pipeline. It suggested that Criteria 1 and 5 were mainly retrieval problems because the returned chunks often did not contain the specific information needed, while Criterion 2 was a generation problem because refusal responses did not name sources. I used that diagnosis to choose hybrid search with BM25 as my single improvement, then compared the before and after evaluation results to see if it helped.

---

## Run Log — Before

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 1 of 5  | 1 of 5 | 1 of 5 | MISSED |
| 2. Every answer names a source | 5 of 5 | 1 of 5 | 1 of 5 | 1 of 5 | MISSED |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks have enough context | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Relevant source appears in top 3 | 5 of 5 | 2 of 5 | 2 of 5 | 2 of 5 | MISSED |

Run evidence produced by: `run_eval.py::main`
Retrieval produced by: `store.py::search`
Chunks produced by: `chunker.py::split_documents`

Criterion 1: 

Question: What do students say about professors' office hour availability?

Best distance: 0.5168

I do not have enough information to answer this question.

Criterion 2:

I do not have enough information in the provided documents to answer what lunches students mention ordering most often.

Criterion 3: 

Question: What is the capital of Mongolia?
Best distance: 0.825
Gate: refused

Criterion 4:

Evidence from python app.py chunks -n 5

Chunk: dining_the_ridgeway_cafe.txt
Produced by: chunker.py::split_documents

The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00. The thing worth going for is the only place on campus with real espresso. The thing to know is that seating is tight; about 40 seats for a building of 900.

Hours are 7:00am to 4:00pm weekdays only. Costs declining balance only, no meal swipes.

Criterion 5:

Question: What do students say about how difficult it is to find parking?

1. admin_parking_permits.txt — distance 0.5795


## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer  | MISSED | Only 1 of 5 test questions had a retrieved chunk that clearly contained the answer, below the 4 of 5 target |
| 2 | Every answer names a source | MISSED | Several generated answers were refusals that did not name a source document, so the 5 of 5 target was not met. |
| 3 | Gate stops out-of-corpus questions | MET | The relevance gate refused all 5 out of scope questions, which exceeded the 4 of 5 target. |
| 4 | Sampled chunks have enough context | MET | I reviewed the five sampled chunks and all 5 were understandable without neighboring chunks, exceeding the 4 of 5 target. |
| 5 | Relevant source appears in top 3 | MISSED | A relevant source appeared in the top 3 for only 2 of 5 test questions, below the 5 of 5 target. |

## Diagnoses

### Criterion 1 — Retrieved chunks contain the answer

**Stage:** Retrieval

For four of the five test questions, the retrieved chunks did not contain enough information to answer the question. For example, the office-hours, roommate-change, and gym questions returned chunks that were related to campus life but did not contain the specific information being asked for. Because the answer was missing before generation began, the failure happened at retrieval.

### Criterion 2 — Every answer names a source

**Stage:** Generation

Several responses correctly refused to answer because the retrieved documents did not contain enough information, but those refusal responses did not name any source document. Since the retrieved source list existed but the final generated response omitted source names, this failure happened during generation.

### Criterion 5 — Relevant source appears in top 3

**Stage:** Retrieval

Relevant sources were not consistently ranked near the top. The parking question retrieved `admin_parking_permits.txt` as the top result, but several other questions returned unrelated or only loosely related documents in the top results. This suggests the semantic retrieval method was matching general topic similarity without consistently finding documents containing the exact information requested.


## The Improvement

**What I changed:**

I changed retrieval from semantic vector search alone to a hybrid approach that combines semantic similarity with BM25 keyword matching. The hybrid score gives more weight to semantic similarity while also considering exact keyword matches.

**Why I picked it:**

My diagnoses showed that Criteria 1 and 5 were mainly failing during retrieval. Several questions returned documents that were generally related to campus life but did not contain the specific information being requested, so I tested whether adding keyword matching would rank more useful documents higher.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 1 of 5 | 1 of 5 | 1 of 5 | MISSED |
| 2. Every answer names a source | 5 of 5 | 1 of 5 | 1 of 5 | 1 of 5 | MISSED |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks have enough context | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Relevant source appears in top 3 | 5 of 5 | 2 of 5 | 2 of 5 | 2 of 5 | MISSED |

**Did it help?**

The hybrid search did not meaningfully improve the measured results. Criteria 1 and 5 still missed their original targets, and the same four questions still did not retrieve enough information to produce supported answers. The change altered which documents were returned and their ranking, but it did not increase the number of test questions the system could answer correctly.

## Limitations and Future Improvements

Criteria 1 and 5 are still missed because retrieval does not consistently return chunks containing the specific information needed by the question. Adding BM25 changed the ranking of some documents, but it did not improve the overall test results. A next step would be to try a different retrieval weighting, increase top-k, or test a different chunking strategy.

Criterion 2 is still missed because refusal responses do not name a source document. A next step would be to change the generation prompt so that even refusal responses identify the retrieved sources. Only one retrieval modification was introduced during this experiment to allow a controlled before and after comparison.

## What I'd Do Differently

I would change Criterion 5 so that it measures whether the top three retrieved chunks actually contain the answer instead of only requiring a relevant source. During testing, some sources were related to the topic but still did not contain enough information to answer the question. Measuring whether the answer is present would make the criterion more precise.
