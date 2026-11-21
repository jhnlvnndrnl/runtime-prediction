# BigO(Bench dataset

## Overview

This project uses [BigO(Bench](https://huggingface.co/datasets/facebook/BigOBench), a benchmark for understanding and generating code with controlled computational complexity. It contains competitive-programming problems, Python solutions, test inputs and outputs, and complexity labels produced by an automated complexity-analysis framework.

BigO(Bench is designed to evaluate whether a model can infer time and space complexity, generate solutions that satisfy complexity constraints, and reason about runtime and memory as input sizes grow.

The dataset is associated with the paper [BigO(Bench -- Can LLMs Generate Code with Controlled Time and Space Complexity?](https://arxiv.org/abs/2503.15242). The Hugging Face dataset is marked synthetic and uses the `CC-BY-NC-4.0` license; check the dataset page for current terms.

## Dataset contents

The full BigO(Bench collection contains five JSONL subsets:

| File | Purpose |
|---|---|
| `problem_and_human_solutions_list.jsonl` | Problem descriptions, human solutions, tests, metadata, and input information. |
| `complexity_labels_light.jsonl` | Compact inferred time/space labels and curve coefficients. |
| `complexity_labels_full.jsonl` | Detailed labels plus runtime and memory measurements across input sizes. |
| `time_complexity_test_set.jsonl` | Filtered set for time-complexity evaluation. |
| `space_complexity_test_set.jsonl` | Filtered set for space-complexity evaluation. |

The full collection contains roughly 3,105 coding problems and 1.19 million solutions. The benchmark subsets are filtered from the broader problem and complexity-label files.

## Subset used by this project

The notebooks load:

| Property | Value |
|---|---|
| Dataset | `facebook/BigOBench` |
| Configuration | `time_complexity_test_set.jsonl` |
| Split | `train` |
| Rows | 640 solutions |
| Problems | 311 |
| Complexity classes | 11 |

Each row represents one Python solution associated with a programming problem. A problem can appear in multiple rows because it may have multiple solutions. Thus, 640 rows are solutions, not 640 unique problems.

The subset covers 11 classes. Common classes include `O(n)`, `O(n log n)`, `O(n²)`, `O(1)`, and `O(n × m)`; less common classes include `O((n + m) log(n + m))`.

## Schema of the time-complexity subset

| Field | Type | Description |
|---|---|---|
| `problem_name` | string | Human-readable problem name. |
| `problem_id` | string | Identifier shared by solutions for the same problem. |
| `solution_id` | string | Identifier for one solution. |
| `description` | string | Programming-problem statement. |
| `solution_code` | string | Python source code. |
| `dataclass_code` | string | Python code used to represent or parse problem input. |
| `inputs_example` | string | Example input used in analysis. |
| `time_complexity_inferred` | string | Inferred asymptotic class, such as `O(n)` or `O(n²)`. |
| `time_curve_coefficient` | float | Numeric coefficient describing the fitted runtime-growth curve. |
| `tests` | object | Public, private, and generated test cases. |
| `problem_time_curve_coefficient_list` | float array | Coefficients associated with the problem across its solutions or measurements. |

The `tests` object contains `public_tests`, `private_tests`, and `generated_tests`. Each test entry contains an `input` string and an `output` string.

## Meaning of the complexity fields

`time_complexity_inferred` is a categorical asymptotic label describing runtime growth as input size increases:

`O(1)` is constant, `O(n)` is linear, `O(n log n)` is linearithmic, `O(n²)` is quadratic, and `O(n × m)` depends on two input dimensions.

`time_curve_coefficient` is a numeric output from runtime-curve analysis. It is not elapsed time in seconds. The complexity framework runs solutions over multiple input sizes, measures runtime and memory behavior, and infers complexity from those measurements.

## How this project uses the data

The current notebooks select only two fields:

```python
X_raw = data["solution_code"].str.len().to_numpy().reshape(-1, 1)
y = data["time_curve_coefficient"].to_numpy()
```

Each sample is one Python solution. The only model feature is `Code_Length`, the number of characters in `solution_code`; the regression target is `time_curve_coefficient`. Other fields are not currently model inputs.

`Code_Length` is a project-derived feature, not an original BigO(Bench field. It counts characters, including formatting and comments, and is not the same as lines of code or algorithmic complexity.

## Limitations

- Solutions from the same problem are related and should not automatically be treated as independent observations.
- Inferred asymptotic labels are not guarantees for every machine or input distribution.
- Runtime measurements depend on the benchmark environment and measurement procedure.
- The coefficient is not wall-clock seconds.
- Source-code length cannot capture control flow, input size, data structures, implementation quality, or hardware effects.
- Generated annotations should be distinguished from human-provided ground truth.

## Loading the subset

```python
from datasets import load_dataset

data = load_dataset(
    "facebook/BigOBench",
    "time_complexity_test_set.jsonl",
    split="train",
)
```

The [official dataset page](https://huggingface.co/datasets/facebook/BigOBench) is the source of truth for the current schema, revisions, license, and citation requirements.

