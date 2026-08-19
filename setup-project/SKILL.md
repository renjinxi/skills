---
name: setup-project
description: Use when the user asks to create a brand-new private GitHub repository from renjinxi's standard project starter. Do not use for adopting, migrating, updating, or cloning an existing repository.
---

# Setup project

Create one new repository from `renjinxi/project-starter`. The stable contract
is:

| Setting | Value |
|---|---|
| Owner | `renjinxi` |
| Visibility | private |
| Template | `renjinxi/project-starter` |
| Local clone | `~/work/products/<repository>` |

The template supplies `AGENTS.md`, the `CLAUDE.md` and skill symlinks,
repository worktree tooling, and `$tmr`.

## Create

1. Obtain the repository name. A description is optional; preserve the user's
   wording rather than inventing product scope.
2. Resolve the destination parent with `cd ~/work/products && pwd -P`. Check
   `gh auth status`, confirm the template still exists and is marked as a
   template, then check both `renjinxi/<repository>` and the destination path.
3. If either the remote repository or local destination already exists, stop
   and report it. Leave it exactly as it is: this skill creates new projects
   and never adopts, clones, migrates, updates, or overwrites old ones.
4. The user's request to create a named private repository is authorization for
   these resolved defaults; do not ask them to confirm the same values again.
   Ask only when the repository name or requested ownership/visibility
   conflicts with the contract.
5. From `~/work/products`, run:

   ```bash
   gh repo create "renjinxi/<repository>" \
     --private \
     --template renjinxi/project-starter \
     --clone
   ```

   Add `--description` only when the user supplied one. Do not add another
   README, license, or gitignore on top of the template.

## Verify

Verify all observable results before reporting completion:

- GitHub reports the target repository as private with default branch `main`.
- The local clone is exactly `~/work/products/<repository>` and its `origin`
  points to `renjinxi/<repository>`.
- `CLAUDE.md` and `.claude/skills` are symlinks with their expected targets.
- `bash tools/work/test.sh` passes in the new clone.
- `git status --short --branch` is clean.

Report the GitHub URL, local path, and verification result. Issue labels,
domain documents, and product-specific context are separate setup work; run
their explicitly requested skill inside the new repository rather than
guessing them here.
