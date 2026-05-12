# Deploy report.gershoncrm.com

**Current state:** Preview deploy — the form is fully designed but the n8n backend is *not* connected yet. Submission shows a "We're putting the finishing touches" message. To re-enable the live backend later, see **§ Re-enable n8n** at the bottom.

---

## Phase 1 — Push the initial commit to GitHub

Open PowerShell in `C:\Users\oatti\Documents\Claude\Projects\Report.gershonCRM.com` and run:

```powershell
# Show hidden items so you can see the .git folder
# (only needed once — turn it back off later if you like)
attrib -h .git 2>$null

# A previous session may have left a partial .git directory — clear it.
if (Test-Path .git) { Remove-Item -Recurse -Force .git }

# Fresh repo
git init -b main
git add .
git commit -m "Initial commit: report.gershoncrm.com preview (n8n backend not yet connected)"
git remote add origin https://github.com/gershonconsulting/report.git
git push -u origin main
```

If `git push` prompts for credentials, use a GitHub Personal Access Token with `repo` scope. Or if you have `gh` installed:

```powershell
gh auth login
git push -u origin main
```

After this, https://github.com/gershonconsulting/report should show all 5 files: `index.html`, `README.md`, `.gitignore`, `_headers`, `DEPLOY.md`.

---

## Phase 2 — Create the Cloudflare Pages project

Goal: `gershoncrm-report.pages.dev` should serve `index.html` whenever you push to `main`.

1. Cloudflare dashboard → **Workers & Pages** → **Create application** → **Pages** tab → **Connect to Git**.
2. Authorize the **gershonconsulting** GitHub org if prompted, then pick the **report** repository.
3. **Project name:** `gershoncrm-report` — this MUST match the CNAME target. Your DNS already has `report.gershoncrm.com → gershoncrm-report.pages.dev`; if you pick a different name, the custom domain won't resolve.
4. **Production branch:** `main`.
5. **Build settings:**
   - **Framework preset:** *None*
   - **Build command:** *(leave empty)*
   - **Build output directory:** `/` *(or leave default)*
6. **Save and Deploy.**

First build runs instantly (no build step). `https://gershoncrm-report.pages.dev` should be live in under a minute. Open it — you should see the form.

---

## Phase 3 — Bind the custom domain

The CNAME `report.gershoncrm.com → gershoncrm-report.pages.dev` is already in your DNS panel, but Cloudflare Pages still needs to know the project owns that hostname.

1. In the `gershoncrm-report` Pages project → **Custom domains** → **Set up a custom domain**.
2. Enter `report.gershoncrm.com` → **Continue** → **Activate domain**.
3. Cloudflare detects the existing CNAME and provisions SSL automatically. Wait ~30 seconds.
4. https://report.gershoncrm.com should now load with a valid green padlock.

---

## Phase 4 — Verify

- [ ] `https://report.gershoncrm.com` loads the form
- [ ] Filling it in and clicking **Generate one-pager** shows the "We're putting the finishing touches" view
- [ ] No redirect loops, no cert warnings
- [ ] Mobile view renders correctly (headline shrinks, info grid stacks)

---

## § Re-enable n8n (when backend is ready)

In `index.html`, restore three things:

**A. Add the form action attributes back:**

```html
<form
  id="reportForm"
  action="https://gershonconsulting.app.n8n.cloud/form/2d07b6f9-b0a5-401c-876b-28879a1b0c7e"
  method="POST"
  enctype="multipart/form-data"
  target="n8nSubmitFrame"
  novalidate
>
```

**B. Restore the success messaging** in the `#successView` section — change the headline back to "Your one-pager is on its way." and the lede to mention the agent reading the folder.

**C. Replace the preview submit handler** with the real iframe-based version:

```js
form.addEventListener('submit', (e) => {
  let ok = true;
  const folderUrl = folderInput.value.trim();
  const email = emailInput.value.trim();
  const clientName = clientInput.value.trim();
  if (!validateDriveUrl(folderUrl)) { showFieldError(folderShell, folderError, "..."); ok = false; }
  if 