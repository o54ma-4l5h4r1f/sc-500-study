# Content changelog

Every structural content change (blueprint remap, learn-map correction, retired feature, renamed doc)
is logged here with the date and the reason.

## 2026-11-15 — Initial build

- Blueprint captured from the live SC-500 study guide (undated outline): 4 domains, 12 subdomains,
  87 bullets, verbatim, into `content/domains.toml`.
- Official curriculum captured from the Learn catalog API (course SC-500T00): 12 learning paths,
  62 unique modules, 412 units, mapped to objective IDs in `content/learn-map.toml`.
  Bullets with no training unit: D1.1.2, D1.1.4, D1.1.6. The module `evaluate-regulatory-compliance`
  is listed under two paths (P3, P10) and recorded once under P3.
- learn-map: `implement-microsoft-defender-servers#3` also maps to D3.2.7 (it teaches agentless scanning limits).
- learn-map: `implement-configure-privileged-identity-management#7` also maps to D1.1.6 (JIT for workload/managed identities).
- learn-map: `evaluate-regulatory-compliance#4` also maps to D1.3.3 (custom standards are configuration). D1.3.7 left unblocked: CIEM works on a Defender CSPM trial; only PIM access reviews need P2 (portal tour on the page).

## 2026-11-15 — Review pass

- Every objective page was re-derived by an independent adversarial reviewer per domain (D1, D2,
  D3.1, D3.2+D3.3, D4) against fresh Microsoft Learn fetches before `status = "verified"` was set.
  Errors fixed include: PIM licence-expiry behaviour, Conditional Access for agents licence routes,
  Key Vault FIPS levels, Backup CMK role (least privilege = Crypto Service Encryption User), SQL Server
  Contributor scope, AKS network policy on Windows (NPM retired → Calico), Defender for Servers direct
  onboarding, Container Apps Key Vault reference refresh, Agent 365 licensing from 1 Jul 2026,
  regulatory-standards plan prerequisite, Storage TLS 1.0/1.1 retirement (3 Feb 2026).
- Final read-only audit (8.6/10). Follow-ups applied: learn-map module URLs (trailing `//`), unit-level
  `license_blocked` derived from bullet flags (33 units), SVG arrowhead fill, D2.1.1 section order,
  D4.2.9 Basic-table management note.
- learn-map: units whose every mapped bullet is licence-blocked now carry `license_blocked = true`.
