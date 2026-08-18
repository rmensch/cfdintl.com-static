# cfdintl.com — static site

A static snapshot of the CFD International website, replacing the Django 1.11
application that previously served it from AWS. Hosted free on GitHub Pages.

Deployment and DNS cutover steps: **[DEPLOY.md](DEPLOY.md)**

## Why static

CFD sold manufacturing rights for its products effective 15 April 2024. The site
is now informational — it presents the product history and directs inquiries to
the new suppliers. Nothing on it needs a database, a login, or server-side code,
so there is no reason to keep paying for a server or to maintain a Django stack
that reached end-of-life in April 2020.

## What this is

Every page reachable by browsing the live site: the home page, 14 product pages
and their sub-pages, About, Capabilities, Contact, and the Supplier Change
Notice — 39 pages in all, plus stylesheets, images, video, and the linked PDF
brochures and terms documents.

Pages that existed in the old codebase but were commented out of the navigation
(`careers`, `affiliates`, `social_media`, `spares`) are **not** included; they
were not reachable by a visitor. `capabilities` *is* included because the About
page links to it in body copy as the "line card."

## Changes made to the mirrored content

Contact details for CFD itself were removed at the owner's request:

- **Footer, sitewide** — the phone number and email were replaced with a plain
  "CFD International" line, so the footer bar does not render empty.
- **Contact page** — the email and phone headings were removed. The page keeps
  its CAGE codes, DUNS numbers, terms-of-sale and terms-of-purchase PDFs, and
  the seven product brochures.
- **Supplier Change Notice** — the line *"Please send inquiries to
  cfdintl@cfdintl.com."* was removed, as it became a dead end. The new
  suppliers' own contact details further down that page are **unchanged**.
- A leftover Google reCAPTCHA script tag was dropped. The contact form itself
  was already absent from production.

No CFD phone number or email address remains anywhere in this repo.

## Editing

Plain HTML and CSS — edit the files directly and push. `index.html` at the root
is the home page; each other page is `<name>/index.html`, which is what keeps
URLs clean (`cfdintl.com/about/`).

Links are root-relative (`/about/`, `/static/...`), which requires the site to
be served from a domain root. That works on `cfdintl.com`; it will look
unstyled on the temporary `rmensch.github.io/cfdintl.com-static/` URL. To
preview locally:

```bash
python3 -m http.server 8080
```

Then open http://localhost:8080.

`.nojekyll` disables GitHub's Jekyll preprocessing, which would otherwise skip
files and folders beginning with an underscore.

## Provenance

Mirrored from the production site with `wget`. The original Django source is at
`~/code/cfdintl.com` and is not needed to run or edit this site.
