# regulad's Tailscale policy

## Layout

## Applying changes

## Policy

## Tests

## DNS

## AGENTS.md

[`AGENTS.md`](AGENTS.md) is the working guide for this repository, written
for AI agents but binding on anyone who changes it. It covers:

- **Commit requirements:** the `Co-Authored-By` trailer every AI-assisted
  commit carries, and commit signing.
- **Applying changes:** every push to `master` applies the policy to the live
  tailnet, small changes are batched until the next semantic one, and
  actions are pinned by commit SHA.
- **Policy conventions:** the whitelist model, comments on every fixture and
  port, `--` for each dot in domain-derived names, IPv6 first, `hosts`
  versus `ipsets`, and `ipset:internet`.
- **Tests:** what the policy's tests assert, how to extend them, and the
  test runner's quirks.
- **DNS:** how `dns.json` is applied, and why every nameserver must stay
  reachable.

## License
