# sys-github-worklines

Keeps the Status of open pull requests on the org Project
[Worklines](https://github.com/orgs/enterprise-r/projects/3) in step with
their draft state and new commits. Part of the GitHub system,
[`sys-github`](https://github.com/enterprise-r/sys-github). Public for free
Actions minutes on GitHub Free. An informal mount at
`/r/sys/github/worklines`, not listed in `mani.yaml`.

## Sync

`.github/workflows/sync.yml` runs hourly and on demand, until the schedule
fires reliably. The schedule is best-effort: GitHub may skip or delay runs.
Each run makes one
GraphQL query for the open pull requests on the project, then one update per
item that moves:

| Pull request | Status | Moves to |
|---|---|---|
| draft | any but Approved, Done | In Progress |
| not draft | Todo, In Progress, none | Ready for review |
| not draft | Needs Work, last commit newer than the latest changes requested | Ready for review |
| any | Approved, Done | stays |

Options are looked up by name each run. A missing option fails the run. Each
move logs `<repo>#<n> <from> -> <to>`.

## Keepalive

GitHub disables a scheduled workflow in a public repository after 60 days
without activity. Once a day, at 04:03 UTC, and on demand, the `keepalive` job
re-enables `sync.yml` through the API, which resets that timer without a
commit. It uses the run's own token with only `actions: write`. A workflow
already disabled cannot revive itself: enable it in the Actions tab.

The only secret is `WORKLINES_TOKEN`: `enterprise-r-bots`, org projects
read and write, and metadata, pull requests and contents read on all org
repositories. Stored by the Founder.
