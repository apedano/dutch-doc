# Docs

Documentation site built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/), automatically built and deployed to GitHub Pages via GitHub Actions on every push to `main`.

## Preliminary steps

### Activate virtual environment
If not done yet, activate the virtual environment

```shell
py -m venv .venv
.\.venv\Scripts\activate
```

we should see the prefix on the shell `(.venv) PS .>`

## Install mkdocs-material

```bash
pip install mkdocs-material
```

## Local development

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000

### Reuirement.txt

Additional dependencies if needed 



## Project structure

```
.
├── docs/                  # Markdown source files
│   └── javascripts/
│       └── mathjax.js     # LaTeX rendering config
├── mkdocs.yml              # Site configuration
├── requirements.txt        # Python dependencies
└── .github/
    └── workflows/
        └── deploy.yml      # CI/CD workflow
```

## Deployment

Every push to `main` triggers a GitHub Actions workflow that:

1. Installs MkDocs Material and dependencies
2. Builds the static site (`mkdocs build`)
3. Publishes the result to the `gh-pages` branch
4. GitHub Pages serves the site from `gh-pages`

No manual `mkdocs gh-deploy` step is needed once this is set up.

### One-time setup

1. Add `requirements.txt` to the repo root:

    ```txt
    mkdocs-material
    ```

2. Add `.github/workflows/deploy.yml`:

    ```yaml
    name: Deploy Docs

    on:
      push:
        branches:
          - main
      workflow_dispatch:

    permissions:
      contents: write

    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout repository
            uses: actions/checkout@v4
            with:
              fetch-depth: 0

          - name: Set up Python
            uses: actions/setup-python@v5
            with:
              python-version: "3.x"

          - name: Cache pip dependencies
            uses: actions/cache@v4
            with:
              key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
              path: ~/.cache/pip
              restore-keys: |
                ${{ runner.os }}-pip-

          - name: Install dependencies
            run: pip install -r requirements.txt

          - name: Build and deploy
            run: mkdocs gh-deploy --force --clean
    ```

3. In your repository settings, go to **Settings → Pages** and set the source to branch `gh-pages`, folder `/ (root)`. (After the first workflow run, GitHub often detects this automatically.)

4. Commit `docs/`, `mkdocs.yml`, `requirements.txt`, and the workflow file, then push to `main`. The Actions tab will show the build running, and the site will be live at:

    ```
    https://<username>.github.io/<repo-name>/
    ```

## Notes

- `mkdocs gh-deploy --force --clean` overwrites the `gh-pages` branch on every run — this is expected, since it's a generated artifact, not source.
- The `site/` directory (local build output) is gitignored and never committed to `main`.
- `permissions: contents: write` is required so the workflow can push to the `gh-pages` branch.