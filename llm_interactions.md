# LLM interactions

**Tools used:** Claude Code (model: Claude Opus 5.5, by Anthropic), used as a coding assistant inside VS Code.

The exchanges below are in chronological order. Short prompts are quoted; long exchanges are faithful summaries. The AI never saw or suggested my ratings: I did the manual annotation (part A) alone, before looking at any model output.

---

## Understanding the assignment

**My prompt:** I shared the assignment PDF and the path to the lab notebook (`lab_annotation.ipynb`) and asked for help structuring the solution.

**Answer (summary):** The assistant read the brief and the lab notebook and listed what the lab code lacks for the assignment: the rating must also return toxic words as evidence, so the output limit and the parser must change; invalid answers must be counted; the tables need `annotator`, `prompt` and `context` columns; and the conversations need a quality check. It suggested a starting grid (gender × politics × age with a "center" value added), putting the conversation in the annotator's `context` in condition D, and testing several models on a few items first. It pointed out that `prepare_items.py` and `data/pool.csv` were missing from my folder.


## My working rules for the assistant

**My prompt:** "I don't want you to provide the whole code and do not execute code without me giving you permission."

From this point, the assistant did not run any code. It only read files and gave snippets.


## Colab GPU limit

**My prompt:** A screenshot of "Cannot connect to GPU backend" and the question of why it happened.

**Answer (summary):** Free Colab GPU limits depend on recent connection time and demand, not only on heavy use, and are not published. Suggested alternatives: run part B without a GPU, move parts C–E to Kaggle Notebooks (free T4), or use Colab Pay As You Go. Advised against a second Google account.

## Base code for parts B–F

**My prompt:** "Do not execute/import anything. As said before, I have finished the manual annotation part. I want you to provide the rest of the code here based on the lab, so I can have a basis and revise."

**Answer (summary):** The assistant first read the SynDisco source code on GitHub to find where the prompt is saved (`entry["prompt"]` holds the annotator's system prompt) and how turn order can be seeded. It then wrote base code, not run, for:
- **B:** a histogram and a boxplot of my ratings;
- **C:** a pool of conversation personas, shared context and instructions, a `problems()` check (empty, cut off, refusal, repeat) with up to 5 retries, saving to `generated.jsonl`;
- **the grid:** built with `itertools.product`, plus the baseline;
- **D and E:** fixed rating instructions, an `annotate()` function with the conversation in `context` (D) or not (E), a parser for `rating | words` that labels each answer `ok`, `no_rating`, `no_evidence` or `evidence_not_in_item`, a resumable `run()` that saves each row to CSV, a table of invalid answers and a check that D and E prompts differ only in the context;
- **F:** simple agreement (categorical and exact) for the four comparisons, a check for constant annotators, a heatmap, and a table of agreement by grid dimension.

It warned that the code was untested.

## Questions about the code

I asked the assistant to explain several parts, and checked each explanation against the code:
- **Part C step by step**
- **The difference between the pool (part C) and the grid (parts D, E)** and why they are not merged: pool personas may be guided, grid personas may only describe who the annotator is, and the grid must vary one controlled attribute at a time.
- how the grid size sets the number of model calls.
- **The generation loop,** the JSONL file, and where `generated.jsonl` is saved on Colab and Kaggle.

## Checking the code against the SynDisco documentation

**My prompt:** I shared the "Creating discussions" guide link and asked the assistant to modify the code if it did not match.

**Answer (summary):** The assistant read the discussion, annotation and experiment guides. The code already followed them: plain dict personas, shared context and instructions, `Actor`, `RandomTurnManager`, `Discussion`, `get_logs()`. It suggested two small changes: the documented key `personality_characteristics`, and a fallback to `annotator.get_system_prompt()` in case the saved prompt is empty. It explained why it kept `"OriginalPoster"` as the item's author and did not use `DiscussionExperiment`.

## My changes, reviewed

- **The D context sentence:** I changed it to the lab's wording, "Before you rate, read the conversation that people had about the comment:". The assistant confirmed it still keeps D and E identical except for the context.
- **The pool:** I rewrote it with my own 8 personas (education level, different jobs and personalities). The assistant noted a name clash with a later variable.
- **A larger grid:** I proposed gender (3) × politics (3) × religion (2) × age (3). The assistant calculated 54 personas and 2,200 calls, too many for the time and GPU available and too noisy with 20 items.
- **Removing the stances:** the assistant listed the trade-off: less influence from me, but a risk of conversations where everyone agrees. It recommended testing on 3 items.
- **My instruction text:** it suggested putting back "do not refuse", because my "offensive" and "rude" personas make refusals more likely.

## Running part C and fixing the conversations

- **Run time:** I ran part C, which took 7 minutes. The assistant had estimated 10–40 minutes.
- **Self-replies:** I noticed personas replying to their own earlier comments. The assistant explained that with 3 participants and 5 replies someone must speak twice, and the model does not know which earlier comments are its own. On my request it gave the full `make_conversation` using 5 participants and `QueueTurnManager`, so each person speaks once.
- **Second problem report:** I shared the 20 new conversations and reported that personas still used their own names, that replies said the same thing in different words, and that the personalities had little effect. The assistant identified the cause of the self-naming (the username inside the persona) and proposed:
  - removing the username from the prompt persona;
  - reintroducing stances, a different one per participant;
  - instructions that ban thanking, "I agree" openings and summarising;
  - `history_context_len=3`;
  - optionally, a larger model.

## GPU quota again and planning

**My prompts:** Colab refused the T4 again although it showed time remaining. I asked for help finishing tonight.

**Answer (summary):** The assistant explained that free Colab has no guaranteed quota, so switching to CPU does not reserve the GPU. It gave Kaggle set-up steps and a time plan for the remaining work.


## Final iterations of part C

- **Run with stances:** I shared the conversations generated with stances and asked whether to switch model. The assistant found that self-addressing and refusals were gone and the stances and personalities showed. The main problem left was later speakers copying earlier ones, which it listed by item. It recommended keeping Qwen 3B and making two cheap changes: a stricter `problems()` check (60% similarity, same three opening words) and `history_context_len=2` with an instruction against copying.
- **`ValueError`:** I had changed `itertools.combinations(replies, 2)` to `3`. The assistant explained that the 2 is the pair size, not a threshold to tune (the threshold is the `0.6`), and that my code comment was inaccurate.
- **Last run:** I shared the final conversations. The assistant judged them the best of the four runs, listed what was still imperfect (item 11 all opening with "Hey Ottawa", weak stances in item 2, persona details leaking into the text, misreadings) and advised keeping this run, saving `generated.jsonl`, and listing the rest as limitations.
- **Before D/E:** it gave a checklist (save the conversations, do not re-run C, delete old outputs, run a 3-item test first) and estimated 15–30 minutes for the 760 D/E calls. The real run took about 35 minutes.

## Another model

**My prompts:** "Which other model is it worth trying to test?"

**Answer (summary):**
- **Model:** it recommended `Qwen2.5-7B-Instruct` first (same family, so only size changes), then `Llama-3.1-8B-Instruct`, and suggested testing on 4 known items before rerunning everything, keeping the 3B outputs either way. I did not run a second model because of GPU limits and time.


## Turning the findings into markdown, and my corrections

**My prompt:** I asked for the findings as markdown sections following the notebook's structure, and to mention that the D/E instructions were based on the guidelines in `annotate.html`.

**Answer (summary):** The assistant wrote the sections for D/E, F and the conclusions, including a note on which guidelines from `annotate.html` were left out of the model's prompt to keep it short.

I then corrected or questioned several points:
- **My number of toxic items:** the draft said 11. I pointed out it was 12. The assistant agreed on 12 and revised the agreement interpretation (a constant "toxic" annotator would score 60%, so several personas without context score below it). It also said my saved file shows 7 items rated 3 and 5 rated 4, the reverse of what I had written, and asked me to check.
- **Religion "separating the personas most":** I argued that agreement around 0.93–0.97 is already very high. The assistant agreed it had overstated this: the difference is about one item in 20, and the ranking changes between label and exact agreement. It rewrote section 8 to say that no dimension clearly separates the personas.
- **Conclusions:** I asked for the whole conclusions section with these corrections, and the assistant rewrote it.

## This file

**My prompts:** "start producing the llm_interactions.md please" (2 November), and "update the llm_interactions.md so I can review it" (3 November). The assistant drafted and updated this file from our conversation, and I checked and completed it.

