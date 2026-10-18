# docs-skills

Claude Code skills for documentation work. The skills are plain Markdown
instructions that Claude reads when a task matches them.

## Skills

| Skill | What it does |
|---|---|
| [`staging`](plugins/docs-skills/skills/staging/SKILL.md) | Propose edits to tracked files as a staging document, check live proposals against each other, and apply each one in a single commit after review. |
| [`stage-manager`](plugins/docs-skills/skills/stage-manager/SKILL.md) | Run one session as a coordinator over the others: who owns which staging document, what is safe to apply, and in what order. Builds on `staging`. |

## Install

As a plugin, from inside Claude Code:

```
/plugin marketplace add jecalles/docs-skills
/plugin install docs-skills@docs-skills
```

Or clone it and link one skill into your user skills folder:

```
git clone https://github.com/jecalles/docs-skills ~/src/docs-skills
ln -s ~/src/docs-skills/plugins/docs-skills/skills/staging \
  ~/.claude/skills/staging
```

If another plugin also ships a skill named `staging`, turn one of them
off. Two live copies of one rule drift apart.

## Credit

`staging` started as the `staging` skill in
[nucleus-eng/nucleus-skills](https://github.com/nucleus-eng/nucleus-skills),
by b.next, Inc. and contributors. Development continues here. Changes
that matter upstream go back there as pull requests. Sources and
commits are in [`PROVENANCE.md`](PROVENANCE.md).

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Licence

MIT. See [`LICENSE`](LICENSE).
