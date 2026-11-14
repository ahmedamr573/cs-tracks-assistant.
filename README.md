# CS Tracks Assistant

A free, open-source, fully local chatbot that answers in-depth questions about computer science career tracks (backend, AI/ML, cybersecurity, data, networking, mobile) in English and Arabic. For anything outside computer science it replies with one fixed sentence:

> I can only help with computer science tracks.

No paid API and no internet are needed at run time. Everything runs on your own computer.

## How it works

```
User question
     |
     v
[1] Scope gate (app.py)
     - clearly off-topic words (jokes, recipes, football, weather, politics, "ignore your rules") -> refuse, no model call
     - clearly CS words (python, sql, tcp, jwt, malware, ...) -> allow
     - unclear -> ask the model "YES or NO: is this about computer science?" with 10 examples
     |
     v
[2] Answer generation: custom Ollama model "cs-bot" (qwen2.5:7b + Modelfile rules)
     - hidden note adds the reply language (English or Arabic)
     - "what should I learn first" questions get a hidden learning-path instruction
     |
     v
[3] Output filters (code)
     - removes lines that name course platforms or quoted course/book titles
     - if the model refuses a question the gate approved, retries once
```

Why the rules are in code and not only in the prompt: a 7B model ignores prompt rules often, so scope checks and filters are enforced in Python.

## Files

| File | What it is |
|---|---|
| `Modelfile` | Builds the `cs-bot` model from `qwen2.5:7b` (temperature 0.2, system rules 1 to 7) |
| `app.py` | Scope gate, answer logic, filters, and a small local web chat page (standard library only) |
| `gate.py` | Earlier terminal version of the chat |
| `run_tests.py` | Runs the 42-question test bank and grades it automatically |
| `track_questions.txt` | The 42 test questions |
| `results_summary.txt` | Result of the final full test run |

## Setup

1. Install [Ollama](https://ollama.com) and Python 3.
2. Download the base model: `ollama pull qwen2.5:7b`
3. Build the custom model: `ollama create cs-bot -f Modelfile`
4. Start the Ollama server: `ollama serve` (keep this window open)
5. In another terminal, in this folder: `python app.py`
6. Open http://localhost:8000 in a browser.

The first answer is slow (model loading). Later answers take about 20 to 50 seconds on a laptop with a small GPU.

Run the tests: `python run_tests.py` (about 45 minutes), or only some questions: `python run_tests.py B5 N5`.

## Results

Test bank: 42 questions. 30 track questions (5 per track), 8 off-topic questions that must be refused, 4 borderline questions (for example "What do pythons eat?" must be refused, "Is Java the same as JavaScript?" must be answered).

Grading is automatic. Each answer needs a minimum number of checklist points (keywords for facts a good answer should contain). Off-topic questions must return exactly the fixed sentence.

| Version | Result | Notes |
|---|---|---|
| Plain model, no rules | not measured | |
| Prompt rules only | not measured | a 7B model often ignores the rules |
| Hybrid gate (v3) | 38 / 42 | failures: A4 (wrong language), C5 and Y3 (named courses), Y2 (virus vs worm refused) |
| Final (v6) | **41 / 42** | only N5 failed (2/4 points) |

Off-topic refusals: 8 / 8 in the final run (instant when keywords catch them).

## Limitations (please read)

- **Wrong facts happen.** The automatic check only looks for keywords. It cannot detect a wrong statement. In manual reading, some answers contained mistakes (for example database normalization and some details about Flutter and the CIA triad). Always verify important facts.
- **Answers vary.** Temperature is low but not zero. N5 passed 4/4 in one run and 2/4 in another.
- **Slow.** A 7B model on a laptop GPU takes 20 to 50 seconds per answer.
- **Arabic is weaker than English.** It is usable, but simpler and sometimes less accurate.
- **Fixes were tuned after seeing failures.** The test bank was used to find problems and the gate, filter and hint were then adjusted. So 41 / 42 is not a clean held-out score. The earlier runs ran on older versions of the code.
- **The test is a sample.** 42 questions do not prove it works for every question.
- **No retrieval or fine-tuning.** Answers come from the base model's knowledge.

## Future work

- Retrieval over a curated, reviewed set of track documents (RAG) to reduce wrong facts.
- Fine-tuning on reviewed question-answer pairs, including Arabic.
- A held-out test set that is never used for tuning, and human grading.
- Website and mobile app integration.

## License and credits

Built with [Ollama](https://ollama.com) and the open-weight model Qwen2.5-7B. Check each project's license before redistributing the model.
