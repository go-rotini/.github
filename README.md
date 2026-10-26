# .github

The organization profile (`profile/README.md`) and the workflows every go-rotini repository
calls. Each repository keeps short caller files in its own `.github/workflows`, and the jobs live
here, so an action upgrade lands once instead of in every repository.

## Shared workflows

Callers use them at `@main`, from `go-rotini/.github/.github/workflows/<file>`.

| Workflow | What it does | A repository calls it from |
|---|---|---|
| `go-ci.yml` | Runs the jobs the caller lists, each on the OSes and Go releases it names, plus dependency review and a conventional-commit check on pull requests. Its `ok` job passes when everything else did. | `ci.yml`, on push, pull request and dispatch |
| `go-release.yml` | Releases from conventional commits with semantic-release, with an attested SBOM. | `release.yml`, when CI completes on main |
| `go-codeql.yml` | CodeQL analysis. | `codeql.yml` |
| `go-vuln.yml` | `make vuln` and `make vuln-tools`. | `vuln.yml`, weekly |
| `go-fuzz-nightly.yml` | Each fuzz target for ten minutes. | `fuzz-nightly.yml`, nightly |
| `go-tools-upgrade.yml` | Moves go.mod's toolchain line to its newest patch release and runs `make tools-upgrade`; commits to main when `make lint` passes, else opens an issue. | `tools-upgrade.yml`, weekly |
| `auto-merge.yml` | Merges a pull request once CI passes on it, when it is a grouped minor or patch Dependabot update or a go-rotini owner enabled auto-merge on it. | `auto-merge.yml`, when CI completes and when auto-merge is enabled |
| `dependabot-merge.yml` | The Dependabot half of `auto-merge.yml`, kept until every repository calls that instead. | |

Each file's header lists its inputs. Go comes from the caller's go.mod (its toolchain line)
unless `go-version` is given.

## Updates

Each repository's `dependabot.yml` groups minor and patch updates: `go-rotini` (sibling modules,
checked daily), `go-deps` (other modules, once a release is three days old) and `actions`
(weekly); rotini also has `docker`. Those pull requests merge themselves when CI passes. A major
version gets a pull request of its own and waits for review. Dependabot doesn't read tools.mod or
move the toolchain line; the weekly tools upgrade does both.

## Auto-merge for owners

A go-rotini organization owner can press **Enable auto-merge** on any pull request, theirs or
someone else's: rotinibot squash-merges it once CI passes on its latest commit. A press by
anyone else is ignored and the pull request waits for review. Each repository has
**Allow auto-merge** on so the button shows; GitHub's own auto-merge would wait for a
code-owner review nobody gives, so the button is only the signal.

## rotinibot

The merge and the tools upgrade act as rotinibot, a GitHub App that each repository's
`default` ruleset lets through, so the review rule still applies to people. Its pushes and merges
start CI and releases as a person's would.

- The App has repository permissions Contents, Pull requests and Workflows (read and write),
  organization permission Members (read, to tell who is an owner), and is installed on every
  repository.
- `ROTINIBOT_CLIENT_ID` is an organization variable holding its client ID.
- `ROTINIBOT_PRIVATE_KEY` is an organization secret holding its private key, as an Actions
  secret and as a Dependabot secret.
- It is a bypass actor on each repository's `default` ruleset.

Without the variable and secret, pull requests stay open for review and the tools
upgrade fails with a message saying what to set.
