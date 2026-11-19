# ci-artifact-env-poc

Minimal, self-contained reproduction of a GitHub Actions **artifact-poisoning → `$GITHUB_ENV`
injection** primitive, built in a repository owned by the researcher.

Shape under test (two workflows, exactly as they appear in the wild):

1. `Validate` runs `on: pull_request`. For a fork pull request GitHub runs the workflow file
   **from the pull request's own merge ref**, so the fork author controls this file and every
   byte of the artifact it uploads. The job is unprivileged: read-only `GITHUB_TOKEN`, no
   secrets, PR-scoped Actions cache.
2. `Privileged consumer` runs `on: workflow_run` after `Validate` completes. That trigger runs
   the **base branch's** workflow in the **base repository's** context — secrets available,
   token not forced read-only. It downloads the artifact the fork produced, `unzip`s it into the
   workspace, and appends every line of the extracted `env` file to `$GITHUB_ENV`.

Result: the fork chooses environment variables for a job running as the base repository, and
chooses files in that job's workspace. The next Node-based action starts under them.

Nothing here touches a secret. The payload is a `console.log`.
