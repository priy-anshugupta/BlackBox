# Evaluation and evidence

## What the saved results mean

The repository includes saved JSON metrics and trained model artifacts from the hackathon experiments. They are historical measurements on specific datasets, workflows, and environments. They were not rerun as part of preparing the personal repository.

`data/metrics.json` records the final in-family ensemble evaluation, including a 322-failure test split and baseline comparisons. `metrics_heldout_workflow.json` evaluates an expense graph that was absent from training: top-1 localization is 0.232 versus 0.4 for the rule baseline. This difference is central to understanding the current diagnosis model.

`metrics_natural.json` covers natural local-model failures, but its labels are concentrated on router argument loss. `metrics_live.json` evaluates bounded recovery against known injected faults. The public Who&When artifacts show weak zero-shot transfer to longer, free-text traces. None of these experiments supports a claim that arbitrary agent failures can always be diagnosed or repaired.

Read the sample counts, split definitions, model labels, and baseline fields in each JSON before comparing metrics. Provider/model names in the files identify the recorded experiment and do not assert current model availability.

## Reproduction boundaries

Generated SQLite datasets, caches, usage ledgers, and some auxiliary metric files are deliberately excluded. Dataset manifest files describe some saved generation settings but do not replace the full underlying traces. Reproducing the numbers requires the original data or a new, clearly distinguished generation/evaluation run.

Keep fresh experiments in separate database, metrics, and model paths. Never overwrite committed final metrics to make new experiments appear identical to the original results. Start with sandbox data and inspect each module's `--help` before running generation or training.

Useful entry points include `blackbox.datagen`, `blackbox.models.train`, `blackbox.eval_natural`, `blackbox.live_eval`, `blackbox.hard_negatives`, `blackbox.benchmarks.heldout_workflow`, and `blackbox.benchmarks.whowhen`. Some require generated inputs or external benchmark data; a module name is not a promise of a self-contained reproduction command.

Record provider, model, dataset split, seed, feature availability, hardware, run count, and output paths for each new result. Billed calls are optional and should be explicitly configured with an appropriate budget.
