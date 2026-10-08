# AGENTS.md — tailscale-acls

regulad's tailnet policy file, managed GitOps-style. `policy.hujson` is the
single source of truth; the admin-console policy editor is locked.

## Commit requirements

Every commit created or rewritten by an AI agent in this repository must
carry a `Co-Authored-By` trailer naming the agent and model, for example:

    Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

Keep Git commit signing enabled; never disable it.

## Applying changes

- Commit straight to `master` and push; no pull requests. Every push runs
  `.github/workflows/tailscale.yml`, which applies `policy.hujson` to the
  live tailnet, so a push is a production change.
- Pushing and watching the run takes minutes, so commit small or cosmetic
  changes locally and push them along with the next semantic change.
- Tailscale validates the file (syntax and any `tests`) before accepting it.
  A rejected file fails the workflow and leaves the tailnet untouched; check
  with `gh run list` and `gh run view <id> --log-failed`.
- Don't change the policy in the admin console: the next push overwrites
  console edits without warning.
- Pin every action by full commit SHA with a `# vX.Y.Z` comment.

## Policy conventions

- The policy is a whitelist: nothing is reachable unless a grant allows it.
  Grants are allow-only (Tailscale has no deny rules and no rule priority)
  and overlapping grants are unioned, so restricting access means narrowing
  or removing the grant that gives it. Don't reintroduce a `* -> *` grant.
- Before removing or narrowing a grant, check who else relies on it; a
  principal with no grant loses all access.
- Comment every tag, grant, nodeAttr and autoApprover with what it is for.
- Names derived from a domain spell each `.` as `--`; a single `-` is an
  ordinary separator (`tag:edge-regulad--internal` is the edge router for
  `regulad.internal`).
