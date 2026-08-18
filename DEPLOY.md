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

## Phase 2 — Re-point DNS at GoDaddy

Your nameservers are `ns29/ns30.domaincontrol.com` (GoDaddy). Manage records at
**https://dcc.godaddy.com/control/portfolio/cfdintl.com/settings** → *DNS*.

### ⚠️ Do not touch the MX records

`cfdintl.com` email runs on **Google Workspace** (`aspmx.l.google.com` plus the
four `alt*` servers). Changing or deleting those stops company email. Touch only
the `A` and `www` records below. Leave everything else — MX, TXT, SPF, DKIM,
domain-verification records — exactly as it is.

### Optional: lower the TTL a day ahead

Set the TTL on the existing apex `A` record to **600 seconds** the day before.
That makes a rollback take minutes instead of hours.

### The changes

**1. Delete the existing apex `A` record**

| Type | Name | Value | Action |
|------|------|-------|--------|
| A | @ | `54.201.74.233` | delete |

**2. Add four new apex `A` records** — all four; this is how Pages does failover

| Type | Name | Value |
|------|------|-------|
| A | @ | `185.199.108.153` |
| A | @ | `185.199.109.153` |
| A | @ | `185.199.110.153` |
| A | @ | `185.199.111.153` |

**3. Repoint `www`**

It is currently a CNAME at `cfdintl.com`. Change its value to:

| Type | Name | Value |
|------|------|-------|
| CNAME | www | `rmensch.github.io` |

Note: `rmensch.github.io` — the account, **not** the repo name.

### Verify propagation

```bash
dig +short cfdintl.com
```

You want the four `185.199.x.153` addresses. Then confirm email is intact:

```bash
dig +short MX cfdintl.com
```

You want the five Google `aspmx` servers, unchanged.

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
If you lowered the TTL first, this takes effect within minutes.

Keep the old AWS box running until you have watched the new site for a few days.

---

## Decommissioning afterward

Once you are confident, the AWS instance at `54.201.74.233` can be shut down. It
is the only reason the Django app, its end-of-life Django 1.11 stack, and its
hosting cost still exist. Nothing else depends on it — but confirm the box is
not also doing something unrelated before you terminate it.
