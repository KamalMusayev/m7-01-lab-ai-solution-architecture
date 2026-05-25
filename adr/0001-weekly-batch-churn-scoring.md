# ADR 0001: Use Weekly Batch Scoring for Churn Predictions

## Context

The sales team needs a ranked list of accounts at risk of churn every Monday morning. The scenario says that the underlying churn score only needs to refresh weekly, so the system does not need to respond to each user action or event in real time.

## Decision

We will use weekly batch scoring for churn prediction. The system will collect account, product usage, billing, and CRM data into the warehouse, build weekly account-level features, and run a scheduled batch scoring job before the Monday sales workflow starts. The output will be written to a churn scores table and consumed by the CRM dashboard and outreach playbooks.

## Alternatives rejected

- **Online inference API:** Rejected because no user or system is waiting for a millisecond-level churn prediction. Keeping an always-on API would increase cost and operational complexity without matching the weekly business workflow.
- **Streaming inference:** Rejected because churn scoring does not need to react instantly to every product event or billing update. The sales team uses a weekly ranked list, not a live event-by-event decision stream.
- **Daily batch scoring:** Rejected because the scenario only requires weekly refresh. Daily scoring would increase compute cost and monitoring load without a clear business consumer for the extra freshness.

## Consequences

- The system is cheaper and simpler to operate because compute is only needed for scheduled weekly scoring instead of running a prediction service all the time.
- The workflow matches how the sales team consumes the result: one ranked list delivered every Monday and used throughout the week.
- The main downside is staleness. If an account's behavior changes sharply during the week, the score will not update until the next scheduled batch run.

## Revisit if

- Revisit this decision if the business starts needing churn scores inside real-time product experiences, daily sales workflows, or automated retention actions that must react immediately to customer behavior.
