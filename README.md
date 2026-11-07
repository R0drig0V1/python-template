# python-template

A [Copier](https://copier.readthedocs.io/) template for new Python projects: uv, ruff, mypy
strict, pytest with coverage, and GitHub workflows with code quality and security checks
pinned by SHA. It is language-specific on purpose. It is separate from the agentic
`multi-agent-herdr` plugin ([claude-code-herdr](https://github.com/R0drig0V1/claude-code-herdr)),
which can be installed on top of a generated project, since its `scripts/check.sh` is already
filled in.

## Create a project

Requires [uv](https://docs.astral.sh/uv/) and access to this (private) repository through git
(`gh auth setup-git`, or SSH).

```bash
uvx copier copy gh:R0drig0V1/python-template my-project
```

Copier asks for the project name, package name, description, author, GitHub owner, Python
version (3.12, 3.13 by default, 3.14), visibility (private by default) and license (MIT or
proprietary). Then it prints the next steps: `git init`, `uv sync`, `scripts/check.sh`, first
commit.

To bring later template improvements (new checks, newer pinned actions) into an existing
project, run this in a clean working tree:

```bash
uvx copier update
```

## What a generated project gets

| Area | Contents |
|---|---|
| Python | `pyproject.toml` (src layout, `uv_build`), `.python-version`, `src/<package>/` with `py.typed`, `tests/` |
| Quality | ruff `E,F,W,I,B,UP,S,SIM,RUF,C90` (mccabe <= 10), ruff format, mypy strict, pytest + branch coverage >= 80% |
| One command | `scripts/check.sh`: everything CI checks; CI runs exactly this script |
| CI | `ci.yml` (check.sh), `quality.yml` (zizmor, actionlint, gitleaks over the whole history) |
| Public only | `codeql.yml` (Python, Actions), `scorecard.yml` (OpenSSF), MIT `LICENSE`, SECURITY.md with private vulnerability reporting |
| Maintenance | `dependabot.yml` (uv weekly, actions monthly, grouped, 7-day cooldown), pre-commit with ruff, PR/issue templates |
| Settings | README lists the `gh api` commands for the repository settings (Dependabot alerts, SHA pinning...) |

Every workflow has least-privilege `permissions:` and `persist-credentials: false`, and pins
its actions by commit SHA. Downloaded binaries are checked against a sha256.

CodeQL, Scorecard, secret scanning and push protection are free only on public repositories;
on private ones gitleaks covers committed secrets.

## Maintenance

- `test-template.yml` generates a private and a public project on every push and PR, plus
  Python 3.12 and 3.14 variants. It runs `uv sync`, `scripts/check.sh`, actionlint and zizmor
  on each, and lints this repository's own workflows.
- Dependabot updates the pinned actions in `.github/workflows/` and
  `template/.github/workflows/`.
- By hand:
  - the lower bounds of the dev tools in `template/pyproject.toml.jinja` (each project's
    `uv.lock` and Dependabot update the exact versions);
  - `uv_build`;
  - the actionlint and gitleaks versions and sha256 in the quality workflows;
  - the `version:` inputs of `setup-uv` and `zizmor-action`;
  - `copier==` in `test-template.yml`.

  Prefer releases at least 7 days old.

Test locally (from this directory):

```bash
uvx copier copy --defaults --vcs-ref HEAD --data project_name=Demo --data author_name=Me . /tmp/demo
cd /tmp/demo && git init -q -b main && uv sync && scripts/check.sh
```
