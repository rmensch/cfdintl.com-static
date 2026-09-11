# 0001 — Replace the Django app with a static mirror of production

**Date:** 2026-08-18  **Status:** accepted, implemented

## Context

cfdintl.com ran on Django 1.11 (end-of-life April 2020) on an AWS box, HTTP
only, with credentials hardcoded in `settings.py`. CFD sold its product lines
in April 2024; the site is now informational. The owner wanted free hosting,
fast, with no learning curve.

## Options

1. **Mirror the live site with `wget` and host the output on GitHub Pages.** Chosen.
2. Upgrade Django to a supported version and keep hosting it. Rejected: `django.conf.urls.url`
   and other 1.x APIs are gone in Django 4+, so every URL and template needs
   rework, and it still costs a server for a site with no dynamic content.
3. Rebuild with a static-site generator. Rejected: same output as option 1 for
   far more work, and the owner does not want to maintain a build toolchain.

## Decision details

- Mirrored **production**, not the local dev server. Production hides some
  elements via a `not-prod` CSS class and serves PageSpeed-optimised assets;
  the local checkout would have exposed things visitors never see.
- Kept PageSpeed's mangled asset filenames rather than de-optimising. The
  original unoptimised images were never downloaded, so it was not possible
  without a second mirror pass.
- Included only pages reachable by browsing. `capabilities` is included because
  the About page links to it in body copy; `careers`, `affiliates`,
  `social_media`, `spares` are excluded (commented out of the nav).
- CFD's own contact details removed sitewide at the owner's request; the new
  suppliers' details on the Supplier Change Notice page preserved.
- One deliberate fix beyond the mirror: page headings sat under the fixed
  navbar on every page (the banner had been added without adjusting the body
  offset). A runtime script now measures the navbar and sets the offset.

## Consequences

- Site is now HTTPS (was HTTP-only), IPv6, and costs nothing to host.
- The contact form is gone; the domain is receive-only for email.
- The old repo at `~/code/cfdintl.com` is source-only and should not be run.
