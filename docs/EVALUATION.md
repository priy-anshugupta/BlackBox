# Evaluation and evidence

## What the saved results mean

The repository includes saved JSON metrics and trained model artifacts from the hackathon experiments. They are historical measurements on specific datasets, workflows, and environments. They were not rerun as part of preparing the personal repository.

`data/metrics.json` records the final in-family ensemble evaluation, including a 322-failure test split and baseline comparisons. `metrics_heldout_workflow.json` evaluates an expense graph that was absent from training: top-1 localization is 0.232 versus 0.4 for the rule baseline. This difference is central to understanding the current diagnosis model.

`metrics_natural.json` covers natural local-model failures, but its labels are concentrated on router argument loss. `metrics_live.json` evaluates bounded recovery against known injected faults. The public Who&When artifacts show weak zero-shot transfer to longer, free-text traces. None of these experiments supports a claim that arbitrary agent failures can always be diagnosed or repaired.

Read the sample counts, split definitions, model labels, and baseline fields in each JSON before comparing metrics. Provider/model names in the files identify the recorded experiment and do not assert current model availability.
