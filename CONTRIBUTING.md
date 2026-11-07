# Working on the ruhnu forks

The `cdoc2-*` repositories here are forks of [open-eid](https://github.com/open-eid).
Changes are meant to go upstream, so every change must stay mergeable into
`open-eid/<repo>:develop` on its own. This page describes how to keep it that way.

## Branches

| Branch | What it is | Who writes to it |
|---|---|---|
| `develop` | Exact mirror of `open-eid/<repo>:develop` | Fast-forward from upstream only |
| `ruhnu` | `develop` plus our changes merged together, so they can run as one system | Merges only |
| `<topic>` | One change, started from `develop` | You |

```
upstream develop ──●──────────────●─────────▶  (mirrored into develop)
                    \              \
 topic               ●──●──●        \           → PR to open-eid:develop
                            \        \
 ruhnu         ──●───────────M────────M──────▶  ← topics and upstream merged in
```

The one rule: **`ruhnu` is where changes are combined, never where work starts.**
A branch started from `ruhnu` carries every unmerged change in it, and its
upstream PR drags all of them along.

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

1. Start from the upstream mirror:
   ```bash
   git fetch upstream
   git switch -c <topic> upstream/develop
   ```
2. Keep it to one logical change. Follow upstream's commit style and conventions
   (read the repo's recent history), and do not bump versions unless upstream
   asks for it.
3. Before opening a PR, rebase onto the current upstream rather than merging it
   in:
   ```bash
   git fetch upstream && git rebase upstream/develop
   git push --force-with-lease origin <topic>
   ```
4. Open an internal PR from `<topic>` into `ruhnu`, and merge it with a merge
   commit (no squash, no rebase), so each topic stays one identifiable merge.
5. When it is ready for upstream, open a PR from the **same** `<topic>` branch
   into `open-eid/<repo>:develop`. Merging into `ruhnu` did not change the
   topic branch, so it still applies cleanly.

Write each commit so it could be cherry-picked on its own: upstream may apply
contributions rather than merge them.

### A change that depends on another unmerged change

Start it from that change's branch, not from `ruhnu`:

```bash
git switch -c <topic-b> <topic-a>
```

Upstream, open the B PR after A, and say in it that it builds on A. Once A is
merged upstream, rebase B onto `upstream/develop`.

### A change across several repositories

Use the same topic branch name in every repository it touches. Open the PRs
in dependency order — `cdoc2-openapi`, then `cdoc2-auth`, then
`cdoc2-java-ref-impl` and `cdoc2-shares-server` — and link them to each other.
The design and task list for the whole change live in
[ruhnu-openspec](https://github.com/ruhnu/ruhnu-openspec), which never goes
upstream.

## Keep up with upstream

```bash
git fetch upstream
git switch develop && git merge --ff-only upstream/develop && git push origin develop
git switch ruhnu   && git merge develop                    && git push origin ruhnu
```

If `--ff-only` fails, something was committed to `develop`. Fix that first;
`develop` must never diverge from upstream.

### When merging upstream into `ruhnu` gets messy

If upstream applies one of our changes in its own form, merging `develop` into
`ruhnu` conflicts on code that is already there. Rather than fight that,
rebuild `ruhnu` from scratch with the topics upstream has not taken yet:

```bash
git switch -C ruhnu upstream/develop
git merge --no-ff <topic> ...        # every topic still open upstream
git push --force-with-lease origin ruhnu
```

Since nothing is ever committed directly on `ruhnu`, this loses nothing. Tell
the others first: their local `ruhnu` will need `git reset --hard origin/ruhnu`.

## Keep tooling files out of commits

Local tooling (`.claude/`, the `openspec/` store wiring, IDE files) must not
appear in a PR diff. Do not add it to `.gitignore` — that change would ride
along into every upstream PR. Put it in the clone's `.git/info/exclude`, or in
your global ignore file (`~/.config/git/ignore`), instead.

## Never

- commit directly to `develop` or `ruhnu`;
- force-push `develop`, or `ruhnu` outside a rebuild everyone knows about;
- start a topic branch from `ruhnu`;
- open an upstream PR from `ruhnu`.
