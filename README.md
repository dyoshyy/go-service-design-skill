# go-service-design

A Claude Code skill for accumulating design criteria for Go services.

It distills the decision criteria from 135yshr's 『Go サービス設計』 (Go Service Design, https://github.com/135yshr/documents/tree/main/books/go-service-design) into a decision map, and grows as projects confirm or revise those criteria. Nothing is reproduced verbatim; everything is paraphrased.

## Layout

```
SKILL.md          Triggers, five-step design, decision map, how to grow it
rule.md           What a diff must never break (meant to be always loaded for Go files)
references/       One file per chapter: the book's default / when it applies / how to spot violations / source
```

## Install

```sh
# User-wide (recommended; available in every project)
npx skills add dyoshyy/go-service-design-skill -g

# Per project
npx skills add dyoshyy/go-service-design-skill
```

`rule.md` is not loaded automatically as part of the skill. To have it apply whenever Go files are touched, place it under `.claude/rules/`.

```sh
# User-wide (a symlink works here)
ln -s ~/.claude/skills/go-service-design/rule.md ~/.claude/rules/go-service-design.md

# Per project (project-level .claude/rules/ does not follow symlinks, so copy)
cp .claude/skills/go-service-design/rule.md .claude/rules/go-service-design.md
```

## How to grow it

When a project shows that a default is wrong, incomplete, or needs a condition, revise the matching `references/NN-*.md` in general terms: adjust the default, add an exception, or add a symptom to "How to spot violations". Project-specific rationale stays in the project's own decision log.

## Update

```sh
npx skills update go-service-design
```
