# Working on the ruhnu forks

The `cdoc2-*` repositories here are forks of [open-eid](https://github.com/open-eid).
Ruhnu work lives in the forks; we don't expect upstream to take it. What matters
is that upstream changes keep merging into our work with as few conflicts as
possible. This page describes how.

## Branches

| Branch | What it is | Who writes to it |
|---|---|---|
| `develop` | Exact mirror of `open-eid/<repo>:develop` | Fast-forward from upstream only |
| `ruhnu` | Default branch: upstream plus our work | PRs; admins may push upstream syncs and small fixes |
| `<topic>` | One change, started from `ruhnu` | You |

```
upstream develop ──●──────────────●─────────▶  (mirrored into develop)
                    \              \
 ruhnu         ──●───M────●───M─────M───────▶  ← upstream merged in
                           \     /
 topic                      ●──●               → PR into ruhnu
```

`ruhnu` is protected by a ruleset: changes go through a pull request. Repository
admins are on its bypass list, so an upstream sync or a small fix can be pushed
directly. Use PRs for anything the other person should see before it lands.

## Set up a clone

```bash
git clone https://github.com/ruhnu/<repo>
cd <repo>
git remote add upstream https://github.com/open-eid/<repo>
gh repo set-default ruhnu/<repo>
```

`gh repo set-default` matters: in a fork, `gh pr create` otherwise targets the
**parent** repository, so a PR meant for internal review would land on open-eid.

## Make a change

1. Start from `ruhnu`:
   ```bash
   git fetch origin
   git switch -c <topic> origin/ruhnu
   ```
2. Keep it to one logical change. Follow upstream's commit style and conventions
   (read the repo's recent history).
3. Open a PR from `<topic>` into `ruhnu` and merge it with a merge commit (no
   squash, no rebase), so each topic stays one identifiable merge.

To keep upstream merges cheap:

- Prefer adding over rewriting upstream code: new classes, new config keys,
  behaviour behind a flag such as `cdoc2.ruhnu.enabled` (default off = upstream
  behaviour). Every line changed in an upstream file is a possible conflict.
- Keep ruhnu-only files clearly named (`ruhnu-*.yml`, `.github/ruhnu-*`), and
  leave upstream's workflows and build scripts alone.
- Write commits that make sense on their own, so one can still be
  cherry-picked if upstream ever wants it.

### A change that depends on another unmerged change

Start it from that change's branch:

```bash
git switch -c <topic-b> <topic-a>
```

Say in the B PR that it builds on A, and merge A first.

### A change across several repositories

Use the same topic branch name in every repository it touches. Open the PRs
in dependency order — `cdoc2-openapi`, then `cdoc2-auth`, then
`cdoc2-java-ref-impl` and `cdoc2-shares-server` — and link them to each other.
The design and task list for the whole change live in
[ruhnu-openspec](https://github.com/ruhnu/ruhnu-openspec).

## Keep up with upstream

When upstream has new commits:

```bash
git fetch upstream
git switch develop && git merge --ff-only upstream/develop && git push origin develop
git switch ruhnu   && git pull --ff-only && git merge develop && git push origin ruhnu
```

The last push needs the admin bypass. Without it, push the merge to a branch
and open a PR into `ruhnu` instead.

If `--ff-only` fails on `develop`, something was committed to it. Fix that
first; `develop` must never diverge from upstream.

Resolve conflicts in the merge commit itself. If upstream changed code we
modified, keep upstream's version and re-apply our change on top, ideally in a
way that touches less of their code.

## If upstream wants a change

Branch from `upstream/develop`, cherry-pick the relevant commits, and open the
PR to `open-eid/<repo>:develop` from that branch, never from `ruhnu`.

## Keep tooling files out of commits

Local tooling (`.claude/`, the `openspec/` store wiring, IDE files) must not
appear in a commit. Do not add it to `.gitignore`: that edits an upstream file
and invites merge conflicts. Put it in the clone's `.git/info/exclude`, or in
your global ignore file (`~/.config/git/ignore`), instead.

## Never

- commit to `develop`;
- force-push `develop` or `ruhnu`;
- open an upstream PR from `ruhnu`.
