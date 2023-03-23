# Delegation Strategy Pipeline
This repository takes care of running Marinade's process scoring Solana validators.

## Scoring results
The directory `scoring` contains results of scoring runs along with reports.

## Pipelines

Several pipelines are defined here, see their flow in the graph attached below:

```mermaid
flowchart TB
    subgraph scheduler [ ]
        direction TB
        scheduler_a[Scheduler pipeline] -->|fa:fa-clock every 15 minutes| scheduler_b
        scheduler_b[Fetch epoch & repository status] --> scheduler_c
        scheduler_c{Is this epoch scored?}
        scheduler_c -->|No| scheduler_d[Trigger scoring pipeline]
        scheduler_c -->|Yes| scheduler_e[noop]
    end
    subgraph scoring [ ]
        direction TB
        scoring_a[Scoring pipeline] -->scoring_b
        scoring_b[Fetch input data] -->scoring_c
        scoring_c[Generate scores] -->scoring_d
        scoring_d[Generate report & summary] -->scoring_e
        scoring_e[Create PR]
    end
    subgraph publish [ ]
        direction TB
        publish_a[Score publisher pipeline] -->publish_b
        publish_b[Publish scoring to DS API] -->publish_c
        publish_c[Publish scoring on-chain] -->publish_d
        publish_d[Emergency unstake] -->publish_e
        publish_e(( ))
    end
    scheduler_d -->scoring_a
    scoring_e -->|PR approved and merged| publish_a
```