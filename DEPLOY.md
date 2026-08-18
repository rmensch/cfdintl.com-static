# Deploying cfdintl.com

Two phases: publish to GitHub Pages, then re-point DNS at GoDaddy.
Phase 1 is safe and reversible. Phase 2 is the live cutover.

---

## Phase 1 — Publish to GitHub Pages

The repo is already created and pushed. To turn on hosting:

1. Go to **https://github.com/rmensch/cfdintl.com-static/settings/pages**
2. Under **Build and deployment → Source**, choose **Deploy from a branch**
3. Branch: **main**, folder: **/ (root)**, then **Save**
4. Under **Custom domain**, enter `cfdintl.com` and **Save**

GitHub will report *"Domain's DNS record could not be verified"* — expected until
Phase 2. The site is live in the meantime at the temporary URL Pages shows you.

> The repo already contains a `CNAME` file with `cfdintl.com`, so step 4 may be
> pre-filled. Leave it as is.

---

## Phase 2 — Re-point DNS at GoDaddy

Your nameservers are `ns29/ns30.domaincontrol.com` (GoDaddy). Manage records at
**https://dcc.godaddy.com/control/portfolio/cfdintl.com/settings** → *DNS*.

### ⚠️ Do not touch the MX records

`cfdintl.com` email runs on **Google Workspace** (`aspmx.l.google.com` and the
four `alt*` servers). Changing or deleting those stops company email. Only touch
the `A` and `www` records described below. Leave every other record — MX, TXT,
SPF, DKIM, verification records — exactly as they are.

### Optional but recommended: lower the TTL first

A day before cutover, set the TTL on the existing apex `A` record to **600
seconds**. This makes a rollback take minutes instead of hours. Skip only if you
are comfortable with a slower undo.

### The changes

**1. Delete the existing apex A record**

| Type | Name | Value | Action |
|------|------|-------|--------|
| A | @ | `54.201.74.233` | delete |

**2. Add four new apex A records** (all four — this is how Pages does failover)

| Type | Name | Value |
|------|------|-------|
| A | @ | `185.199.108.153` |
| A | @ | `185.199.109.153` |
| A | @ | `185.199.110.153` |
| A | @ | `185.199.111.153` |

**3. Change the `www` record**

It is currently a CNAME pointing at `cfdintl.com`. Repoint it:

| Type | Name | Value |
|------|------|-------|
| CNAME | www | `rmensch.github.io` |

Note the trailing `.github.io` — **not** the repo name.

---

## Phase 3 — Verify, then enforce HTTPS

Propagation typically takes 15–60 minutes.

Check from your terminal:

```bash
dig +short cfdintl.com
```

You want the four `185.199.x.153` addresses. Then confirm email is untouched:

```bash
dig +short MX cfdintl.com
```

You want the five Google `aspmx` servers, unchanged.

Once DNS resolves, return to **Settings → Pages**. GitHub will validate the
domain and issue a free Let's Encrypt certificate (can take up to an hour).
When the **Enforce HTTPS** checkbox becomes available, tick it.

This is a real upgrade: your current site is HTTP-only — port 443 is closed on
`54.201.74.233`, so browsers flag it as *Not secure* today.

---

## Rollback

Delete the four `185.199.x.153` A records, re-add a single apex `A` record
pointing to `54.201.74.233`, and set `www` back to a CNAME at `cfdintl.com`.
If you lowered the TTL first, this takes effect within minutes.

Keep the old AWS box running until you have watched the new site for a few days.

---

## Decommissioning afterward

Once you are confident, that AWS instance at `54.201.74.233` can be shut down —
it is the only reason the Django app, its EOL Django 1.11 stack, and its server
costs still exist. Nothing else depends on it.
