# Data annotation with SynDisco

First assignment for *Practical Data Science* (MSc Data Science). I rated 20 online comments for toxicity myself, then compared my ratings with those of LLM annotators that have different personas (gender, politics and religion). Each LLM annotator rated the comments both with and without a generated discussion about them as context.

All steps use [SynDisco](https://github.com/dimits-ts/syndisco) with `unsloth/Qwen2.5-3B-Instruct-bnb-4bit`.

## Run it in Google Colab

The easiest way to reproduce the results is to run the whole notebook in Google Colab, because the model needs a GPU.

1. Open the notebook with the badge above.
2. Choose a GPU: **Runtime → Change runtime type → T4 GPU**.
3. Upload `items/items.csv` and `items/items_rated.csv` from this repository using the **Files** panel on the left.
4. Run everything: **Runtime → Run all**.

The first cell installs SynDisco, and the model downloads automatically. Part C (generating the conversations) takes about 10 minutes on a T4. Parts D and E (760 annotation calls) take about 35 minutes. The notebook writes `generated.jsonl` and `annotations.csv` to the Colab session, so download them before the session ends if you want to keep them.

The free Colab GPU has usage limits.

## What the notebook does

| Part | Content |
|---|---|
| A | Selecting 20 items with my seed (`prepare_items.py`) and rating them by hand (`annotate.html`) |
| B | My ratings: distribution and comments |
| C | Generating one conversation per item, with a pool of personas and stances, plus a quality check |
| Grid | 18 annotator personas (gender × politics × religion) and a baseline with no persona |
| D, E | Every annotator rates every item, with the conversation as context (D) and without it (E) |
| F | Agreement with my ratings, the effect of the context, and agreement between annotators |
| G | Conclusions and limitations |

## Repository contents

| Path | Content |
|---|---|
| `First Assignment.ipynb` | The notebook, with all outputs saved |
| `prepare_items.py`, `data/pool.csv` | The script and pool used to select my 20 items |
| `items/items.csv` | My 20 items |
| `items/items_rated.csv` | My own ratings |
| `items/annotate.html` | The annotation page I used to rate the items |
| `items/annotations.csv` | The LLM annotators' ratings from parts D and E |
| `model output/` | The conversations from each of the four part C iterations; the final one is `generated_qwen2_5_stances_improved.jsonl` |
| `llm_interactions.md` | A log of my use of an AI coding assistant |

## Reproducibility

All random choices come from the seed in the first cell, and the model uses greedy decoding. The same seed and model should therefore reproduce the saved outputs, though different GPU or library versions can cause small differences.
