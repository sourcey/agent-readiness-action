# Sourcey Agent Readiness Action

Verify that an agent can do your service's job, with your own account, on your
own GitHub Actions runner. Sourcey never holds your credentials.

The Action runs one job binding from your service's Agent Readiness listing:
it asks Sourcey for a challenge, performs the binding's calls with your
credentials, records every exchange, signs the record with your job's GitHub
identity and submits it. Sourcey replays the record against the binding, checks
the job's assertions itself, and the report card shows the job verified.

## Add the workflow

Add one workflow to a repository you control, naming the profile and binding
your report card shows, at a released commit of this repository. The card shows
the job verified with the date of the last run that did it. The workflow runs
once a month by default, and you can run it from the Actions tab whenever you
like. Take the workflow from your report card's verify page: it gives your job
a day and a time of its own, so runs spread across the month.

```yaml
name: Agent Readiness

on:
  schedule:
    - cron: "<minute> <hour> <day> * *"
  workflow_dispatch:

jobs:
  verify:
    permissions:
      id-token: write
    uses: sourcey/agent-readiness-action/.github/workflows/run.yml@<commit>
    with:
      profile: arp_...
      binding: <binding id>
    secrets:
      credentials: ${{ secrets.SOURCEY_AGENT_READINESS_CREDENTIALS }}
```

Pin the full commit of a release tag. Sourcey accepts runs only from commits
its release chain tagged, and only from GitHub-hosted runners.

## The credentials secret

One repository secret holds a JSON object from the binding's credential roles
to their values, and any input the job asks its runner for, such as the number
a text message goes to:

```json
{ "account_key": "..." }
```

An OAuth binding names `client_id`, `client_secret` or `refresh_token` as its
form needs. The Action refuses a secret that lacks a role, or names one the
binding never uses, and any credential shorter than 8 characters: one that short
cannot be told from ordinary text, so nothing could keep it out of what a
service returns. Every value is masked in the job's log.

### A rotating refresh token

A service that issues a new refresh token on each use leaves the stored one
unusable. Pass the refresh token in a secret of its own, and give the Action a
fine-grained token with Actions secrets write on this repository alone; a
rotation is then sealed to the repository's public key inside the run's custody
and written back to that secret, and nothing else is done with the token:

```yaml
    with:
      profile: arp_...
      binding: <binding id>
      refresh_token_secret: SOURCEY_REFRESH_TOKEN
    secrets:
      credentials: ${{ secrets.SOURCEY_AGENT_READINESS_CREDENTIALS }}
      refresh_token: ${{ secrets.SOURCEY_REFRESH_TOKEN }}
      secrets_token: ${{ secrets.SOURCEY_SECRETS_WRITE_TOKEN }}
```

## What leaves the runner

- Your credentials never do. They live in an in-memory custody for the run,
  attached only to the binding's own endpoints and methods, and the run's
  record names each by a digest of its handle.
- A response that echoes any credential the run holds, however it is escaped,
  is never kept: its body is withheld whole and a header carrying one is
  dropped. That call's answer cannot be read, so the run stops there as
  residue, which neither proves the job nor counts against it.
- Sourcey receives the record of each exchange (method, URL, header names,
  status, sizes and digests), the scrubbed response bodies, and a Sigstore
  attestation of the record. It publishes the run record; it keeps the bodies
  and the attestation privately.

## The transparency log

The attestation is signed on the Sigstore instance GitHub assigns your
repository. For a public repository that is the public-good instance, whose
public log names your repository and the run. For a private or internal
repository it is GitHub's own instance, with no public log. Sourcey's published
record never names your repository: it says whether the run was the vendor's
own, which you state with a DNS TXT record at
`_sourcey-agent-readiness.<your domain>` reading
`github-owner-id=<owner id> github-repository-id=<repository id>`.

## Releases

Every file here is projected by Sourcey's release chain from the exact Catalog
source that Sourcey replays runs with; `dist/run.mjs` is one bundle whose digest
`SOURCEY-BUNDLE.json` records, and `THIRD_PARTY_LICENSES.txt` carries the
license of everything bundled in it.
