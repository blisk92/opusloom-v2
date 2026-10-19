# Triage Labels

The skills speak in terms of five canonical triage roles. This file maps those roles to the actual label strings used in this repo's issue tracker.

| Label in mattpocock/skills | Label in our tracker | Meaning                                  |
| -------------------------- | -------------------- | ---------------------------------------- |
| `needs-triage`             | `needs-triage`       | Maintainer needs to evaluate this issue  |
| `needs-info`               | `needs-info`         | Waiting on reporter for more information |
| `ready-for-agent`          | `ready-for-agent`    | Fully specified, ready for an AFK agent  |
| `ready-for-human`          | `ready-for-human`    | Requires human implementation            |
| `wontfix`                  | `wontfix`            | Will not be actioned                     |

When a skill mentions a role (e.g. "apply the AFK-ready triage label"), use the corresponding label string from this table.

Edit the "Label in our tracker" column to match whatever vocabulary you actually use.

## Bootstrap (run when the GitHub repo is created)

The five labels need to exist on the GitHub repo before `gh` can apply them. Once the repo is initialised and `git remote -v` points at it, run:

```sh
gh label create "needs-triage"     --description "Maintainer needs to evaluate this issue"  --color "fbca04"
gh label create "needs-info"       --description "Waiting on reporter for more information" --color "0e8a16"
gh label create "ready-for-agent"  --description "Fully specified, ready for an AFK agent"   --color "1d76db"
gh label create "ready-for-human"  --description "Requires human implementation"            --color "5319e7"
gh label create "wontfix"          --description "Will not be actioned"                     --color "ffffff"
```

These are idempotent: if a label already exists, `gh label create` reports it and exits non-zero; the script is safe to re-run after switching to a fresh repo. The five colours are placeholders — change them in this file if your repo has house style.
