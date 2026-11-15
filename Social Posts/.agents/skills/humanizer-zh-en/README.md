# Humanizer-zh: A Skill for Polishing Chinese Text

Edit filler, repetition, and templated phrasing in existing articles, commentary, and documents, while preserving facts, degree of certainty, and the author's voice. The input is a passage of text or a file; the default output is the final rewritten draft, and sentences with no problem can be left unchanged.

This is editing guidance that an Agent reads and carries out, not a standalone detection program. It cannot prove who wrote an article, and it does not guarantee passing any AI detector.

> Translator's note: this is an English translation of the Chinese `humanizer-zh` README. The skill itself still edits Chinese text; the examples below are translated from Chinese.

> Update 2026-09-23: rewrote the rules and examples to explicitly preserve facts, the author's position, and genre. Ordinary parallelism, em dashes, and connectives are no longer changed across the board. See the [changelog](CHANGELOG.md).

## Installation

### Method 1: One-step install via npx (recommended)

```bash
npx skills add https://github.com/op7418/Humanizer-zh.git
```

This is the simplest way to install; it puts the skill in the correct directory automatically.

### Method 2: Clone with Git

```bash
# Clone into Claude Code's skills directory
git clone https://github.com/op7418/Humanizer-zh.git ~/.claude/skills/humanizer-zh
```

### Method 3: Manual install

1. Download this project's ZIP file or clone it locally
2. Copy the `Humanizer-zh` folder into Claude Code's skills directory:
   - **macOS/Linux**: `~/.claude/skills/`
   - **Windows**: `%USERPROFILE%\.claude\skills\`

3. Make sure the folder structure looks like this:
   ```
   ~/.claude/skills/humanizer-zh/
   ├── SKILL.md       # Skill definition file
   └── README.md      # Documentation
   ```

### Verify the installation

After restarting Claude Code or reloading skills, type in a conversation:

```
/humanizer-zh
```

If the installation succeeded, the skill will be activated.

## Usage

Pasted text:

```text
Please use humanizer-zh to polish the article below, keeping the tone of a technical commentary:
[original text]
```

Editing a file:

```text
Please use humanizer-zh to polish the body text in article.md.
```

For review only, say "give suggestions, don't modify the file". If you have an author sample, you can provide it too; the Skill will borrow its expressive habits but will not move experiences, data, or viewpoints from the sample into the original.

## Rewrite examples

All of the following are teaching inputs; there is no supplementary material that is not shown.

**Product description, missing data:**

> Original: The plan is stable and reliable, fast in response, and easy to maintain.
>
> Rewrite: The plan runs stably, responds quickly, and is also easy to maintain.

No launch date, response speed, or maintenance cost was added. The original lacks evidence; you can ask the author for it, but polishing must not supply data.

**Bug report, with uncertainty:**

> Original: This problem has been reported by the community many times, is thought to possibly be related to a memory leak, but the cause has not been confirmed.
>
> Rewrite: The community has reported this problem many times. It is thought to possibly be related to a memory leak, but the cause has not been confirmed.

"Possibly related" and "not yet confirmed" are kept; a guess is not turned into a causal conclusion, and the judgment about the cause is not attributed to the community without basis.

**Release notes, with three real features:**

> Original: This update demonstrates the team's pursuit of innovation. It adds export, search, and batch renaming. Offline editing is planned for the next version and has not yet finished testing.
>
> Rewrite: This update adds export, search, and batch renaming. Offline editing is planned for the next version and has not yet finished testing.

The three features are kept, with no item cut or padded just to avoid a triplet; the plan and testing status keep their original meaning.

**Text that is already clear:**

> Original: Save the file first, then close the window. Unsaved changes will be lost.
>
> Rewrite: Save the file first, then close the window. Unsaved changes will be lost.

Not every invocation is required to produce a change.

## Editing principles

The order of constraint priority is: preserve information and degree of certainty, respect the user's scope and genre, match the author's voice, then handle the specific expression problems. A pattern hit cannot override these requirements.

- Do not invent features, data, sources, people's identities, or first-person experiences.
- Preserve negation, conditions, attribution, scope, time, and completion status; "may" does not become "definitely", and "planned" does not become "already".
- Three-item lists, four-character phrases, passive sentences, em dashes, and connectives are judged by their function and are not deleted mechanically.
- Essays can keep their personality, and academic and business texts keep their formality; not every article is turned into the same colloquial style.
- File editing by default preserves code, commands, paths, link targets, YAML, data, headings, and anchors. Changing headings or structure requires checking references.
- By default only the final draft is delivered; showing intermediate drafts, hit lists, or self-scores is not required.

## Pattern catalog

31 checkpoints are kept, following the A–F classification of PR #39 for easy comparison. This is a list of editing problems, not a standard for detecting authorship.

| Category | Checkpoints |
|---|---|
| A: Lead-in instead of statement (1–5) | False contrast, dramatic fragments, pseudo-depth, run-up lead-ins, defense with no target |
| B: Formulaic rhythm (6–11) | Forced triplets, repeated openings, all-purpose em dash, stacked hedges, coined compounds, passive and missing subject |
| C: Inflation and borrowed authority (12–18) | Empty high-frequency words, inflated significance, vague associations, tail-end inflation, promotional language, appeals to authority, complex copulas |
| D: Formulaic layout (19–21) | Ineffective bold, decorative headings, quotation marks and punctuation |
| E: Chat and draft residue (22–25) | Customer-service tone, repeated disclaimers and speculation filler, first sentence echoing the heading, editing-process residue |
| F: Chinese supplementary checks (26–31) | Long modifiers, "进行 + verb", stacked 被-constructions, four-character parallelism, all-purpose background, boilerplate endings |

Each item includes the condition for changing, before-and-after examples, and the boundary to keep; see [SKILL.md](SKILL.md) for details.

## Verification

The [test notes](tests/README.md) include 18 short-text cases, a Markdown file sample, and a structure-check script. This update also ran a long-text comparison of the old and revised versions, checking numbers, conditions, attribution, the author's attitude, and file structure. A single, limited set of samples cannot represent all models and genres; reduced word count or model self-scores cannot be used to prove effectiveness.

## Sources and license

- [blader/humanizer v3.0.0](https://github.com/blader/humanizer/blob/v3.0.0/SKILL.md): the original project and the source of the A–E classification, voice calibration, and file mode.
- [Humanizer-zh PR #39](https://github.com/op7418/Humanizer-zh/pull/39): the structural basis of this revision and the source of the Chinese checkpoints.
- [Humanizer-zh PR #34](https://github.com/op7418/Humanizer-zh/pull/34): comparison reference for the example-fidelity fix. This revision did not merge that branch directly.
- [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop): reference source for concise expression and editing checks.
- [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing): observation source of the original project.

Released under the [MIT License](LICENSE) in the repository.
