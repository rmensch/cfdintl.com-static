# HANDOFF — cfdintl.com-static

**Last session:** 2026-09-11 — DMARC moved to reject after 24 days of reports showed 361 forgeries and zero legitimate mail; DNSSEC enabled and validated; old AWS server confirmed decommissioned. (Session began 2026-08-18 with the Django→static migration and registrar move.)
**Status at close:** GREEN
**Start here:** in ~2 weeks, drop new DMARC reports into `~/Dropbox/_inbox/dmarc/` and run `python3 dmarc_parse.py .` there — confirm forgery volume is falling and no `pass` entries appear.

## Done this session

- Django 1.11 site mirrored to static HTML, published on GitHub Pages at https://cfdintl.com with HTTPS and IPv6
- Registrar moved GoDaddy → Namecheap with zero email downtime; zone rebuilt from snapshot
- SPF `-all`, DKIM (selector `google`), DMARC `p=reject` — all live, verified via DNS-over-HTTPS
- DNSSEC enabled, validated by Cloudflare and Google
- Old AWS box `54.201.74.233` decommissioned; obsolete local dev-server files removed from `~/code/cfdintl.com`
- Decision records: `docs/decisions/0001`, `0002`; `~/Dropbox/_inbox/2026-09-11 cfdintl.com domain and email decisions.md`

## In flight

Nothing.

## Next actions

- **Re-check DMARC in ~2 weeks** — `cd ~/Dropbox/_inbox/dmarc && python3 dmarc_parse.py .` after adding new report zips. Expect forgeries trending down; any `pass` = first legitimate sender, worth knowing.
- **Decide on the "Contact Us" heading** — the page has no contact details now (CAGE/DUNS, terms PDFs, brochures only). Rename to "Company Information" or similar? One-line edit in `contact/index.html` plus the nav `<li>` on all 40 pages.
- **Investigate DNS interception on the MacBook Pro** — every `dig @server` is answered by a Cloudflare resolver (108.162.x.x). Likely Cloudflare WARP or a router setting. Not urgent; DoH works around it. Noted in global CLAUDE.md.
- **Bump `actions/*` versions** in `.github/workflows/pages.yml` when Node 24 releases land — GitHub is retiring Node 20 on runners. Cosmetic until then.
- **Archive `~/code/cfdintl.com`** to `~/code/_archive/`? Owner's call — it is retired source, 306 MB, and contains plaintext credentials that were deliberately not rotated.

## Open questions

- Rename the Contact page heading? — owed by: Rusty
- Archive the old Django source? — owed by: Rusty

## Known gotchas

- Namecheap: changed/deleted records take 20–30 min to appear on their own nameservers; new records are instant. The host-record list collapses behind SHOW MORE. Both caused false "it didn't save" diagnoses this session.
- `dig` from this Mac lies — use DoH (`https://dns.google/resolve?name=…&do=1`).
- Google Admin Toolbox "Check MX" times out routinely; not evidence of anything.
- **DNSSEC is on.** If DNS or registrar ever moves: disable it first, wait 1–2 days, then change nameservers.
- Rapid `curl` sweeps of the live site draw 503s from GitHub's edge.

## Dropped

- reCAPTCHA / SMTP / SECRET_KEY rotation — site gone, mailbox inactive, server decommissioned (Rusty)
- `/contact/` redirect — page was restored, moot
- Quarantine stage for DMARC — skipped deliberately; no legitimate mail to protect
