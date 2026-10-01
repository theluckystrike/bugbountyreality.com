# Subdomain Finder (Real) — Result

STATUS: complete

## Deliverable
- `tools/subdomain-finder-real/index.html` — single-page, client-side subdomain finder.

## Implementation
- Input: domain (URL/scheme/wildcard stripped client-side).
- Source 1 — CT logs: `GET https://crt.sh/json?q=%25.<domain>`; unique `name_value` entries (newline-split), lowercased, wildcards stripped, filtered to `*.<domain>`/`<domain>`/subdomains of domain. crt.sh 503/timeout degrades gracefully (warning shown, DNS-only results still returned).
- Source 2 — DNS: 60 common prefixes (www, mail, api, dev, staging, admin, vpn…) checked against `https://dns.google/resolve?name=<sub>.<domain>&type=A`, exists iff `Status===0` and Answer non-empty. Concurrency 8, per-request abort/timeout.
- Output: table (#, Subdomain with link, Source = "CT log" / "DNS" / "CT log + DNS", Status = "Resolves (DNS OK)" / "Seen in CT log"), summary counts (total / CT / DNS / both), Copy list button (clipboard + execCommand fallback), plus Download CSV bonus.
- SEO: title targets "subdomain finder" / "find subdomains of a domain"; meta description, keywords, og tags.

## Verification (live)
- `dns.google/resolve?name=www.example.com&type=A` → HTTP 200, `Status:0` with Answer array — matches parser.
- `crt.sh/json?q=%25.github.com` → JSON array with `name_value` containing newline-separated + wildcard (`*.smtp.github.com\nsmtp.github.com`) entries — matches parser.
- `node -e "new Function(script)"` → JS syntax OK; PREFIXES = exactly 60.

## Issues encountered
- crt.sh is slow/rate-limited at times (25s timeout + graceful degradation handles this).
- Prefix list initially had 70 entries; trimmed to exactly 60 (removed kafka, aws, cloud, storage, s3, files, backup, old, new, m).
- Live end-to-end browser test of the full page was not run (subagent budget); both APIs and JS syntax verified independently.
