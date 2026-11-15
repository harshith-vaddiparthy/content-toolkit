# Changelog

## 2026-09-23

This update focuses on one problem: the article was made smoother, but the meaning changed.

- Explicitly require preserving facts, numbers, negation, conditions, time, attribution, and the author's attitude. "May" cannot be changed to "definitely", and "everyone feels" cannot be changed to "I used to feel".
- Organized 31 checkpoints based on upstream v3.0.0 and PR #39, rewrote the examples, and added to each rule its applicable conditions and the expressions that should be kept.
- Fixed existing examples that invented product features, research data, personal experiences, and the like out of thin air. Information the original does not contain is not to be supplied through polishing.
- No longer change text mechanically based on words, punctuation, or the number of listed items. Genuine three-feature lists, necessary em dashes, Chinese parallelism, and connectives can be kept.
- Match the author's sample and genre. Technical descriptions, business reports, and personal essays are handled separately rather than all being changed to a chat tone.
- File mode explicitly protects code, commands, paths, links, YAML, headings, and anchors; by default only the final draft is returned, and the self-scoring requirement is dropped.
- Added 18 short-text cases, a Markdown file sample, and a read-only structure-check script. Completed a single-version comparison on two long articles; the effect still needs to be observed across more models and genres.

References and thanks: [blader/humanizer v3.0.0](https://github.com/blader/humanizer/blob/v3.0.0/SKILL.md), [PR #39](https://github.com/op7418/Humanizer-zh/pull/39), [PR #34](https://github.com/op7418/Humanizer-zh/pull/34), and [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop). This consolidation did not merge those PRs directly.
