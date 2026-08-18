# Deploying cfdintl.com

Your live site is untouched until Phase 2. Everything before that is safe.

Order matters here: point DNS at GitHub **first**, then tell GitHub about the
custom domain. Doing it the other way round is what GitHub's own docs warn
against, and it is why the `CNAME` file is not in this repo yet.

---

## Phase 1 — Confirm the site is being served ✅ mostly done

- Repo: https://github.com/rmensch/cfdintl.com-static (public)
- Pages source: branch `main`, folder `/` (root)
- Temporary URL: **https://rmensch.github.io/cfdintl.com-static/**

Open that URL. The pages and text will be correct, but **it will look unstyled
and the images will be missing** — that is expected, not a fault. Every link in
this site is root-relative (`/static/...`), which resolves correctly only at a
domain root. It will render properly the moment it is served at `cfdintl.com`.

To see it styled right now, preview it locally instead:

```bash
cd ~/code/cfdintl.com-static && python3 -m http.server 8080
```

Then open http://localhost:8080.

---

## Phase 2 — Re-point DNS (at Namecheap)

The domain is being transferred from GoDaddy to Namecheap. **Do not start this
phase until the transfer has completed and DNS is confirmed working on
Namecheap's nameservers.** Changing nameservers mid-transfer can disrupt it.

### ⚠️ A registrar transfer does not carry your DNS records

Only the registration moves. GoDaddy stops answering DNS for domains that leave,
so every record has to be recreated at Namecheap — including several that have
nothing to do with this website:

- **`MX` — Google Workspace.** If these are missing when the nameservers flip,
  all inbound company email bounces. Recreate these first and test them before
  touching anything else.
- **`vpn` and `remote`** — both point at the office IP.
- **`dev`** — a second AWS server.
- **`ftp`**.

A full snapshot of the zone as it stood on 2026-08-18, captured from GoDaddy's
authoritative nameservers, is saved outside this repo at:

    ~/Dropbox/_inbox/cfdintl.com DNS snapshot 2026-08-18.md

It is deliberately not in this public repo, because it maps hostnames to the
office IP.

### Do the website cutover as a separate, later step

Get the zone rebuilt at Namecheap with the *existing* values first, confirm
email works, and only then make the two changes below. If something breaks you
will know which change caused it.

### The changes

**1. Apex `A` records** — replace the single record pointing at `54.201.74.233`
with all four GitHub Pages addresses. All four; this is how Pages does failover.

| Type | Host | Value |
|------|------|-------|
| A | @ | `185.199.108.153` |
| A | @ | `185.199.109.153` |
| A | @ | `185.199.110.153` |
| A | @ | `185.199.111.153` |

**2. `www`** — currently a CNAME at `cfdintl.com`. Change its value to:

| Type | Host | Value |
|------|------|-------|
| CNAME | www | `rmensch.github.io` |

Note: `rmensch.github.io` — the account, **not** the repo name.

In Namecheap these live under *Domain List → Manage → Advanced DNS*. Namecheap
uses `@` for the apex and its own TTL control (Automatic is fine).

### Verify

```bash
dig +short cfdintl.com
```

Expect the four `185.199.x.153` addresses. Then confirm email is untouched:

```bash
dig +short MX cfdintl.com
```

Expect the five Google `aspmx` servers. Send yourself a test message from an
outside account before you consider this done.

---

## Phase 3 — Tell GitHub about the domain, then enable HTTPS

Only once `dig` returns the GitHub addresses:

1. Go to **https://github.com/rmensch/cfdintl.com-static/settings/pages**
2. Under **Custom domain**, enter `cfdintl.com` and **Save**

GitHub will run a DNS check, then issue a free Let's Encrypt certificate. That
can take up to an hour. When **Enforce HTTPS** becomes available, tick it.

This is a genuine upgrade: the current site is HTTP-only — port 443 is closed on
`54.201.74.233`, so browsers flag it as *Not secure* today.

Saving the custom domain writes a `CNAME` file back into this repo. If you edit
the site locally afterwards, run `git pull` first so you do not clobber it.

---

## Rollback

Delete the four `185.199.x.153` records, re-add a single apex `A` record
pointing at `54.201.74.233`, and set `www` back to a CNAME at `cfdintl.com`.

Keep the old AWS box running until you have watched the new site for a few days.

---

## Decommissioning afterward

Once you are confident, the AWS instance at `54.201.74.233` can be shut down. It
is the only reason the Django app, its end-of-life Django 1.11 stack, and its
hosting cost still exist. Nothing else depends on it — but confirm the box is
not also doing something unrelated before you terminate it.
