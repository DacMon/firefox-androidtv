# One-click pipeline: AI Studio → GitHub → Netlify

This repo is set up so that **a single "Save to GitHub" click in Google AI
Studio** (or any commit to `main`, however you make it) automatically:

1. Builds the custom Firefox for Android TV APKs on GitHub Actions
   (`.github/workflows/build-firefox.yml`), and
2. Publishes a download page with the fresh APKs to your Netlify site.

```
AI Studio ──(Save to GitHub)──► GitHub main branch
                                     │  push triggers Actions
                                     ▼
                          Build APKs (fenix/, Gradle)
                                     │  on success
                                     ▼
                    Netlify site: download page + APK files
```

There is no button to add inside AI Studio itself — AI Studio can only sync
to GitHub. The trick to making this reliable is letting GitHub Actions do
everything downstream of that one click. Netlify's own "connect a Git repo"
feature does **not** work here because Netlify cannot build Android apps;
the workflow pushes the finished files to Netlify instead.

## One-time setup

### 1. Connect AI Studio to this repo

In AI Studio, open your app → **Save to GitHub** → choose this repository
and the `main` branch. After that, every save is one click.

### 2. Create the Netlify site

1. In Netlify: **Add new site → Deploy manually**, drag in any placeholder
   file (it gets replaced by the first real deploy).
2. Copy the **Site ID** from *Site configuration → General*.
3. Create a Personal Access Token: Netlify → *User settings → Applications →
   New access token*.

### 3. Add the two secrets to GitHub

In this repo: **Settings → Secrets and variables → Actions → New repository
secret**:

| Secret name          | Value                        |
|----------------------|------------------------------|
| `NETLIFY_AUTH_TOKEN` | your Netlify personal token  |
| `NETLIFY_SITE_ID`    | the Site ID                  |

Until these secrets exist, the workflow still builds and uploads the APKs
as an Actions artifact — it just skips the Netlify deploy steps.

## Using it day to day

- Edit in AI Studio → **Save to GitHub**. That's it. A few minutes later the
  Netlify page has the new APKs.
- To rebuild without an edit: repo → **Actions → Build Custom Firefox for
  TV → Run workflow**.
- On the TV, open the **Downloader** app and enter your Netlify site URL to
  install the APK.

## What was wrong before (why builds kept failing)

- The workflow ran Gradle from the repo **root**, but this monorepo keeps the
  Firefox app in `fenix/` — `gradlew` doesn't exist at the root, so every run
  died on its first real step. It now runs with `working-directory: fenix`.
- The artifact path was `app/build/...`; the real output (with the `fenix`
  product flavor and per-ABI splits) is
  `fenix/app/build/outputs/apk/fenix/debug/*.apk`.
- `fenix/app/src/main/res/drawable/ic_cursor_pointer.xml` was an **empty
  file** (created via the GitHub web UI), which makes the Android resource
  compiler fail the whole build. It now contains a real cursor icon.
