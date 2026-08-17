# George Parker Young, PC — Website

A fast, dependency-free static site (plain HTML/CSS/JS) with a Git-backed
Journal (blog). No framework, no build step. It runs on any static host;
these notes assume **Cloudflare Pages**.

```
gpylaw-site/
├─ index.html            The whole site (home, practice, about, honors, journal, contact)
├─ content/
│  └─ posts.json         The Journal's content — edited by the CMS
├─ admin/
│  ├─ index.html         The CMS (loads Decap CMS)
│  └─ config.yml         CMS configuration  ← edit 2 placeholders before go-live
├─ assets/uploads/       Images uploaded via the CMS land here
├─ _headers              Tells Cloudflare not to cache posts.json
└─ .gitignore
```

The Journal loads from `content/posts.json` at runtime. If that file can't be
reached (e.g. you double-click `index.html` locally), the page falls back to an
identical copy embedded in the HTML, so it always renders.

---

## 1. Put it live on Cloudflare Pages (~5 min)

1. Create a new Git repo (GitHub is easiest for the CMS later) and push this folder to it.
2. In the Cloudflare dashboard: **Workers & Pages → Create → Pages → Connect to Git**, pick the repo.
3. Build settings: **Framework preset = None**, **Build command = (leave blank)**,
   **Build output directory = `/`** (the repo root). Save and deploy.
4. You'll get a `*.pages.dev` URL. That's the live site.

That's the whole deploy. Every `git push` re-publishes automatically.

## 2. Point gpylaw.com at it

1. In your Pages project → **Custom domains → Set up a domain** → enter `gpylaw.com` (and `www`).
2. Follow the DNS prompt. If the domain is on Cloudflare, records are added for you;
   if it's elsewhere (or still on Wix), update the nameservers/CNAME as instructed.
3. Wait for SSL to provision (usually minutes). Done.

> Moving off Wix: keep the old site up until DNS has fully switched, then cancel Wix.

## 3. Turn on the Journal CMS (~10 min, one-time)

The site works without this — you can also just edit `content/posts.json` by hand.
But this gives George a real login-and-write editor at `gpylaw.com/admin/`.

Cloudflare Pages has no built-in login relay (Netlify does), so you add a tiny
OAuth worker once:

1. **GitHub OAuth app** — GitHub → *Settings → Developer settings → OAuth Apps → New*.
   - Homepage URL: `https://gpylaw.com`
   - Authorization callback URL: your worker URL + `/callback` (from the next step)
   - Note the **Client ID** and generate a **Client Secret**.
2. **Deploy an auth worker** — the community `sveltia-cms-auth` Cloudflare Worker
   works with Decap too (find "sveltia-cms-auth" on GitHub; it has a one-command
   deploy). Set `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` as its secrets.
   Copy the worker's `*.workers.dev` URL.
3. **Edit `admin/config.yml`** — set the two `<-- EDIT` placeholders:
   - `repo:` → `your-github-username/gpylaw-site`
   - `base_url:` → your worker URL
4. Commit, push. Visit `https://gpylaw.com/admin/`, log in with GitHub, and publish.

Saving in the CMS commits to `content/posts.json` → Cloudflare rebuilds → the post
is live in under a minute.

### Prefer to try the editor first, with zero setup?
`admin/config.yml` already has `local_backend: true`, so on your own machine:

```bash
npx decap-server            # in one terminal (from this folder)
npx serve .                 # in another (serves the site at http://localhost:3000)
```
Open `http://localhost:3000/admin/` — edit freely, no auth, changes write to your
local `content/posts.json`.

### Want a nicer editor UI?
Swap Decap for **Sveltia CMS** (drop-in, same `config.yml`): in `admin/index.html`
replace the Decap script tag with
`<script src="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.js"></script>`.

---

## Editing posts by hand

`content/posts.json` is a plain list. Newest first. Each post:

```json
{
  "title": "Ninth Update: Back in Warsaw",
  "cat": "Dispatches",
  "author": "George Parker Young",
  "date": "August 2022",
  "read": "6 min read",
  "excerpt": "One sentence shown on the card.",
  "body": "First paragraph.\n\n## A subheading\n\nNext paragraph."
}
```

Body rules: blank line = new paragraph; a line starting with `## ` becomes a subheading.

---

## Things to confirm before launch

- **Post dates.** The four migrated posts are George's 2022 dispatches; the old Wix
  export mislabeled them "Mar 20, 2025." They're set to May/June 2022 here — confirm exact dates.
- **2026 Best Lawyers.** Verify the exact practice-area category on the Honors seal.
- **Earlier updates.** These are the Fifth–Eighth. If First–Fourth exist, add them via the CMS.
- **George's photo.** The bio expects a headshot at `assets/george-young.jpg` (save the one
  from the current site into that path). Until it's there, the bio shows a "GPY" monogram frame.
- **Contact form.** The form is front-end only. Wire it to a handler (Cloudflare
  Pages Functions, Formspree, or similar) so submissions reach the firm's inbox.
