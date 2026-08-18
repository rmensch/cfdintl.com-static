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
and their sub-pages, About, Capabilities, and the Supplier Change Notice — 39
pages in all, plus stylesheets, images, video, and the PDF brochures and terms
documents.

Pages that existed in the old codebase but were commented out of the navigation
(`careers`, `affiliates`, `social_media`, `spares`) are **not** included; they
were not reachable by a visitor. `capabilities` *is* included because the About
page links to it in body copy as the "line card."

## Changes made to the mirrored content

Contact details for CFD itself were removed at the owner's request:

- **Footer, sitewide** — the phone number and email were replaced with a plain
  "CFD International" line, so the footer bar does not render empty.
- **Contact page** — removed entirely, along with its navigation link. With the
  email and phone gone there was nothing left on it but a documents panel, and
  the owner asked for that to go too. Note this leaves the PDFs in
  `static/docs/` unlinked from anywhere: the CAGE/DUNS details, the two terms
  documents, and the seven product brochures are all still served, just no
  longer reachable by browsing. Restore with `git revert` if that was not
  intended.
- **Supplier Change Notice** — the line *"Please send inquiries to
  cfdintl@cfdintl.com."* was removed, as it became a dead end. The new
  suppliers' own contact details further down that page are **unchanged**.
- A leftover Google reCAPTCHA script tag was dropped. The contact form itself
  was already absent from production.

No CFD phone number or email address remains anywhere in this repo.

## One fix beyond the mirror

`project.css` sets `body { margin-top: 51px }` to clear the fixed navbar. The
Supplier Change Notice banner was later added *inside* that navbar, roughly
doubling its height, but the 51px was never increased — so on every page the
first heading sat underneath the header, half-hidden. This is broken on the
live Django site today.

Each page now carries a short script before `</body>` that measures the navbar
and sets the body offset to match. Measuring at runtime rather than hard-coding
a value keeps it correct when the banner wraps to two lines on a narrow screen,
and it self-corrects if the banner is ever removed.

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
