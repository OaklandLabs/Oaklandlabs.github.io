# Oakland Labs Website

Static marketing + compliance website for **Oakland Labs LLC** and its apps — **Provenant**
(a collection-management app for sports memorabilia and autographs) and **Timepiece Tracker**
(a watch-collection tracker, coming soon). Hosted on **GitHub Pages** at the custom domain
**[oaklandlabs.app](https://oaklandlabs.app)**.

Each app owns a directory (`provenant/`, `timepiece/`) holding its own landing page, legal pages,
and deep-link/email landing pages. The root-level `privacy.html` / `terms.html` /
`delete-account.html` predate that split and cover Provenant; the app-scoped copies under
`timepiece/` are the ones the Timepiece Tracker app links to.

There is **no build step** — the site is hand-authored HTML/CSS (all styles are inlined per page).
Edit the files, push to `main`, and GitHub Pages redeploys automatically.

## Architecture

```
                              ┌─────────────────────────────┐
                              │   Visitor's browser / app   │
                              └──────────────┬──────────────┘
                                             │  https://oaklandlabs.app
                                             ▼
                       ┌───────────────────────────────────────────┐
                       │              GitHub Pages                  │
                       │   (repo: Oaklandlabs.github.io, branch     │
                       │    main; CNAME → oaklandlabs.app;          │
                       │    .nojekyll = serve files as-is)          │
                       └──────────────────────┬────────────────────┘
                                              │
      ┌──────────────────┬────────────────────┼───────────────────┬─────────────────────┐
      ▼                  ▼                     ▼                   ▼                     ▼
┌───────────┐   ┌─────────────────┐   ┌────────────────┐  ┌──────────────────┐  ┌──────────────┐
│ Marketing │   │ Legal /         │   │ App-store      │  │ Deep-link        │  │ Site config  │
│           │   │ compliance      │   │ redirect       │  │ landing pages    │  │ & crawl      │
│ index.html│   │ privacy.html    │   │ provenant/     │  │ provenant/join   │  │ robots.txt   │
│ 404.html  │   │ terms.html      │   │  index.html    │  │ provenant/profile│  │ sitemap.xml  │
│           │   │ delete-account  │   │ (UA-sniffs →   │  │ provenant/       │  │ app-ads.txt  │
│           │   │  .html          │   │  store)        │  │  unsubscribe     │  │ .well-known/ │
└───────────┘   └─────────────────┘   └───────┬────────┘  └────────┬─────────┘  └──────┬───────┘
                                              │                    │                   │
                                              ▼                    ▼                   ▼
                                     ┌─────────────────┐  ┌──────────────────┐ ┌─────────────────┐
                                     │ App Store /     │  │ provenant://     │ │ Universal Links │
                                     │ Google Play     │  │ deep links into  │ │ (iOS) + App     │
                                     │                 │  │ the mobile app   │ │ Links (Android) │
                                     └─────────────────┘  └──────────────────┘ └─────────────────┘
                                                                   │
                       unsubscribe page ─────────────────────────►│
                       POSTs to Supabase edge function            ▼
                       (project ref xbwdrotfwiojupjphjzt)  ┌──────────────────┐
                                                           │ Supabase backend │
                                                           │ (email alerts)   │
                                                           └──────────────────┘

  Cookieless web analytics (Cloudflare Web Analytics) beacon is loaded on every top-level page.

  The timepiece/ directory mirrors this shape for the second app: timepiece/index.html
  (landing), timepiece/{privacy,terms,delete-account}/ (legal), and timepiece/unsubscribe/
  (POSTs to the *Timepiece* Supabase project, ref zdcbwdyxlfgmfdowjamf).
```

## Pages

| Path | Purpose |
|------|---------|
| `index.html` | Marketing homepage — hero, Provenant overview (available now), Timepiece Tracker (coming soon), feature grid, about + contact. |
| `privacy.html` | Privacy Policy (CCPA/GDPR). Required by App Store & Play. |
| `terms.html` | Terms of Service. |
| `delete-account.html` | Account/data deletion instructions — **required for Google Play** compliance. |
| `404.html` | Branded not-found page. GitHub Pages auto-serves it on unknown paths. |
| `provenant/index.html` | "Get Provenant" — sniffs the user agent and redirects to the App Store (iOS) or Google Play (Android); shows both buttons on desktop. |
| `provenant/about/index.html` | Provenant marketing landing page — available-now badge, feature grid, links to its legal pages. |
| `provenant/join/index.html` | Group-invite landing. Reads a 6-char `?code=`, opens `provenant://groups/join?code=…`, falls back to download. |
| `provenant/profile/index.html` | Public-profile landing. Reads `?u=<username>`, opens `provenant://profile/<username>`, falls back to download. |
| `provenant/unsubscribe/index.html` | One-click email unsubscribe. POSTs the signed `?token=` to a Supabase edge function; supports re-subscribe. |
| `timepiece/index.html` | Timepiece Tracker landing page — coming-soon badge, feature grid, links to its legal pages. |
| `timepiece/privacy/index.html` | Privacy Policy for Timepiece Tracker. Linked from in-app Settings and the paywall (`src/config/links.ts` → `PRIVACY_URL`). |
| `timepiece/terms/index.html` | Terms of Service for Timepiece Tracker (`TERMS_URL`). |
| `timepiece/delete-account/index.html` | Account/data deletion instructions for Timepiece Tracker — required for Google Play. |
| `timepiece/unsubscribe/index.html` | One-click unsubscribe for the Timepiece wishlist digest. Same flow as Provenant's, pointed at the Timepiece Supabase project. |

> Paths under `timepiece/` are directories with an `index.html`, so they resolve extensionless
> (`/timepiece/privacy`) — that's the exact form the app and edge functions link to.

## Integrations & configuration

| File / item | Role |
|-------------|------|
| `CNAME` | Maps GitHub Pages to `oaklandlabs.app`. |
| `.nojekyll` | Disables Jekyll processing so files are served exactly as committed. |
| `.well-known/apple-app-site-association` | iOS Universal Links — associates `/provenant/join` and `/provenant/profile` paths with the app (`3MV66VKC53.com.provenant.app`). |
| `.well-known/assetlinks.json` | Android App Links — verifies `com.provenant.app` via signing-key fingerprints. |
| `app-ads.txt` | Authorizes the AdMob publisher account for in-app ads. |
| `provenant://` | Custom deep-link scheme used by the join/profile landing pages. |
| Supabase (`xbwdrotfwiojupjphjzt`) | **Provenant** backend — powers `/provenant/unsubscribe`. |
| Supabase (`zdcbwdyxlfgmfdowjamf`) | **Timepiece Tracker** backend (separate project) — powers `/timepiece/unsubscribe`. |
| Cloudflare Web Analytics | Cookieless, privacy-first traffic analytics. Snippet is staged (commented out) on every top-level page — paste your Site Token and uncomment to activate. |

## How sign-ups work

There is **no waitlist / email-capture form** on the site. An earlier version had one, but it
posted to an unconfigured placeholder endpoint (`formspree.io/f/YOUR_FORM_ID`), so no submissions
were ever captured or viewable — it has been removed.

If you want to collect launch sign-ups later, wire a form to one of:

- **Supabase** (recommended for consistency) — the same backend already powering the unsubscribe
  flow. Add a table + a small edge function; submissions are then viewable in the Supabase
  dashboard / queryable via SQL.
- **Formspree** or similar — quickest path; create a form, drop in the real form ID, and
  submissions arrive by email and in the Formspree dashboard.

Whichever you choose, replace the placeholder before shipping so sign-ups actually land somewhere.

## Local development & deploy

```bash
# Serve locally from the repo root
python3 -m http.server 8000
# → open http://localhost:8000
```

Deploy by pushing to `main`; GitHub Pages rebuilds and serves within a minute. No compilation,
bundler, or dependencies.
