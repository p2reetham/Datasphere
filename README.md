# Data Sphere Club — website

Static site (index.html — image, fonts and styles all inlined) plus one
serverless function (`api/config.js`) that gives you a single global
admin password and shared content (Google Form link, gallery, mission/about
text) that updates on every visitor's device, not just your own.

## Deploy on Vercel

1. Go to vercel.com → Add New → Project → import this folder (or drag-and-drop
   it in the dashboard).
2. Framework preset: "Other" — no build step needed.
3. Deploy once first (it'll work immediately in browser-only fallback mode —
   see below). Then follow the steps under "Enable the global admin backend"
   and redeploy to switch on the shared password + shared content.

## Enable the global admin backend (one password, synced across every device)

This uses a GitHub repo as a tiny JSON database and a Vercel serverless
function as the only thing allowed to write to it — your GitHub token never
reaches the browser, so it can't be read from the page source.

1. **Create a GitHub repo** (private is fine, e.g. `datasphere-config`).
2. **Add a file** named `config.json` at the root of that repo with:
   ```json
   {
     "formUrl": "https://forms.gle/REPLACE_WITH_YOUR_GOOGLE_FORM_LINK",
     "gallery": [],
     "mission": "The Department of Artificial Intelligence & Data Science is built to turn curious students into confident practitioners...",
     "about1": "Data Sphere is the student-run club of the AI & DS department..."
   }
   ```
3. **Create a GitHub token**: GitHub → Settings → Developer settings →
   Fine-grained tokens → generate one scoped only to the `datasphere-config`
   repo, with **Contents: Read and write** permission.
4. In your **Vercel project → Settings → Environment Variables**, add:
   | Name | Value |
   |---|---|
   | `GITHUB_TOKEN` | the token from step 3 |
   | `GITHUB_OWNER` | your GitHub username or org |
   | `GITHUB_REPO` | `datasphere-config` |
   | `GITHUB_PATH` | `config.json` |
   | `GITHUB_BRANCH` | `main` |
   | `ADMIN_PASSWORD` | `preetham` |
5. **Redeploy** (env var changes need a redeploy to take effect).

Once redeployed: everyone who opens the live URL loads the shared
`config.json` on page load. Clicking the gear icon anywhere now always asks
for the same password (`preetham`, or whatever you set `ADMIN_PASSWORD` to).
A successful save commits the update straight to `config.json` on GitHub —
so it's the same content and the same password on every device, no local
per-browser state involved.

To change the password later, just update `ADMIN_PASSWORD` in Vercel's
environment variables and redeploy — there's no UI for it, by design, so it
can't be read out of the page source.

## If you skip the steps above

The site still works without any of this — the gear icon falls back to
per-browser storage (each device sets and remembers its own password and
content, not shared with other visitors). That's fine for testing, but for
one shared password and content synced everywhere, do the setup above.

## Editing content without any admin panel at all

You can also just hardcode things directly in `index.html` before deploying:
search for `REPLACE_WITH_YOUR_GOOGLE_FORM_LINK` for the form link, `gallery: []`
in the `<script>` for photos, and the `#mission` / `#about` sections in the
HTML for the text. Redeploy and it's permanent for everyone, no backend needed.
