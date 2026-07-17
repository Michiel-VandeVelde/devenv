# @jitsedesmet/devenv

A [Copier](https://copier.readthedocs.io/) template that publishes a personal
`.devcontainer` setup and keeps it in sync across repositories.

The exact `.devcontainer` used for my projects lives in `template/`, so a single
command drops it into any project and `copier update` later reconciles your local
tweaks with new template versions using a git-style 3-way merge.

## Requirements

- [Copier](https://copier.readthedocs.io/en/latest/#installation) 9 or newer
  (`pipx install copier` / `uv tool install copier` / `brew install copier`)
- Git 2.27 or newer

## Usage

### Add the devcontainer to a repository

```sh
# Run from the root of the target repository
copier copy gh:jitsedesmet/devenv .
```

You are asked for a `name` (the devcontainer name shown in your editor / IDE); it
defaults to the target directory name. Copier writes:

- `.devcontainer/devcontainer.json` — with `name` filled in
- `.devcontainer/Dockerfile`
- `.copier-answers.yml` — records the template version and your answers so future
  updates know where you started. **Do not edit it by hand.**

Copier refuses to overwrite existing files unless you pass `--force`, so it is safe
to run in a populated repository.

Use `--defaults` to accept every default without prompting (handy in CI / scripts).

### Refresh the template later

```sh
# Run from the root of a repository that was created with this template
copier update
```

Copier checks out the newest release tag of the template, replays your recorded
answers, and merges the new template into your project:

- Files you never touched are refreshed to the new template.
- Files you changed are merged with the upstream changes. Non-overlapping edits are
  combined automatically; overlapping edits are surfaced as conflicts.

Conflicts are written inline with the usual `<<<<<<<` / `=======` / `>>>>>>>`
markers (`--conflict inline`, the default). Pass `--conflict rej` to get `.rej`
files instead. Review and resolve conflicts before committing — a
[pre-commit](https://pre-commit.com/) `check-merge-conflict` hook is recommended.

## How it works

- `copier.yml` declares the questions (currently just `name`) and points Copier at
  the `template/` subdirectory via `_subdirectory`.
- Everything under `template/` is rendered into the target project. Only files
  ending in `.jinja` are processed as templates (the suffix is stripped);
  `Dockerfile` is copied verbatim.
- `template/.devcontainer/devcontainer.json.jinja` injects your `name`; the
  `${localWorkspaceFolder}` mount variables are left untouched because Copier uses
  `{{ ... }}` delimiters.
- `template/{{_copier_conf.answers_file}}.jinja` renders the `.copier-answers.yml`
  file that powers `copier update`.

## Development

The template content lives in `template/`. To try changes locally:

```sh
# Render into a scratch directory
copier copy --defaults --vcs-ref=HEAD . /tmp/devenv-test
```

`--vcs-ref=HEAD` renders the current commit instead of the latest tag, which is
useful while iterating.

## Releasing

Copier selects the newest git tag that is a valid
[PEP 440](https://peps.python.org/pep-0440/) version. To publish a new template
version, commit your changes and push an annotated tag:

```sh
git tag -a 1.2.0 -m "1.2.0"
git push --follow-tags
```

Consumers pick it up the next time they run `copier update`.

Dependencies referenced by the template (e.g. the Docker base image) are kept
current automatically with Renovate.
