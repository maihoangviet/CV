# CV — Mai Hoang Viet

Live at **<http://cv.vietmh.asia>**

LaTeX source for my CV, plus a GitHub Actions workflow that compiles it and publishes the PDF to GitHub Pages. Pushing a change to `src/cv.tex` on `main` is the whole release process.

## Repository layout

| Path | Purpose |
|---|---|
| `src/cv.tex` | The CV. Single source file — no includes |
| `.github/workflows/build.yml` | Compile and publish pipeline |
| `index.html` | Landing page that redirects to the PDF |
| `CNAME` | Custom domain for GitHub Pages |
| `CLAUDE.md` | Working notes, loaded automatically by Claude Code: layout constraints, decisions already made, ideas not used yet |
| `Makefile` | Local build shortcut — run it from inside `src/`, not the repo root |

Generated PDFs are gitignored. The published copy lives on the `build` branch, never on `main`.

## Building locally

Requires a TeX distribution with `latexmk` and `pdflatex` (TeX Live or MacTeX). The document pulls in `fontawesome5`, `CormorantGaramond`, `FiraSans`, `roboto`, `sourcesanspro`, `noto-sans`, `titlesec`, `enumitem`, and `tabularx` — a full TeX Live install covers all of them.

```bash
cd src
latexmk -pdf cv.tex          # build
latexmk -c cv.tex            # clean aux files
latexmk -C cv.tex            # clean aux files and the PDF
```

Add `-g` to force a rebuild when `latexmk` insists nothing changed.

## How publishing works

`build.yml` has a **single job**, `publish`, that runs on `ubuntu-latest`.

**Triggers**

- Push to `main` touching `src/**/*.tex`, `.github/workflows/build.yml`, `CNAME`, or `index.html`
- `workflow_dispatch` — run it by hand from the Actions tab

Concurrent runs on the same ref cancel each other, and the job times out after 15 minutes.

**Steps**

1. Check out the repo.
2. `dorny/paths-filter` decides whether any `.tex` file actually changed.
3. If a `.tex` changed *or* the run was dispatched manually, compile `src/cv.tex` with `dante-ev/latex-action`.
4. Assemble `github_deploy/` from `index.html` and `CNAME`.
5. Add the PDF, one of two ways:
   - **Compiled this run** → copy the fresh PDF in.
   - **No `.tex` change** → fetch the last published PDF from the `build` branch. This is why editing only `index.html` or `CNAME` does not force a LaTeX rebuild.
6. Assert `index.html` exists and the PDF is non-empty. The job fails here rather than publishing a broken page.
7. Push `github_deploy/` to the `build` branch with `clean: true`, replacing its contents.

GitHub Pages serves the `build` branch at the domain in `CNAME`. That publish step is GitHub's own `pages-build-deployment` workflow, which appears as a second run in the Actions tab — it is not defined in this repo.

## Configuration

**Renaming the published PDF** — change `PDF_NAME` in `build.yml`, then update the redirect target in `index.html` to match. If the two disagree the landing page 404s.

**Custom domain** — `CNAME` holds the domain, and it must also be set under Settings → Pages. The file is copied into every deploy so the domain survives the `clean: true` push.

**Required setup** (already done here, needed again only on a fresh fork):

- Settings → Actions → General → Workflow permissions → **Read and write**
- Settings → Pages → Source → **Deploy from a branch** → `build` → `/(root)`

## Troubleshooting

**The workflow did not run.** Check the push was on `main` and touched a watched path. If it looks correct and the run is stuck in `queued`, check <https://www.githubstatus.com> — an Actions incident will hold runs indefinitely. Re-run with `gh workflow run build.yml --ref main`.

**The PDF did not update but the run succeeded.** The `.tex` filter did not match, so step 5 reused the previous PDF. Trigger `workflow_dispatch`, which forces a compile regardless of the filter.

**Pages returns 404.** Verify the Pages source is the `build` branch at `/(root)`, and that `PDF_NAME` matches the redirect in `index.html`.

**LaTeX fails in CI but builds locally.** The CI image is pinned to `dante-ev/latex-action@2024-B` and may ship different package versions than your machine. Read the `Build LaTeX document` step log for the first `!` line — that is the real error.
