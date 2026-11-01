# Deploy LeadPro for the next law firm (under 30 minutes)

New client repos live on the personal account **peejayedz** and are named `leadpro-kpi-<client>` (lowercase, hyphens, e.g. `leadpro-kpi-speaks`). This repo, `peejayedz/leadpro-kpi-mahdavi`, is the template.

Before you start, have these ready:
- The client's GHL **sub-account name** and **location ID**.
- A **GHL Private Integration Token** from that sub-account (Settings → Private Integrations → Create new integration), with every scope in the README's scope table. Include the conversations/messages read scope, otherwise speed-to-lead and first contact stay blank and the sync log shows `401 … missing a scope`.
- A **dashboard passphrase** for the client (a long random phrase). It unlocks names in the drill-downs.

Never paste the token or passphrase into a file, commit, issue or chat. They go only into the repo's Actions secrets and the GHL menu link.

## Checklist

1. **Create the repo from the template**
   ```
   cd "<your LFMP folder>/assets/code"
   gh repo create peejayedz/leadpro-kpi-<client> --public --template peejayedz/leadpro-kpi-mahdavi --clone
   cd leadpro-kpi-<client>
   ```
2. **Remove Mahdavi's data.** The template carries Mahdavi's synced files, which must not ship with another client (the first sync recreates `data/`):
   ```
   git rm -r data
   ```
3. **Set the location ID in the workflow.** In `.github/workflows/sync-kpis.yml`, change the default in
   `GHL_LOCATION_ID: ${{ vars.GHL_LOCATION_ID || 'INZKoa3nu45xtcZB0Yuq' }}` to the new client's location ID.
4. **Replace `config.json`.** Start from this repo's `config.json` and change at least:
   - `locationName` and `title` (e.g. `"Speaks KPI Scorecard"`)
   - `notice` (keep the "Testing period" banner for the first few weeks, or set it to `null`)
   - `pipelines`, `stages`, `turnedAway`, `tags`: leave them for now; step 9 fills them from the log
   - `booking.calendarKeywords`: `["consult"]` for now
   - `targets` and `adSpend.monthly`, if the client gave you goals or spend
5. **Update `README.md`**: firm name in the title and intro, and the location ID (appears twice).
6. **Commit and push**
   ```
   git add -A && git commit -m "Set up LeadPro scorecard for <Client>" && git push
   ```
7. **Add the secrets** at `https://github.com/peejayedz/leadpro-kpi-<client>/settings/secrets/actions`:
   - `GHL_TOKEN`: the Private Integration Token
   - `DASHBOARD_PASSPHRASE`: the passphrase
   - `DISPATCH_TOKEN` (optional, enables the Refresh data button): a fine-grained GitHub token, only this repo, Actions: Read and write
   
   Check (names only, never values): `gh secret list -R peejayedz/leadpro-kpi-<client>`
8. **Enable GitHub Pages and run the first sync**
   ```
   gh api -X POST repos/peejayedz/leadpro-kpi-<client>/pages -f 'source[branch]=main' -f 'source[path]=/'
   gh workflow run sync-kpis.yml -R peejayedz/leadpro-kpi-<client>
   gh run watch -R peejayedz/leadpro-kpi-<client>
   ```
   If it fails, open the log (`gh run view --log-failed -R …`). A 401 means a wrong token or a missing scope.
9. **Read the stage and calendar names from the log**
   ```
   gh run view -R peejayedz/leadpro-kpi-<client> --log | grep -A3 -E "Pipelines and stages|Calendars|Attribution coverage|Wrote data"
   ```
10. **Fix `config.json` to match the log exactly**, character for character:
    - `pipelines`: the pipeline that holds new-client opportunities
    - `stages`, `turnedAway.stages`, `turnedAway.reasonLabels`: the exact stage names
    - `booking.calendarKeywords`: a word from the consultation calendar's name, or `[]` if there are no calendars (every calendar counts)
11. **Commit, push and rerun the sync**, then confirm the log's stage names and counts look right.
12. **Check the live page** after about a minute: `https://peejayedz.github.io/leadpro-kpi-<client>/`. It should load with the client's title and no errors. Numbers count leads by creation date, so a quiet 30 days can show low totals even when the log says many leads were pulled.
13. **Add the Custom Menu Link in GHL** in the client's sub-account (check the location ID in the URL and the sub-account name on screen first): Settings → Custom Menu Links → Add.
    - Name: `KPI Dashboard`
    - URL: `https://peejayedz.github.io/leadpro-kpi-<client>/#k=<DASHBOARD_PASSPHRASE>`
    - URL-encode the passphrase: `#` becomes `%23`, space `%20`, `&` `%26`, `?` `%3F`, `/` `%2F`, `%` `%25`
    - Open in: iframe
14. **Open the menu link in GHL** and confirm the dashboard loads and "Unlock names" works.
