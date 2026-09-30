# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This answers questions about campus life at a university, using a corpus of 88
short posts written by students — course workloads and exam formats, what each
residence hall is actually like, dining hall wait times, and administrative
deadlines like add/drop and pass/fail. Each post covers one topic in a few
sentences, in the voice of someone who took the course or lived in the building.

It handles questions with a specific factual answer: "What is the expected
workload for ECON 101?", "Which study rooms have whiteboards?", "In Aldridge
Hall, which floors are quiet and how many washers are there?" — including ones
whose answer is split across two posts. It will not answer "which dorm is
best", because the documents don't contain that answer and neither does the
system.

Questions the corpus doesn't cover are refused before the model is ever called.
A relevance check in `gate.py` compares the closest retrieved chunk against a
cutoff of 0.55 and stops there if nothing came back close enough, so asking it
about diesel engines gets "I don't have enough information about that" rather
than a confident guess.

## Chunking Strategy

**Chunk size:** 800
**Overlap:** 0

I measured all 88 documents in `campus_life` before deciding anything: shortest
178 characters, median 309, longest 549. Not one reaches 800. They are short
forum-style posts, one topic each — `course_biol_160_workload.txt` is four
sentences about how many hours BIOL 160 takes, and nothing else.

That measurement decided it for me: the right number of chunks per document
here is one. Splitting a 309-character post in half would separate a claim from
the sentence that qualifies it, and the starter's 800-character window already
leaves these documents whole, so there was nothing to fix. I set overlap to 0
for the same reason — overlap exists to rescue sentences cut at arbitrary
positions, and nothing here is being cut.

**I changed my mind once, and it cost me.** I wrote `split_documents` to split
on `##` Markdown headings, because I had been reading the `city_guides` corpus,
whose 14 documents are sectioned guides. Then I switched to `campus_life` and
did not re-check the assumption. Zero of its 88 documents contain a `##`
heading, so `re.split(r"(?m)^(?=## )", doc.text)` finds no split point and
returns the whole document. I verified this by running both functions over the
same corpus and comparing every chunk: `split_documents` and `fallback_split`
produce byte-identical output, 88 chunks averaging 317 characters.

So my Milestone 3 chunker is, on this corpus, a no-op. The output is correct —
one whole post per chunk is what these documents want — but I got there by
accident, not by design. What I actually learned is that a chunking strategy
is a claim about the documents, and mine was a claim about a different corpus.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

======================================================================
Chunk 1  |  source: admin_add_drop_deadline.txt#0  |  produced by: chunker.py::split_documents | function: def split_documents
======================================================================
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

======================================================================
Chunk 2  |  source: course_biol_160.txt#0  |  produced by: chunker.py::split_documents | function: def split_documents
======================================================================
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.

======================================================================
Chunk 3  |  source: course_hist_118_workload.txt#0  |  produced by: chunker.py::split_documents | function: def split_documents
======================================================================
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.

======================================================================
Chunk 4  |  source: dining_pellew_dining_hall_followup.txt#0  |  produced by: chunker.py::split_documents | function: def split_documents
======================================================================
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.

======================================================================
Chunk 5  |  source: housing_innisfree_hall.txt#0  |  produced by: chunker.py::split_documents | function: def split_documents
======================================================================
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** Which course is the heaviest first-year course by reputation and how many exams does it have per semester?

**Answer:**

Based on course_biol_160_workload.txt and course_biol_160.txt, BIOL 160
Cell Biology is the heaviest first-year course by reputation. According to
course_biol_160.txt, it has four unit tests and a cumulative final per
semester.

Sources retrieved: admin_graduation_requirements.txt, admin_pass_fail_option.txt,
course_biol_160.txt, course_biol_160_workload.txt, course_cs_340.txt

Produced by `app.py::ask_pipeline`, which calls `generate.py::answer_from_chunks`.
Best distance 0.4844, under the 0.55 cutoff.

``
```

**My relevance cutoff:** 0.55

I ran all ten questions through `app.py retrieve`, which reports distances
without spending a model call, and recorded the best distance for each.

The two groups don't overlap and they don't even come close. The worst
in-corpus question landed at 0.4844; the closest out-of-scope question at
0.8232. That leaves 0.3388 of empty space with nothing in it.

I moved the cutoff down from the starter's 0.6 rather than up. Both directions
pass all ten, so the ten numbers alone don't decide it — what decides it is
which failure I'd rather have. A confident wrong answer is harder to notice
than a refusal, so I'd rather the system refuse a question it could have
answered than answer one it couldn't.

I first tried 0.5 and backed off. It clears 0.4844 by only 0.0156, and I know
that number moves: adding a single `?` to my ECON 101 question shifted its
distance from 0.3504 to 0.3438. One punctuation mark is worth 0.0066, so 0.5
gives me about two punctuation marks of room. 0.4844 is the worst distance
among the five questions I happened to write, not the worst a real question
could produce. 0.55 keeps 0.0656 — roughly ten times the punctuation effect —
and still refuses all five out-of-scope questions by a margin of 0.27.

| Question | In corpus? | Best distance |
|---|---|---|
| What is the expected workload for ECON101? | Yes | 0.3438 |
| What type of clothing is recommended during winter, and by what time are the paths cleared on weekdays? | Yes | 0.3617 |
| For students need whiteboards, which study rooms should they go? | Yes | 0.3656 |
| In Aldridge hall, which floors are quiet floors and how many washers and dryers does it have? | Yes | 0.4139 |
| Which course is the heaviest first-year course by reputation and how many exams does it have per semester? | Yes | 0.4844 |
| What is the capital of Mongolia? | No | 0.8232 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| Who won the 1994 World Cup? | No | 0.8859 |
| How do I write a for loop in Rust? | No | 0.8960 |
| How do I change the oil in a diesel engine? | No | 0.9340 |


## How I Used AI

**1.** I asked Claude to help me test a threshold value before committing to it,
and it gave me `python app.py retrieve "..." --threshold 0.55`. That failed with
`error: unrecognized arguments: --threshold 0.55`. I checked the table in
`RUNNING.md` and `--threshold` is only wired to `ask`, not `retrieve` —
`cmd_retrieve` in `app.py` calls `gate.check(results)` with no threshold
argument, so it always reads `config.THRESHOLD`. I edited `config.py` directly
instead. I also realised the test wasn't needed: the gate is `best < threshold`,
and I already knew the best distance was 0.4844.

**2.** I asked why `course_cs_210.txt`, a Data Structures course, came back as
the second result for my question about which study rooms have whiteboards.
Claude said it was because my question contained "210" and collided with the
course number. My question doesn't contain "210" — it's "For students need
whiteboards, which study rooms should they go?", with no number in it. When I
opened the file, it had no "whiteboard" and no "room" in it either. The real
answer was that nothing pulled it in: only one of my 88 documents is about study
rooms, and retrieval had to return five, so it padded the list with whatever was
least far. I now read 0.56 as "not related" rather than "somewhat related".

*Unit 2:*

**3.** My first run had 4 of 5 questions marked "fail" by `scorer.py`, but the
answers looked right to me. I asked Claude to look at the pattern across the
failures. It pointed out that the four failures were exactly the questions whose
`expects` contains a `;` or a number, and that `judge()` looks for the whole
string word for word. It also found that `(answer or "".lower())` lowercases the
empty string, not the answer. I checked by reading all 15 answers against my
`expects` text, and every answer was correct. So I scored criteria 1 and 2 by
reading the answers and retrieved sources myself, not from the scorer's
pass/fail column.

**4.** Claude recommended hybrid search and wrote `_fuse_with_bm25` and
`_tokenize` in `store.py`. I had never used BM25, so I asked it to explain the
code section by section, then tried `rank_bm25` on a three-document toy example
myself to see how the scores behave. Its prediction was that BM25 would cut the
noise in my retrieved chunks, and that was only partly right. The after run
showed it helped the one question with a course code (ECON101) and did nothing
for questions in everyday words. I wouldn't have known that without comparing
the retrieved sources question by question.

**5.** When the ECON101 answer in the after run added that the course is
"front-loaded", I didn't assume it was a hallucination or assume it was fine.
I opened `course_econ_101_workload.txt` and checked: the line is in the document.


<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Complete factual statement | 4 of 5 | 5/5 | 5/5 | 5/5 | MET | 
| 5. Multi-document retrieval | 2 of 2 (revised) | 2/2 | 2/2 | 2/2 | MET |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

Real output below is from run 1 of `results/run_2026-09-30_1620.md`, written
by `run_eval.py::main`. Retrieval is `store.py::search` over chunks from
`chunker.py::split_documents` (top-k 5, cutoff 0.55).

**Criterion 1 — retrieved chunk contains the answer** (`store.py::search`)

| Question | Source that holds the answer | In retrieved sources? |
|---|---|---|
| ECON101 workload | course_econ_101_workload.txt | yes |
| Whiteboard study rooms | study_group_rooms.txt | yes |
| Winter clothing + path clearing | winter_gear.txt | yes |
| Heaviest first-year course + exams | course_biol_160.txt, course_biol_160_workload.txt | yes |
| Aldridge quiet floors + laundry | housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt | yes |

```
Q: What is the expected workload for ECON101
- Best distance: 0.3504 (passed the gate)
- Sources retrieved: course_biol_160_workload.txt, course_cs_210_workload.txt, course_econ_101_workload.txt, course_engl_205_workload.txt, course_phys_130_workload.txt
```

**Criterion 2 — every answer names a source** (`generate.py::answer_from_chunks`, called from `run_eval.py::run_once`)

```
The expected workload for ECON 101 Introduction to Economics is 4 hours a week outside class (course_econ_101_workload.txt).
```
```
Students needing whiteboards should go to Rooms 210 and 211, as they have whiteboards that actually erase. (Source: study_group_rooms.txt)
```
```
Layers matter more than a heavy coat during the winter, and the paths are cleared by 7am on weekdays. This information comes from the document `winter_gear.txt`.
```
```
BIOL 160 Cell Biology is the heaviest first-year course by reputation, and it has four unit tests and a cumulative final per semester (from course_biol_160_workload.txt and course_biol_160.txt).
```
```
In Aldridge Hall, floors 3 and 4 are quiet floors, and the building has eight washers and six dryers (source: `housing_aldridge_hall.txt`, `housing_aldridge_hall_laundry.txt`).
```

**Criterion 3 — gate stops out-of-corpus questions** (`run_eval.py::check_out_of_scope`, cutoff 0.55)

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.934 | refused |
| Who won the 1994 World Cup? | 0.886 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.844 | refused |
| How do I write a for loop in Rust? | 0.896 | refused |

**Criterion 5 — multi-document retrieval** (`store.py::search`)

```
Q: Which course is the heaviest first-year course by reputation and how many exams does it have per semester?
- Best distance: 0.4844 (passed the gate)
- Sources retrieved: admin_graduation_requirements.txt, admin_pass_fail_option.txt, course_biol_160.txt, course_biol_160_workload.txt, course_cs_340.txt
```
```
Q: In Aldridge hall, which floors are quiet floors and how many washers and dryers does it have?
- Best distance: 0.4139 (passed the gate)
- Sources retrieved: housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt, housing_aldridge_hall_noise.txt, housing_old_brewhouse_noise.txt, housing_tamsin_court_noise.txt
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | In all three runs, the file holding each answer appeared in the top-5 retrieved sources for 5 of 5 questions, above the 4-of-5 target. `scorer.py` marked four questions "fail", but it checks answer wording, not retrieval, so I judged this criterion from the sources lists. |
| 2 | Every answer names a source | MET | I read all 15 answers (5 questions × 3 runs) and every one names at least one `.txt` file. The citation format varied between runs (inline, "Source:", backticks), but a source was always named. |
| 3 | Gate stops out-of-corpus questions | MET | All 5 out-of-scope questions were refused. The closest one had a best distance of 0.825, far above the 0.55 cutoff, while the in-corpus questions ranged from 0.35 to 0.48, so this wasn't close. |
| 4 | Complete factual statement | MET | All 5 sampled chunks from `chunker.py::split_documents` stand on their own. Each post is short (about 317 characters on average) and fits in one 800-character chunk, so nothing gets split mid-fact. Chunking is deterministic, so the count is the same across runs. |
| 5 | Multi-document retrieval | MET | Both cross-document questions retrieved every document they needed in all three runs (2/2). The original "4 of 5" couldn't be measured with only 2 cross-document questions, so I revised it in criteria.md to "2 of 2". That's the strictest version I can check, and I kept the original line visible. |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

**Criterion 5 (multi-document retrieval): not a pipeline failure.** No stage
failed. In all three runs, `store.py::search` retrieved both documents each
cross-document question needed: `course_biol_160.txt` + `course_biol_160_workload.txt`
for the BIOL question, and `housing_aldridge_hall.txt` + `housing_aldridge_hall_laundry.txt`
for the Aldridge question. The problem was in my test set. The target said
"4 of 5 cross-document questions", but `questions.py` has only two, so it
couldn't be measured whatever the pipeline did. Because the criterion was
unmeasurable, not unmet (retrieval was 2/2), I revised it in `criteria.md` to
"2 of 2" and kept the original line visible. With the revision, it's MET, so I
have no misses.

**One real problem that no criterion caught: `scorer.py` fails correct answers.**
It marked 4 of 5 questions "fail" in every run, and when I read them all 15
answers were correct. The problem is in scoring, not generation. `judge()`
looks for the whole `expects` string word for word, so a multi-part expectation
like `"Layers; 7 A.M."` never matches "Layers ... 7am". It also lowercases `""`
instead of the answer. The pattern: the four that failed are exactly the four
where `expects` contains a `;` or a number. Only the ECON question, a single
phrase copied from the source, passed. That's one bug, not four separate failures.

**Were my targets set low? Yes.** Criteria 1–4 all passed on the first try with
room to spare, and the reason is the corpus, not the system:

- **Criterion 4 was guaranteed.** The 88 documents average 317 characters and
  the chunk size is 800, so `chunker.py::split_documents` never actually splits
  anything. Every chunk is a whole post. A chunk can't cut a fact in half if
  it's never cut.
- **Criterion 1 was loose.** "The answer is somewhere in the top 5" out of 88
  short, single-topic posts is easy to hit when each question names its subject
  (ECON101, Aldridge, whiteboards). The retrieved sets also carry noise: the
  winter question pulled in three laundry posts and a dining post, and the BIOL
  question pulled in two admin posts. None of that counted against the criterion.
- **Criterion 3 had a huge margin.** The in-corpus best distances were 0.35–0.48
  and the out-of-corpus ones were 0.83–0.93. My out-of-scope questions (Mongolia,
  diesel engines) are so unlike campus life that any cutoff between them would pass.

**What I'd tighten:**
- **Criterion 1 →** "For at least 4 of 5 questions, the **top-ranked** chunk
  comes from the document that holds the answer." This tests ranking, not just
  whether the answer made it into the top 5.
- **Criterion 3 →** replace the out-of-scope questions with near-misses the
  corpus doesn't cover but that sound like campus life, e.g. "What's the
  parking permit cost?" or "When does the gym open?". Then the gate has to tell
  a nearby topic from a covered one.

## The Improvement

**What I changed:** Hybrid search. `store.py::search` still ranks every chunk
by cosine distance, and `store.py::_fuse_with_bm25` now also ranks them by
BM25 keyword score and merges the two lists with Reciprocal Rank Fusion
(k = 60) before keeping the top 5. `store.py::_tokenize` splits letters from
digits, so "ECON101" matches "ECON 101". The distances aren't changed, so the
gate in `gate.py::check` compares against the same 0.55 cutoff as before. You
can switch it off with `config.HYBRID = False`.

**Why I picked it:**
The diagnosis found that retrieval carries noise: the winter question pulled in
three laundry posts and the BIOL question pulled in admin posts. My questions
are also built around exact names and codes (ECON101, Aldridge, BIOL 160,
Room 210), which is where keyword matching should help semantic search.
Chunking wasn't an option: every post already fits in one chunk.

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Complete factual statement | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Multi-document retrieval | 2 of 2 (revised) | 2/2 | 2/2 | 2/2 | MET |

Source: `results/run_2026-09-30_1929_after.md`, written by `run_eval.py::main`.

**Before and after, side by side** (each cell is Run 1 / Run 2 / Run 3)

| Criterion | Target | Before (semantic) | After (hybrid) | Change |
|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 · 5/5 · 5/5 | 5/5 · 5/5 · 5/5 | none |
| 2. Every answer names a source | 5 of 5 | 5/5 · 5/5 · 5/5 | 5/5 · 5/5 · 5/5 | none |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | none |
| 4. Complete factual statement | 4 of 5 | 5/5 | 5/5 | none (chunking unchanged) |
| 5. Multi-document retrieval | 2 of 2 (revised) | 2/2 | 2/2 | none |

**What did change: the retrieved sources** (`store.py::search`, top 5)

| Question | Dropped by hybrid | Added by hybrid | Better? |
|---|---|---|---|
| ECON101 workload | course_biol_160_workload.txt | course_econ_101_exams.txt | yes: now two ECON 101 posts instead of a BIOL one |
| Whiteboard study rooms | study_library_hours.txt | money_jobs.txt | no: swapped one irrelevant post for a less related one |
| Winter clothing + paths | dining_north_kitchen_followup.txt, three `*_laundry.txt` | dining_north_kitchen.txt, dining_the_atrium.txt, two `*_laundry.txt` | no: still four irrelevant posts out of five |
| Heaviest course + exams | admin_graduation_requirements.txt, course_cs_340.txt | course_math_220.txt, course_math_220_exams.txt | neutral: the words "course" and "exams" pulled in another course |
| Aldridge quiet floors + laundry | housing_old_brewhouse_noise.txt | housing_tamsin_court.txt | neutral |

Out-of-scope best distances (`run_eval.py::check_out_of_scope`):

| Question | Before | After |
|---|---|---|
| What is the capital of Mongolia? | 0.825 | 0.869 |
| How do I change the oil in a diesel engine? | 0.934 | 0.934 |
| Who won the 1994 World Cup? | 0.886 | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | 0.844 | 0.860 |
| How do I write a for loop in Rust? | 0.896 | 0.900 |

Real output from the after run, run 1 (`generate.py::answer_from_chunks`):

```
The expected workload for ECON 101 Introduction to Economics is 4 hours a week outside class, which is real time and front-loaded so that the first month is heavier than the rest. 

Source: `course_econ_101_workload.txt`
```
```
Layers matter more than a heavy coat, and the paths are cleared by 7:00 am on weekdays (winter_gear.txt).
```

**Did it help?**

Not measurably. None of the five criteria moved. They were at or near their
ceiling before the change, so this run could only show a regression, and there
wasn't one. That limit comes from how I set the targets, not from the fix.

At the level the diagnosis pointed to, the noise in the retrieved chunks,
hybrid search helped one question and didn't help the other four:

- **ECON101 improved.** This is the case BM25 is built for: the question has a
  course code, `_tokenize` matched "econ" + "101", and a second ECON 101 post
  replaced a BIOL one. The answer also got more complete: it now includes that
  the course is front-loaded. I checked that line against
  `course_econ_101_workload.txt` and it's in the document.
- **The winter question didn't improve.** It has no rare keywords, only common
  words like "winter", "clothing" and "paths". So BM25 had nothing distinctive
  to match, and it swapped one set of dining and laundry posts for another.
- **The BIOL question got a different kind of noise.** Common words like
  "course" and "exams" pulled in MATH 220 posts instead of admin posts.

The pattern: hybrid search helps when a question has a rare, exact term (a
course code), and does nothing when the question is phrased in everyday words.
Only one of my five questions has a term like that.

One side effect: three out-of-scope best distances went up (Mongolia
0.825 → 0.869). That happened because the fusion sometimes pushes the closest
chunk out of the top 5. It doesn't matter here, since they were all far above
the cutoff anyway. But for an in-corpus question near the 0.55 cutoff, the same
effect could make the gate refuse something it should answer.

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

**No criterion is missed after the fix.** Criterion 5's original "4 of 5"
couldn't be measured with only 2 cross-document questions, so I revised it to
"2 of 2" in `criteria.md` and left the original line visible. That doesn't mean
nothing is broken. These three things are, even though none of my criteria
measures them:

- **`scorer.py` fails correct answers.** It marked 4 of 5 questions "fail" in
  every run, and all 15 answers were correct when I read them. `judge()` looks
  for the whole `expects` string word for word, and it lowercases `""` instead
  of the answer. In the after run it even failed ECON101 run 2 because the
  answer said "outside **of** class".
  - *What I'd do:* split `expects` on `;`, normalize case and punctuation, and
    check each part on its own.
  - *Why I stopped:* the brief said one change, and fixing the scorer would
    have been a second one in the same round.
- **Retrieval noise on everyday-word questions.** The winter question still
  gets four irrelevant posts out of five. BM25 didn't help, because the
  question has no rare terms for it to match.
  - *What I'd do:* try a lower `TOP_K` (such as 3), or drop chunks whose cosine
    distance is much worse than the best one.
  - *Why I stopped:* none of my criteria measures this noise, so I couldn't
    have shown whether a fix worked. I'd need the tighter criterion 1 below first.
- **Hybrid fusion can push the closest chunk out of the top 5.** It raised
  three out-of-scope best distances (Mongolia 0.825 → 0.869). That's harmless
  for them, but an in-corpus question sitting near the 0.55 cutoff could get
  refused.
  - *What I'd do:* have the gate read the best distance from the full semantic
    ranking, before fusion.
  - *Why I stopped:* none of my five questions is near the cutoff (the worst is
    0.4844), so I have no test case that would show the problem.

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->

Four of my five criteria cleared their target on the first try with room to
spare, which says more about the targets than about the system. In the next
unit I would change:

- **Criterion 5:** write the questions first, then the target. I wrote "4 of 5
  cross-document questions" before writing any, ended up with two, and had to
  revise it.
- **Criterion 4:** this one could never fail. My posts average 317 characters
  and the chunk size is 800, so nothing is ever split. I'd replace it with
  something chunking can actually get wrong, or test it at a chunk size small
  enough to split posts.
- **Criterion 1:** "the answer is somewhere in the top 5" out of 88 short posts
  was too easy. I'd make it "the **top-ranked** chunk comes from the document
  that holds the answer". That tests ranking, which is where hybrid search
  actually changed anything.
- **Criterion 3:** swap the out-of-scope questions for near-misses that sound
  like campus life but aren't covered, like parking permits or gym hours.
  Mongolia and diesel engines were so far away (0.83+) that any cutoff would
  have passed.
- **Add a correctness criterion:** none of my five criteria asks whether the
  answer is *right*. The only thing measuring that was a broken scorer.
