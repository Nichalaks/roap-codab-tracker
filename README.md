# ROAP COD-AB Tracker

A one-page dashboard showing the COD-AB (administrative boundary) review status for the countries covered by OCHA ROAP. Countries are sorted by how urgently each dataset needs its review. You can filter and sort every column.

**Data source:** the [COD-AB Status Dashboard](https://ocha-dap.github.io/hdx-cod-ab-status/) run by OCHA Field Information Services (FIS).

## How it works

```
FIS dashboard CSVs ──(GitHub Action, daily 06:30 UTC)──► data/*.csv in this repo ──► GitHub Pages site
        ▲                                                                                │
        └──────────────────── "Refresh from FIS" button reads the FIS files directly ◄───┘
```

- **Daily copy.** `.github/workflows/update-and-deploy.yml` downloads the 8 CSV files from FIS every day at 06:30 UTC (13:30 Bangkok), 30 minutes after FIS updates them.
  - Before saving, it checks that each file has data and that the columns the page reads still exist.
  - If anything changed, it commits the files to `data/`. Then it publishes the site.
  - Git history keeps every daily version, so you can see when a country's status changed.
- **Page.** `index.html` reads `data/*.csv` and works out the review status in the browser.
- **Refresh button.** "Refresh from FIS" reads the FIS files directly, for changes made since the last daily copy. FIS allows other websites to read these files.
- **If the copy is missing**, for example before the first run, the page reads FIS directly instead.

**How "next review" is calculated:** the later of `date_reviewed` and `date_updated`, plus `update_frequency` in years (1 year for most datasets). This is the 12-month review cycle from the IM Toolbox. It matches the "Next review" column on the FIS dashboard.

## Files

| Path | What it is |
|---|---|
| `index.html` | The dashboard (HTML, CSS and JavaScript in one file) |
| `config.json` | Regional office code, the FIS data address, the Regional Focus Model ranks and short display names |
| `data/` | Daily copy of the FIS CSV files, written by the Action. Don't edit by hand. |
| `.github/workflows/update-and-deploy.yml` | Daily sync and GitHub Pages deployment |

## Publish it on GitHub Pages: step by step

You need permission to create a repository in the team's GitHub organization (or your own account) and to change its settings.

1. **Create the repository**
   - On GitHub, click **New repository**.
   - Pick the owner (the team's organization) and a name, e.g. `roap-codab-tracker`.
   - Choose **Public**. GitHub Pages on a private repository needs GitHub Enterprise. The data is already public on HDX and the FIS site.
   - Click **Create repository**.

2. **Upload the files**
   - Click **uploading an existing file**.
   - Drag in everything from this folder, keeping the structure: `index.html`, `config.json`, `README.md`, `.gitignore`, the `data` folder and the `.github` folder.
   - The `.github` folder may be hidden on your computer:
     - **Windows File Explorer:** View → Show → Hidden items.
     - **macOS Finder:** press Cmd + Shift + .
   - Check that `.github/workflows/update-and-deploy.yml` appears in the upload list. Then click **Commit changes**.
   - *Command-line alternative:* `git init`, `git add .`, `git commit -m "Initial tracker"`, `git branch -M main`, `git remote add origin <repo-url>`, `git push -u origin main`.

3. **Turn on GitHub Pages**
   - Go to **Settings → Pages**.
   - Under **Build and deployment → Source**, choose **GitHub Actions**. Not "Deploy from a branch".

4. **Let the Action commit data**
   - Go to **Settings → Actions → General → Workflow permissions**.
   - Select **Read and write permissions**, then **Save**.
   - If these options are greyed out, an organization owner has to allow them.

5. **Run it the first time**
   - Go to **Actions → Update COD-AB data and deploy → Run workflow** (branch `main`).
   - It takes about 1–2 minutes. Both jobs, *Copy CSV files* and *Publish to GitHub Pages*, should show green ticks.

6. **Open the site**
   - The address is shown under **Settings → Pages**, and in the *deploy* job summary.
   - It looks like `https://<organization>.github.io/roap-codab-tracker/`.
   - Share this link with the team.

After that, it runs every day by itself. Every push to `main` also republishes the site.

## Routine maintenance

- **New Regional Focus Model each year:**
  - Edit `config.json`: change `rfm.year`, `rfm.source` and the `ranks` list (ISO3 → rank).
  - Commit the change. The site republishes automatically.
- **Countries move between regional offices:** nothing to do. The ROAP list comes from the FIS `regions.csv` file every day.
- **The Action fails with "no longer has column …":**
  - FIS changed its file format. The Action stops, so the last good copy stays online.
  - Update the column names in `index.html` (function `buildCountries`) and in the `check` lines of the workflow.
- **The page says "the last daily sync could not read the FIS files":**
  - FIS was unreachable at 06:30 UTC.
  - Re-run the workflow from the Actions tab, or use "Refresh from FIS" on the page.
- **GitHub pauses scheduled workflows** after 60 days with no activity in a public repository. If the daily runs stop, open the Actions tab and click **Enable workflow**. Any commit also resets the timer.

## Run it locally

The page loads files with `fetch`, so it doesn't work when opened as a file (`file://`). Serve the folder instead:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

---
Maintained by OCHA ROAP Information Management (GIS). Data © OCHA FIS / national data providers, as listed in each dataset's metadata.
