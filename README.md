# Our List — setup guide

A one-page to-do list for the two of you, split into three tabs — **Eat**, **Activities**,
and **Go** — each a simple checklist. Log in, check things off, add or remove items;
changes save straight to this GitHub repo. No external services — just GitHub Pages +
the GitHub API.

**Important limits (read once):**
- This is a *soft lock*, not real security. The password check happens in the browser,
  so anyone who really wants to could view page source and see the hash. Fine for a
  personal tool; don't put anything you can't afford a determined person to see.
- There's one shared login for both of you (one email + password you agree on together),
  not separate accounts — simplest for a couple sharing one list.
- Your GitHub token lives only in *your own browser's* local storage — it is never
  written into the site's code/repo. Still, don't sign in to this site on a device you
  don't trust.

## 1. Create the repo

1. On GitHub, create a **new repository** (public is simplest — private repos need a
   paid GitHub plan to use Pages).
2. Upload these three files to the repo root: `index.html`, `data.json`, `README.md`.

## 2. Turn on GitHub Pages

1. In the repo: **Settings → Pages**.
2. Under "Build and deployment", set **Source: Deploy from a branch**.
3. Branch: `main`, folder: `/ (root)`. Save.
4. GitHub gives you a URL like `https://yourname.github.io/your-repo/` — that's your site
   (it can take a minute to go live).

## 3. Set your email + password

Open your browser's console (F12 → Console tab) on any page, and run this, swapping in
the email and password you want to use:

```js
async function hash(s){const b=await crypto.subtle.digest("SHA-256",new TextEncoder().encode(s));return Array.from(new Uint8Array(b)).map(x=>x.toString(16).padStart(2,"0")).join("");}
hash("you@example.com:yourpassword").then(console.log);
```

Copy the printed hash. In `index.html`, find the `CONFIG` block near the bottom and set:

```js
const CONFIG = {
  owner: "your-github-username",
  repo: "your-repo-name",
  path: "data.json",
  branch: "main",
  credentialHash: "PASTE_THE_HASH_HERE"
};
```

Commit that change. (Email is lowercased automatically before hashing, so always type it
in lowercase when signing in.)

## 4. Create a GitHub token (used for saving data)

1. Go to **github.com → Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → Generate new token**.
2. **Repository access**: select "Only select repositories" → choose this one repo.
3. **Permissions**: under "Repository permissions", set **Contents: Read and write**.
   Leave everything else as "No access".
4. Generate, and copy the token (starts with `github_pat_...`). You won't see it again —
   if you lose it, just generate a new one.

## 5. First login

1. Visit your Pages URL.
2. Sign in with the email/password you hashed in step 3.
3. You'll be asked to paste the GitHub token — paste it once. It's saved in that browser
   only, so you'll need to paste it again if you use a different browser or device.
4. You'll see three tabs — **Eat / Activities / Go**. Type in the box at the bottom of
   a tab and hit **Add** to add an item, tick the checkbox to mark it done, or tap ✕ to
   remove it. Everything autosaves to `data.json` in the repo about a second after you
   stop typing. A small "Saved" stamp confirms it.

## Changing the tabs or fields

Each tab stores a plain array of `{ text, notes, done }` objects inside `data.json`,
under the keys `eat`, `activities`, and `places`. To add, remove, or rename a tab:

1. In `index.html`, find the `TABS` array near the top of the `<script>` block and edit
   the `key`/`label`/`placeholder` entries (the `key` must match a key in `data.json`).
2. Update the `<div class="tabs">` buttons in the HTML to match (`data-tab` must match
   the `key`).
3. If you add a new tab key, add it to the `data = { ... }` defaults in `loadData()` and
   to `data.json` itself.

Each item currently has a title and an optional notes line (e.g. an address or a reason
you want to go). To add more fields (like a date or a link), extend the object shape in
`renderList()` similarly to how `notes` works.

## Using it on multiple devices

Instead of embedding the token in the site's code (which would make it visible to
anyone who finds your URL or looks at the repo — never do that), use the **"Copy
device link"** button that appears once you're signed in and connected:

1. On your first device, sign in and paste your token as usual.
2. Tap **Copy device link** — it copies a URL with the token tucked into the part
   after `#`, which browsers never transmit to any server, so it isn't logged
   anywhere.
3. Send that link to your other device privately (e.g. paste it into a note or
   message to yourself) and open it there. It saves the token to that device's
   local storage automatically, then scrubs it from the address bar.
4. Repeat per device. Each device stores its own local copy — there's nothing
   shared or synced beyond that.

Treat that link like a password while it exists in transit (e.g. don't post it
somewhere public) — anyone who opens it gets full read/write access to `data.json`.

## Losing/rotating the token

If a token is ever compromised: GitHub → Settings → Developer settings → Fine-grained
tokens → revoke it, then generate a new one and re-paste it into the site
(sign out and back in to be prompted again, or clear the site's local storage).
