# Sourcey Agent Readiness Action

Verify that an agent can do your service's job, with your own account, on your
own GitHub Actions runner. Sourcey never holds your credentials.

The Action runs one job from your service's Agent Readiness listing: it asks
Sourcey for a one-time challenge, makes the job's requests with your
credentials, records each one, signs the record with your workflow's GitHub
identity and sends it to Sourcey. Sourcey replays the record, checks the job's
results itself, and your report card shows the job verified.

## Set it up

1. Open your report card on sourcey.com and choose **Verify the job**. The page
   shows the workflow for your job, filled in: the profile, the binding, the
   release commit to pin, and whether it can run monthly.
2. Save it under the workflow filename the page shows on your repository's
   default branch. Each report card and binding has its own filename.
3. In **Settings → Secrets and variables → Actions**, add the repository
   secret under the name the page shows (see below).
4. Run it now from **Actions → Agent Readiness → Run workflow**. After that it
   runs once a month on its own if the job can be repeated safely.

A job that keeps a consequential result, such as claiming a bounty, runs only
when you start it with **Run workflow**. Supply the bounty and claimant you
intend to use. The Action refuses scheduled runs and other automatic events
before making any request to the service. It does not invent a cancellation
API or treat an operator's later cleanup as part of the worker's capabilities.
`workflow_dispatch` can also be invoked through GitHub's API; it is not proof
that a person clicked a button. Do not automate dispatches for these jobs.

The card shows the job verified with the date of the last run that did it.
When a run doesn't do the job, the workflow fails, so GitHub tells you, and the
run's summary says what failed. GitHub stops the scheduled runs of a public
repository that has gone 60 days without a commit, so use one that sees
commits, such as your SDK's.

The workflow looks like this:

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

Pin the full commit the Verify the job page shows. Sourcey accepts runs only
from commits its release chain tagged, and only on GitHub-hosted runners.

## The credentials secret

The secret is one JSON object: each credential the binding uses, and any input
the job asks for, such as the number a text message goes to. The Verify the job
page lists exactly what yours needs:

```json
{ "account_key": "..." }
```

The Action refuses a secret that misses one of these or adds another. Credential
values must be at least 8 characters so custody can keep them out of a service's
response. Supplied inputs, such as a bounty number, need only be non-empty.
Every value is masked in the job's log.

### A refresh token that changes on each use

If your service replaces its refresh token each time it's used, keep the
refresh token in a secret of its own, `SOURCEY_REFRESH_TOKEN`, and leave
`refresh_token` out of the credentials JSON. Create a fine-grained GitHub token
with **Secrets: write** on this repository only, save it as
`SOURCEY_SECRETS_WRITE_TOKEN`, and add these lines to the workflow:

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

The Action writes each new refresh token back to `SOURCEY_REFRESH_TOKEN` as
soon as your service issues it, sealed to the repository's key, and does
nothing else with the GitHub token.

## What leaves your runner

- Your credentials never do. They're held in memory for the run and sent only
  to the binding's own endpoints, and the run's record refers to each by a
  fingerprint.
- If a response echoes a credential back, however it's encoded, that response
  isn't kept: the run stops there and proves nothing either way. So does a
  service that answers 429 or 5xx.
- Sourcey receives a record of each request (method, address, header names,
  status, sizes and fingerprints), the response bodies with credentials
  removed, and a signed attestation of the record. It publishes the record and
  keeps the bodies and the attestation private.

## Say the runs are yours

A run from any repository can verify the job. Your own runs count for more:
two of them, a while apart, that fail the same way put that failure on the card
in place of Job verified until one of your runs passes. To make a repository's
runs yours, add a DNS TXT record at `_sourcey-agent-readiness.<your domain>`
reading `github-owner-id=<owner id> github-repository-id=<repository id>`.

The attestation is signed on the Sigstore instance GitHub assigns your
repository. For a public repository that's the public-good instance, whose
public log names your repository and the run; for a private or internal one
it's GitHub's own, with no public log. Sourcey's published record never names
your repository.

## Releases

Every file here is projected by Sourcey's release chain from the exact Catalog
source Sourcey replays runs with. `dist/run.mjs` is one bundle whose digest
`SOURCEY-BUNDLE.json` records, and `THIRD_PARTY_LICENSES.txt` carries the
license of everything bundled in it.
