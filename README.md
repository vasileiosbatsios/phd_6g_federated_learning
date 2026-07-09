# PhD 6G Federated Learning Results

Experimental results from a comparative performance evaluation of federated learning frameworks for edge-oriented 6G applications.

This repository stores benchmark outputs only. It does not contain the benchmark harness or framework integrations used to produce the runs.

## Study

**Comparative Performance Evaluation of Federated Learning Frameworks for Edge-Oriented 6G Applications**

| Framework | Identifier in filenames |
|-----------|-------------------------|
| [FedML](https://github.com/FedML-AI/FedML) | `fedml` |
| [Flower](https://github.com/adap/flower) | `flower` |
| [NVIDIA FLARE](https://github.com/NVIDIA/NVFlare) | `nvflare` |
| [OpenFL](https://github.com/securenlg/openfl) | `openfl` |
| [PySyft](https://github.com/OpenMined/PySyft) | `pysyft` |
| [TensorFlow Federated](https://www.tensorflow.org/federated) | `tff` |

**Datasets:** MNIST, CIFAR-10

**Typical run configuration**

- 6 clients, 10 federated rounds
- IID data partition
- Batch size 32, 1 local epoch per round, learning rate 0.01
- Ideal network profile (0 ms latency, 10 Gbps bandwidth, no stragglers)

## Repository layout

```
results/
└── Comparative Performance Evaluation of Federated Learning Frameworks for Edge-Oriented 6G Applications/
    ├── {framework}_{YYYYMMDD}_{HHMMSS}.json    # latest run exports (root)
    ├── {framework}_{YYYYMMDD}_{HHMMSS}.csv
    └── results_archive/
        ├── paper1/                             # MNIST campaign (one folder per framework)
        ├── paper1-2/                           # CIFAR-10 campaign
        ├── paper1_cifar10_edge/                # edge-oriented Flower subset
        ├── paper1_summary.json                 # aggregated MNIST results (7 runs)
        └── paper1-2_summary.json               # aggregated CIFAR-10 results (9 runs)
```

Archived runs are grouped under `results_archive/` by campaign. The study root also holds the most recent exported copies of each run.

## File formats

### JSON (`*.json`)

Full run record. Top-level fields include experiment metadata (`framework`, `dataset`, `network_profile`, `num_clients`, `num_rounds`, `wall_time_sec`, `implementation`, and others) plus a `rounds` array with per-round metrics.

### CSV (`*.csv`)

Tabular per-round export with columns:

`round_number`, `train_loss`, `train_accuracy`, `eval_loss`, `eval_accuracy`, `fit_duration_sec`, `network_delay_sec`, `comm_bytes_up`, `comm_bytes_down`, `num_clients`

### Summary files (`*_summary.json`)

Combine multiple framework runs into a single JSON document. Each entry in `frameworks` mirrors the structure of an individual run JSON file. Use these for cross-framework comparison without loading every per-run file.

### Global model snapshots

TensorFlow Federated runs may include global model checkpoints as JSON, for example:

`tff_{dataset}_{timestamp}_global_model.json`

Model weight files (`.pt`) are referenced in run metadata via `model_checkpoint_path` but are not stored in this repository.

## Experiment campaigns

| Campaign | Dataset | Frameworks | Summary file |
|----------|---------|------------|--------------|
| `paper1` | MNIST | FedML, Flower, NVIDIA FLARE, OpenFL, PySyft, TFF (2 runs) | `results_archive/paper1_summary.json` |
| `paper1-2` | CIFAR-10 (+ 1 MNIST Flower baseline) | FedML, Flower (3 runs), NVIDIA FLARE, OpenFL, PySyft, TFF (2 runs) | `results_archive/paper1-2_summary.json` |
| `paper1_cifar10_edge` | Edge-oriented Flower runs | Flower | — |

## Using the data

**Quick overview** — open a `*_summary.json` file and inspect the `frameworks` array.

**Per-run analysis** — load the matching `.json` for full metadata and round-by-round metrics, or the `.csv` for tabular plotting.

**Example (Python)**

```python
import json
from pathlib import Path

summary = Path(
    "results/Comparative Performance Evaluation of Federated Learning Frameworks "
    "for Edge-Oriented 6G Applications/results_archive/paper1_summary.json"
)
data = json.loads(summary.read_text())

for run in data["frameworks"]:
    last = run["rounds"][-1]
    print(
        f"{run['framework']:8s}  "
        f"acc={last['eval_accuracy']:.4f}  "
        f"wall_time={run['wall_time_sec']:.0f}s"
    )
```

## File naming convention

```
{framework}_{YYYYMMDD}_{HHMMSS}.{json|csv}
```

The timestamp reflects when the run started (UTC).
