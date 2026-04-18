# CV (LaTeX + GitHub Pages)

This repository contains a personal CV in LaTeX and a GitHub Actions pipeline that builds and publishes the latest PDF automatically.

## What this setup does

- Builds `src/cv.tex` on each eligible push to `main`
- Renames the generated PDF to a custom public filename
- Publishes the PDF to the `build` branch
- Copies static web files (`index.html`, `CNAME`) to `build`
- Serves the CV through GitHub Pages with a redirect page

## Repository structure

- `src/cv.tex`: main CV source file
- `.github/workflows/build.yml`: CI build and publish pipeline
- `index.html`: redirect page to the generated PDF
- `CNAME`: custom domain for GitHub Pages (optional)
- `Makefile`: legacy local build commands (currently expects `cv.tex` at repo root)

## Prerequisites

- A GitHub account (for fork + Actions + Pages)
- LaTeX installed locally if you want to build on your machine
- Recommended local tools: `latexmk`, `pdflatex`

## Quickstart (GitHub)

1. Fork this repository.
2. In your fork, enable workflow write permission:
   `Settings -> Actions -> General -> Workflow permissions -> Read and write permissions`.
3. Edit `src/cv.tex`.
4. Push to `main`.
5. Configure GitHub Pages source:
   `Deploy from a branch -> build -> /(root)`.

After the workflow succeeds, your CV is available at:
`https://<username>.github.io/<repo>/`

If you use a custom domain, update `CNAME` and your DNS records.

## Local build

Build from repository root:

```bash
latexmk -pdf src/cv.tex
```

Clean intermediate files:

```bash
latexmk -c src/cv.tex
```

Full clean (including generated PDF):

```bash
latexmk -C src/cv.tex
```

Alternative (build inside `src/`):

```bash
cd src
latexmk -pdf cv.tex
```

## PDF filename and redirect

Public PDF name is controlled by `PDF_NAME` in `.github/workflows/build.yml`.

If you change it, also update the redirect target in `index.html`:

```html
window.location.href = "./<your-pdf-name>.pdf"
```

This keeps your landing URL stable while serving the latest generated PDF.

## Deployment workflow summary

`build.yml` runs 3 jobs:

1. `build`:
   compiles LaTeX in `src/`, renames PDF, uploads artifact.
2. `deploy`:
   publishes artifact contents to `build` branch.
3. `sync-assets-to-build`:
   copies `index.html` and `CNAME` from `main` to `build`.

Workflow triggers on pushes to `main` when these files change:

- `src/**.tex`
- `.github/workflows/build.yml`
- `index.html`
- `CNAME`

## Troubleshooting

- Workflow did not run:
  check if the push is on `main` and includes a watched path.
- Pages shows 404:
  verify Pages source is `build` branch and `/(root)`.
- Wrong file opens or redirect breaks:
  verify `PDF_NAME` in workflow matches `index.html` redirect target.
- Build branch not updating:
  check Actions logs for failed `build` or `deploy` jobs.
