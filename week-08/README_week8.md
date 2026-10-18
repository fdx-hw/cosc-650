# Elden Ring Build & Lore Assistant

COSC 650, Week 8 capstone. Ask about Elden Ring weapons, bosses, talismans and spells. The assistant answers from game data instead of memory, uses tools for checks and arithmetic ("can I wield this with 20 Str and 12 Dex?", "how much heavier is X than Y?"), and says "I don't have that in the data" when the data doesn't cover the question.

**In 60 seconds:** four techniques from earlier weeks, wired into one pipeline and evaluated on 18 held-out cases with a second Claude model as judge. It got 15 of 18 right. Two of the three misses show a gap between the components: when retrieval misses a record, the model declines instead of using a tool that could have looked it up. I then tested two fixes on 12 new questions where retrieval missed: matching exact item names before retrieval took the system from 0 of 12 right to 12 of 12, and a prompt rule telling the model to try the tool first took it to 8 of 12. The third miss was partly the answer check being strict about wording. Details below and in the [research note](notes/week-08.txt).

## Architecture

```
question
   |
   v
[router: 4 nearest labeled questions -> Claude Haiku]          (Week 7)
   |                                    |
 "lookup"                             "tool"
   |                                    |
   v                                    v
[retrieve top-5 records]            [retrieve top-5 records]    (Week 5)
   |                                    |
   v                                    v
[grounded prompt -> answer]         [tool loop: get_weapon_stats,
   (Week 3)                          check_requirements,
   |                                 calculate (guarded)]       (Week 4)
   |                                    |
   +------------------+-----------------+
                      v
                   answer
                      v
   [LLM-as-judge, a different Claude model: correctness, grounding, calibration]
```

## Where each component came from

| Week | Technique | Role here | Why I chose it | What I changed |
|---|---|---|---|---|
| 5 | Retrieval: MiniLM embeddings + FAISS | finds the records a question is about | there are 738 records; the model should read the relevant ones, not recall them | one chunk per game record instead of fixed-size text chunks |
| 4 | Tool use: schemas, validator, `dispatch`, guarded code-runner, Claude tool loop | exact stat lookups, requirement checks, arithmetic | weights and requirement checks have exact answers; the model shouldn't do arithmetic in prose | guarded runner, validator and dispatcher copied in unchanged; two new tools (`get_weapon_stats`, `check_requirements`) |
| 7 | Retrieval-selected dynamic few-shot | the **router**: `lookup` or `tool` | cheap way to spend the tool loop only on questions that need it | applied to routing instead of ticket labels |
| 3 | Grounded prompt | answer only from the context and tool results, otherwise decline | makes "I don't have that" possible and judgeable | none |
| 6 | Dense + lexical re-ranker | tested as a fix for the failure below (helped, did not fully fix) | game names are rare tokens that word overlap catches and embeddings blur | reused unchanged |

## Results (one run)

Data: 738 records from the [Elden Ring Fan API](https://docs.eldenring.fanapis.com/) (307 weapons, 106 bosses, 87 talismans, 98 incantations, 71 sorceries, 69 shields). Generator, router and tool-user: Claude Haiku 4.5. Judge: `claude-sonnet-4-5-20250929`.

| what | result |
|---|---|
| end-to-end, 18 held-out cases, plain-code checks | **15/18 = 83.3%** [60.8, 94.2] |
| judge, correctness / grounding / calibration (1 to 5) | 4.33 / 4.89 / 4.33 |
| judge vs. plain-code check | agreed on all 18 (15 pass, 3 fail) |
| routing, 60 held-out questions | few-shot 100%, zero-shot 100%, kNN vote 85%, starter keywords 50% |
| unanswerable questions (should decline) | 3/3 |
| ablation: full / oracle route / oracle retrieval | 15 / 15 / 17 of 18 |
| retrieval, every needed record in top 5 (76 questions) | dense 86.8% to re-ranked 94.7% (p = 0.031) |

The few-shot router did no better than a zero-shot prompt in this run, so its benefit is not shown here.

## The failure I found

When retrieval misses a record, the tool path does not reach for the tool. "Could someone with 21 Str and 13 Dex swing the Arbalest?" retrieved five unrelated clubs and polearms, made no tool call, and answered "I don't have the Arbalest in the provided context data", even though `check_requirements` takes the weapon name and would have answered it. The ablation shows retrieval is the trigger: giving the assistant the correct records fixes it, and forcing the correct route does not. Full trace in the notebook's Part 3.

The third miss (case 7, the Godskin duo) turned out to be two things: a real retrieval error in the full system, and, under oracle retrieval, a correct answer ("1 Godskin Swaddling Cloth") that my check rejected because it wanted the data's wording ("x1").

## What I tried afterwards

Two fixes, both written after I had seen the failures, so the original 18 cases are not a fair test of them. The fair test is 12 new tool-path questions (different items, selected only because dense retrieval missed a named record, chosen before any model call).

| condition | 12 new questions right [95% CI] | original 18 right |
|---|---|---|
| baseline | 0/12 = 0% [0, 24] | 15/18 |
| Week 6 re-ranker | 5/12 = 42% [19, 68] | 17/18 |
| **exact-name linking** (add records whose full name appears in the question) | **12/12 = 100% [76, 100]** | 18/18 |
| tool-path rule (look a named weapon up with the tool before declining) | 8/12 = 67% [39, 86] | 16/18 |
| linking + rule | 12/12 | 18/18 |

Linking fixed 12 and broke none (sign test p < 0.001). The rule changed tool use when the record was missing from 1 of 12 to 9 of 12 with retrieval unchanged, which supports my guess that the "answer only from the context" rule was suppressing the tool. It is not a full fix: 4 of 12 stayed wrong. Caveats: these questions contain the full item name, which is the best case for exact matching, so misspellings and nicknames would still miss; and I did not measure latency or cost.

## Run it

1. Open `week8_capstone.ipynb` in Google Colab.
2. Add your key as a Colab secret named `ANTHROPIC_API_KEY`.
3. Run all cells. It fetches the data once into `data/raw/` (about a dozen requests to a free community API; please don't hammer it), then makes a few hundred model calls and takes a few minutes.
4. If the judge shows `[JUDGE-BLOCKED]`, change `JUDGE_MODEL` near the top to a Claude model your key can use. The rest still runs.

## Limits

- Eighteen cases is small; the interval on 15/18 runs from 61% to 94%. The fresh set that tests the fixes is 12 questions.
- The plain-code check wants the data's exact wording, so a correct paraphrase can fail it (this happened on case 7).
- The gold answers come from the same scraped records the assistant reads, so this measures fidelity to that data, not knowledge of the game.
- Questions are generated from templates containing the full item name, which flatters word-overlap retrieval.
- The judge is the same model family as the generator. The model I used as judge is scheduled for end-of-life on November 30, 2026.
- One run; I did not measure latency or cost.

## Files

- `week8_capstone.ipynb`: the notebook, with outputs
- `data/week8_results.json`: run summary

The data is scraped from fan sources, is not mine to redistribute, and the notebook re-fetches it.

*Fan project, not affiliated with or endorsed by FromSoftware or Bandai Namco. Elden Ring is their trademark.*
