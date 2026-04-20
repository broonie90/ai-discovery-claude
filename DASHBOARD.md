# Claude AI Usage — Dashboard Assembly

The package ships the reusable PA atoms — indicators, cubes, breakdowns,
units, widgets. The dashboard layout itself is best assembled in the UI
(the XML chain for a fully declarative dashboard is ~30 fragile records
and ServiceNow's own guidance is UI + Source Control capture).

## What ships in the package

### Indicators (7)
Visible under **Performance Analytics → Indicators → search "Claude —"**.

| Indicator | Aggregate | Table | Chart |
|---|---|---|---|
| Claude Model — Input Tokens (Daily)        | SUM input_tokens      | model_usage      | line |
| Claude Model — Output Tokens (Daily)       | SUM output_tokens     | model_usage      | line |
| Claude Model — Cost USD (Daily)            | SUM cost_usd          | model_usage      | line |
| Claude Skill — Invocations (Daily)         | SUM invocation_count  | skill_usage      | line |
| Claude Skill — Distinct Users (Daily)      | SUM distinct_users    | skill_usage      | line |
| Claude Connector — Invocations (Daily)     | SUM invocation_count  | connector_usage  | line |
| Claude Connector — Distinct Users (Daily)  | SUM distinct_users    | connector_usage  | line |

### Breakdowns (3)
- **By AI Model** — slices model_usage rows on `x_agiro_ai_disc_cl_ai_model`
- **By Skill** — slices skill_usage rows on `x_agiro_ai_disc_cl_skill`
- **By Connector** — slices connector_usage rows on `x_agiro_ai_disc_cl_connector`

### Widgets (16)
5 scorecards · 7 trend lines · 4 column-breakdown charts.

### Units (2)
Tokens and USD, so chart labels read properly.

## First-time activation

After pulling the app from GitHub onto a fresh instance:

1. **Backfill scores** — for each indicator, click `Performance Analytics → Indicators`, open the indicator, click **Collect Scores**. First run backfills all historical days from `usage_date`. Subsequent daily runs are automatic (picked up by the OOB Daily data collector job at ~01:00 UTC).

2. **Verify data** — `Performance Analytics → Scores → filter "indicator = Claude ..."`. Should show one score row per `usage_date`.

3. **Assemble the dashboard** (15 minutes in the UI):
   - `Performance Analytics → Dashboards → + New`
   - Name: `Claude AI Adoption`
   - Add a tab: `Overview`
   - Drop the **5 scorecards** across the top row
   - Drop the **4 column/breakdown charts** below
   - Drop the **7 trend lines** in a grid
   - Save

4. **Capture the dashboard as source** — Studio → `AI Discovery Claude` app → Application Files → find the `pa_dashboards` record you just created → **Edit → Add to source control**. Commit & push. The dashboard is now versioned alongside the widgets.

## Registering in AI Control Tower as a tab

AI Control Tower (on Zurich+ instances with the `sn_ai_governance` family) surfaces dashboards via workspace tabs. The integration point depends on instance version:

### Option A — Workspace tab (Zurich+)

1. Studio → **UX Experience** (or navigate to `sys_ux_page_registry`) → find the AI Control Tower experience record.
2. Open the experience → **Related Lists → Tabs** (or `sys_ux_page_property` / `sys_ux_client_action`).
3. **+ New** → Label: `Claude Adoption`, Type: `Dashboard`, Content: reference the `pa_dashboards` sys_id of our Claude AI Adoption dashboard.
4. Save. The tab appears in Control Tower navigation for users with the appropriate role.

### Option B — Classic Dashboard listed under Control Tower menu

If Workspace UX isn't available, expose as a module under the AI Control Tower application menu instead:

1. Studio → find the `sys_app_application` for "AI Control Tower" (or whichever menu wraps it — often `sn_ai_disc` or `sn_grc_ai_gov`).
2. Add a new `sys_app_module` with:
   - `title = Claude AI Adoption`
   - `link_type = DASHBOARD`
   - `args = <dashboard sys_id>` (or set on `name` depending on link type)
3. Save, log out / in, module appears under Control Tower.

The exact path on hestanonprod needs the workspace schema checking — we can drop an `sys_ux_*` XML into the package once we see it.

## Widget naming convention

All widgets start with `Claude —` so they group together in any picker.
Use wildcard `Claude —*` when adding widgets to a dashboard and they'll
all surface at once.

## Unit customisation

Widgets inherit their unit from the indicator. Cost shows `USD`, tokens
show `tokens`. If you want different display precision, change the
`precision` field on the indicator (e.g. `2` for cents on cost).

## Control Tower integration notes

The OOB Control Tower "Adoption" tab reads **only** from `sys_gen_ai_usage_log`
(Now Assist internal). Our dashboard is a separate consumer — it reads our
three custom usage tables directly. Adding Anthropic usage to the OOB
dashboard would require writing our data into `sys_gen_ai_usage_log` too,
which has been flagged as a feedback item to ServiceNow but is out of
scope for this package.
