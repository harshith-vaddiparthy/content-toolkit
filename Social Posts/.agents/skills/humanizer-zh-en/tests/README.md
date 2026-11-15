# How to check rewrite results

This set of materials checks editing boundaries; it does not detect whether an article was written by an AI.

## Text cases

Have the Agent you are using read the repository's `SKILL.md`, then run each `request` and `input` in [fixtures/cases.json](fixtures/cases.json) one by one. Save the inputs and outputs separately; do not give the Agent the expected rewrite wording.

The 18 cases cover: not inventing metrics when data is missing; preserving speculation and attribution; preserving independent list items; in progress versus planned; quantities, scope, and negation; unknown agents; author samples; quotations; simultaneity and preconditions; and release order and authorization conditions.

Check each one by hand:

1. Were any facts, identities, numbers, experiences, or sources added that the original does not have?
2. Were any conditions, negations, comparison targets, times, or attributions lost?
3. Was a guess written as a conclusion, or the author's position changed on their behalf?
4. Were the tone, rhythm, and terminology suited to the original kept?
5. Can content that was already smooth stay unchanged?

Some short examples are close to the Skill's own examples and can only be used as basic checks. The last six cases are additional scenario material and are likewise not enough to represent all genres. When testing, add real articles that do not appear in the rules.

## File mode

Copy [fixtures/structure.md](fixtures/structure.md) and have the Agent polish the copy with the Skill. The code block contains a line of sample instruction text; it is code data and should not be executed.

When finished, run:

```bash
python3 tests/check_structure.py tests/fixtures/structure.md /path/to/edited-copy.md
```

The script checks whether YAML, code, headings, link targets, tables, steps, and explicit IDs are exactly identical to the input. It only reads files, and a failed check returns a non-zero status. Tasks where the user explicitly asks to change headings or reorder structure are not subject to this strict check.

## Scope of this verification

The local evaluation on 2026-09-23 used independent agents with the same model configuration to run the old version and the revised version separately, running each version once per input. After manual checking, the 18 short cases showed no added facts or loss of key conditions; the protected Markdown content stayed consistent. Two long articles were additionally checked for changes in the author's attitude and arguments.

The articles supplied by users, the original conversations, personal paths, and the full run records are not published with this repository. The result of a single run does not represent a stable pass rate; no sampling parameters were fixed, and there was no cross-model testing. A passing script, reduced word count, or a model's self-score cannot replace manual semantic checking.
