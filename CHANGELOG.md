# Changelog

Each version is a tag of this repository, and a workflow pins the tag's full commit.

## 1.0.5

- Use Zod 4.6.7 and the matching Almanac dependency closure. The workflow,
  credential handling and grading policy are unchanged.

## 1.0.4

- The workflow locates its own exact code using GitHub's default identity-token
  audience. GitHub refuses the previous cross-owner GitHub audience.
- A refused or incomplete identity stops before checkout, with no service request.

## 1.0.3

- Claim jobs check the returned claim against the bounty and claimant you supplied,
  without requiring an artificial note or a cancellation API.
- Jobs that retain a consequential result require `workflow_dispatch`. They never
  receive a generated monthly schedule, and automatic CI events stop before a
  request reaches the service.

## 1.0.2

- The run says what happened, in its log and its summary: whether it did the
  job, what failed if it didn't, whether DNS counts it as the vendor's own, and
  where the report card is. A run that doesn't do the job fails the workflow,
  so GitHub tells you.
- A new refresh token is written back the moment your service issues it, so a
  later failure in the run can't lose it.
- A service that answers 429 or 5xx proves nothing either way, and a cleanup
  request that fails says what to remove by hand.
- An ID token your service issues is kept out of every later response too.
- Calls to Sourcey are retried when the network or Sourcey fails for a moment.
- The README and the workflow's input descriptions say what to do in plain
  steps. Earlier releases are refused: pin 1.0.2.

## 1.0.1

- The workflow the verify page gives runs once a month, not daily, on a day
  and at a time of the job's own. A workflow you saved earlier keeps its own
  schedule: copy the page's `cron` line into it. The card shows the job
  verified with the date of the last run that did it.

## 1.0.0

- First release: performs one Agent Readiness job binding on its account
  holder's runner, attests the record with the job's GitHub identity and
  submits it for Sourcey to replay.
