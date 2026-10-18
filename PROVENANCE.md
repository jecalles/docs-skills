# Provenance

Where each skill came from. Each source names repo, branch and commit.

## staging

- **Source:** [nucleus-eng/nucleus-skills](https://github.com/nucleus-eng/nucleus-skills),
  `plugins/nucleus/skills/staging/SKILL.md`, by b.next, Inc. and
  contributors.
- **Base:** branch `staging/questions-carry-their-answers`, commit
  `9fa8ccc` (pull request #94, unmerged at the time of copying). It sits
  on `main` at `80ed001`.
- **Changes here:** examples and checker names that pointed into the
  source repository are made generic.
- **Not yet ported:** the source's checkers in `scripts/`
  (`check-tags.py`, `check-sites.py`, `check-citations.py`,
  `check-pins.py`, `check-pin-freshness.py`, `check-table-shape.py`,
  `staging_common.py`). They are written for the source's repositories
  and need generalizing first.

## stage-manager

- **Source:** new in this repository. It builds on `staging` above, and
  its rule that rulings live beside their question follows pull
  request #94.
