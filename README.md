# report.gershonCRM.com

Front-end for the **Gershon One-Pager Generator** — a tiny static page that takes a Google Drive folder link + recipient email and triggers an n8n agent ("Report Generation Workflow" → "Company one-pager generator") that emails back a documented one-pager PDF.

## Live

- **Production:** https://report.gershoncrm.com
- **Cloudflare Pages target:** `gershoncrm-report.pages.dev`
- **DNS:** `report.gershoncrm.com` CNAME → `gershoncrm-report.pages.dev`
- **Backend:** n8n Form Trigger at `https://gershonconsulting.app.n8n.cloud/form/2d07b6f9-b0a5-401c-876b-28879a1b0c7e`

## Stack

Pure static HTML — Tailwind via CDN, FontAwesome via CDN, no build step. Drops straight into Cloudflare Pages with no framework or build command.

## Local preview

Just open `index.html` in a browser. No dev server required.

```bash
# optional: serve locally on http://localhost:8080
python3 -m http.server 8080
```

## Deploy

Pushing to `main` triggers an auto-deploy via Cloudflare Pages → `gershoncrm-report.pages.dev` → CNAME serves at `report.gershoncrm.com`. Usually live in ~30 seconds.

```bash
git add .
git commit -m "Update report page"
git push origin main
```

## How submission works

The form posts directly to the n8n **Form Trigger** node (no intermediate webhook, no API key in the client). It uses a hidden iframe as the form target so the user stays on `report.gershoncrm.com` while n8n's response loads off-screen, then JS swaps in our own success card.

Field names (the `name` attribute on each input) must match the n8n form-field labels **exactly**, including capitalization and spaces:

| HTML `name`                  | Required | Notes                              |
|------------------------------|----------|------------------------------------|
| `Client name`                | No       | Free text, shown on the PDF header |
| `Client logo URL`            | No       | Direct image URL                   |
| `Google Drive folder URL`    | **Yes**  | Validated against `/folders/<id>`  |
| `Recipient email`            | **Yes**  | `type="email"` + regex validated   |

If you ever rename a field in the n8n Form Trigger, you must update the matching `name="..."` attribute in `index.html`.

## Sibling apps in the gershonCRM suite

- `client.gershoncrm.com` — client management
- `crm.gershoncrm.com` — main CRM
- `dash.gershoncrm.com` — dashboards
- `finance.gershoncrm.com` — Xero finance
- `form.gershoncrm.com` — public intake forms
- `pulse.gershoncrm.com` — activity feed
- `roi.gershoncrm.com` — ROI calculator
- `social.gershoncrm.com` — social tracking
- `task.gershoncrm.com` — tasks
- `company.gershoncrm.com` — company directory
