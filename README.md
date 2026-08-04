# The Illuminated Word

A public scripture-sharing platform: a Medium-style feed, a GotQuestions-style
Q&A section, a live Bible reader, and text-to-speech — with likes, comments,
and sharing on every post, and publishing restricted to a single admin
behind real backend authentication. Content is stored in SQLite.

## What's in this folder

```
illuminated-word/
├── index.html          → the public site (visitors)
├── admin.html           → the admin-only login + publishing dashboard
└── word-backend/        → the server that powers both pages
    ├── server.js         (Express API + SQLite: auth, articles, Q&A, pending questions, likes/comments)
    ├── package.json
    ├── hash-password.js  (helper to generate your admin password hash)
    ├── data.json         (legacy seed content — auto-imported into SQLite on first run)
    └── README.md         (backend-specific setup detail)
```

## How it all fits together

- **`index.html`** is read-only for publishing purposes. Visitors can read
  articles, read Q&A, read the Bible (live via a public API), listen to any
  of it via text-to-speech, submit a question through the "Ask" tab, and
  like / comment / share any article, Q&A, or Bible verse. They cannot
  publish articles or Q&A pairs.
- **`admin.html`** is a separate page. You log in with a real username and
  password (checked server-side, hashed with bcrypt, session via JWT) and
  from there can publish articles, publish Q&A directly, answer questions
  visitors submitted, and moderate (delete) any comment site-wide.
- **`word-backend/`** is the only place data is stored and the only thing
  that checks the password. Both HTML pages are just front-ends that talk to
  it over a JSON API. Content lives in a single SQLite file (`data.sqlite`),
  created automatically the first time the server runs.

Nothing works "live" until the backend is deployed somewhere reachable
over the internet — right now both pages point at `http://localhost:4000`,
which only exists on your own machine. Until then, `index.html` shows sample
preview content instead of a blank page, clearly labeled as such, and
likes/comments fall back to a local, non-persistent preview mode.

## Getting it running

### 1. Set up the backend
```bash
cd word-backend
npm install
node hash-password.js "yourStrongPassword"
```
Copy the printed hash. Set these as environment variables wherever you run
or deploy the server:
```
ADMIN_USERNAME=admin
ADMIN_PASSWORD_HASH=<the hash you generated>
JWT_SECRET=<any long random string>
```
Then:
```bash
npm start
```
Full deployment instructions (Render, Railway, Fly.io) — including how to
keep `data.sqlite` from being wiped on redeploy — are in
`word-backend/README.md`.

### 2. Point the front-end at your backend
In both `index.html` and `admin.html`, find this line near the top of the
`<script>` tag:
```js
const API_BASE = 'http://localhost:4000';
```
Change it to your deployed backend's URL, e.g.
`https://illuminated-word-api.onrender.com`.

### 3. Host the two HTML pages
Any static host works — Netlify, Vercel, GitHub Pages, or even just opening
them directly in a browser. Keep `admin.html` unlinked from `index.html`'s
navigation if you don't want visitors to stumble onto the login screen
(currently there is no public link to it by design).

## Feature summary

| Feature | Where | Notes |
|---|---|---|
| Article feed | `index.html` → Read | Medium-style, read-only for visitors |
| Publish articles | `admin.html` | Admin-only |
| Q&A | `index.html` → Questions | GotQuestions-style, read-only for visitors |
| Ask a question | `index.html` → Ask | Public submit, goes to a review queue — not published automatically |
| Answer questions | `admin.html` → Pending Questions | Admin reviews, answers, and publishes or discards |
| Publish Q&A directly | `admin.html` → New Q&A | Admin can also post a Q&A pair without a visitor submission |
| Like, comment, share | Articles, Q&A, Bible verses | Public, no accounts needed. Share copies a deep link; likes/comments stored in SQLite |
| Comment moderation | `admin.html` → Comments | Admin can view and delete any comment across the whole site |
| Bible reader | `index.html` → Bible | Live lookup via bible-api.com, KJV/WEB/BBE, themed decorative background per verse |
| Text-to-speech | Everywhere | Browser's built-in speech synthesis, "Listen" buttons |
| Large print mode | `index.html` nav | Toggles larger, more accessible type sizing |
| Spam protection | Backend | Honeypot field + rate limiting (5 submissions / 15 min / IP) on questions and comments |
| Storage | SQLite (`data.sqlite`) | Real indexed database, safe concurrent writes, handles thousands of posts comfortably |

## Known limitations, honestly

- SQLite is a single file on disk — great up to serious scale, but if this
  ever needs to run across multiple server instances simultaneously or
  handle very heavy concurrent write traffic, that's the point to move to
  Postgres or similar. Ask and I can help with that migration when you're
  there.
- On many free hosting tiers, disk storage is wiped on every redeploy unless
  you attach a persistent volume — see `word-backend/README.md` for how to
  handle that.
- Likes can't be tied to a real person without accounts — the front-end
  disables the like button per-session after one click as a light
  deterrent, but a determined visitor could still like a post more than
  once by reloading. Good enough for a public devotional site, not
  bulletproof.
- There's no image upload — this was intentionally kept to the features
  requested.
- The admin page has no "forgot password" flow — if you lose the password,
  generate a new hash and update the environment variable.
