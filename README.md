# regulad's Tailscale policy

This repository contains my Tailscale configuration: both DNS and ACL. It is the sister project to my [`dotfiles`](https://github.com/regulad/dotfiles) configuration.

My tailnet includes site-to-site routing, self-hosted and Mullvad-hosted exit nodes, and scoped access via tags. For instance, CI job runners that need to SSH into a specific host are only permitted to access that host (and any DNS resolvers). 

Anthropic LLMs were used to assist with writing test cases and converting policy to newer formats, but the architecture is hand-defined.

I hope this repository proves useful to you! Much of the techniques for scoping access here will be very useful if you plan to use Tailscale to allow AI agents restricted access to your personal networks.

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
