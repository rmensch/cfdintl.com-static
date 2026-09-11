# CLAUDE.md — cfdintl.com-static

Static snapshot of the CFD International website, served by GitHub Pages at
https://cfdintl.com. Not in the ship-it audience — nothing runs on the Mac mini.
Deploy is a push to `main`; the Actions workflow publishes it.

**Read `HANDOFF.md` first** for where the last session left off.
`DEPLOY.md` is the migration record and the DNS/email/DNSSEC reference.

## Editing

Plain HTML. `index.html` is the home page; every other page is `<name>/index.html`.
Links are root-relative (`/static/...`), so the site only renders correctly at a
domain root — the `rmensch.github.io/cfdintl.com-static/` URL will look unstyled
and that is expected. Preview locally with `python3 -m http.server 8080`.

`git pull` before editing: GitHub may write to the repo when Pages settings change.

## Things that look broken but are not

- ~220 gallery image references 404. The template emits nine slots per gallery
  and each `<img>` has `onerror="this.parentNode.remove()"`. By design; identical
  on the original site.
- The `Contact Us` page has no contact details. Removed at the owner's request;
  the page keeps CAGE/DUNS numbers, terms PDFs and brochures.
- Filenames like `x1_lg.jpg.pagespeed.ic.XXXX.jpg` are Google PageSpeed output
  from the old server, mirrored as-is. Do not "clean them up" — the HTML
  references them by those names.

## DNS and email (Namecheap)

- **Changed or deleted records take 20–30 minutes** to appear on Namecheap's own
  authoritative servers. Brand-new records appear in seconds. Do not diagnose an
  edit as "unsaved" inside that window — this caused three false alarms.
- The Advanced DNS host-record list **collapses** behind a SHOW MORE control.
  Rows that "disappeared" were hidden, twice.
- MX is Namecheap's Gmail preset (legacy Google set, priorities 1/5/5/10/10).
  Works; leave it alone.
- SPF `-all`, DKIM selector `google`, DMARC `p=reject`, DNSSEC on. Reports go
  to administrator@cfdintl.com; parser and analysis in
  `~/Dropbox/_inbox/dmarc/`.
- **If DNS or registrar ever moves: switch DNSSEC off first and wait 1–2 days
  before changing nameservers.** Details in `DEPLOY.md`.

## Verifying DNS from this Mac

Plain `dig @server` is intercepted locally (see global CLAUDE.md). Use
DNS-over-HTTPS, which cannot be intercepted:

```bash
curl -s "https://dns.google/resolve?name=cfdintl.com&type=TXT" | python3 -m json.tool
```

Add `&do=1` and read the `AD` field to confirm DNSSEC validation.

## GitHub Pages

- Deploys via `.github/workflows/pages.yml` (Actions), not the legacy branch
  builder — the legacy builder failed four times with a generic error.
  Pushing a workflow file needs the `workflow` scope on the `gh` token.
- With Actions deploys the custom domain lives in Pages settings, not the
  `CNAME` file. The file is kept anyway so the domain survives a source change.
- Rapid `curl` sweeps of the live site trigger 503s from the edge. Not a fault.
