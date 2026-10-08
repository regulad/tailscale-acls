# AGENTS.md — tailscale-acls

regulad's tailnet policy file, managed GitOps-style. `policy.hujson` is the
single source of truth; the admin-console policy editor is locked.

## Commit requirements

Every commit created or rewritten by an AI agent in this repository must
carry a `Co-Authored-By` trailer naming the agent and model, for example:

    Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

Keep Git commit signing enabled; never disable it.

## Applying changes

- Commit straight to `master` and push; no pull requests. Every push that
  changes `policy.hujson` (or the workflow) runs
  `.github/workflows/tailscale.yml`, which applies the policy to the live
  tailnet, so such a push is a production change.
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
- Comment every tag, grant, nodeAttr and autoApprover with what it is for,
  and every port in a grant's `ip` list with the service it serves.
- Names derived from a domain spell each `.` as `--`; a single `-` is an
  ordinary separator (`tag:edge-regulad--internal` is the edge router for
  `regulad.internal`).
- IPv6 is first class in this network; IPv4 is secondary. Every machine and
  subnet is listed with its IPv6 address(es), IPv6 first. Don't add an
  IPv4-only entry without finding the IPv6 counterpart; a machine that
  genuinely has no IPv6 is a legacy host, and its comment says so.
- Never list the homelab's public IPv6 prefix anywhere: it is
  DHCPv6-assigned and changes. Its ULA (`fd83:b84c:aa57:1::/64`) is what the
  policy uses. Grants give "the internet" as `ipset:internet`
  (`autogroup:internet` minus the VCN's public /64), not as
  `autogroup:internet` directly.
- Name a single-address machine in `hosts`; anything with several addresses
  (a site's subnets, a machine with IPv6 and IPv4) is an `ipsets` entry.

## Tests

The policy's `tests` assert, per principal, what it must reach and what it
must not; Tailscale rejects any policy that fails one. Tests can't reference
IP sets, so their destinations are literal addresses (IPv6 bracketed, e.g.
`[fd83::1]:443`): when a machine's address changes, update its tests too.
Site devices are tested through the test-only `bogus--*` hosts. A tag
source matches the every-node (`*`) grants only over IPv4 in tests, so check
those grants over IPv6 from `bogus--tail11540--ts--net`, not from a tag.
When you add a principal or a grant, add tests for it, including denies for
ports it must not reach (SSH, NetBIOS, SMB, AFP, lockdownd).

## DNS

`dns.json` is the tailnet's DNS configuration, in the shape of
`GET /api/v2/tailnet/-/dns/configuration`. Every push to `master` that
changes it runs `.github/workflows/dns.yml`, which prints a diff against the
live configuration and POSTs the file to `/dns/configuration`. That
replaces the whole configuration: anything the file omits is cleared, and
omitted `preferences` (`magicDNS`, `overrideLocalDNS`) default to false, so
always edit the full file and never change DNS in the admin console. The
workflow can also be run by hand (`gh workflow run dns.yml`) to re-apply
it. Every node must be
able to reach the global nameservers ("Override DNS servers" is on, so a
node that can't loses DNS entirely). The resolvers grant opens every
nameserver in `dns.json`, global and split-DNS, to every node on port 53;
add new nameservers there.
