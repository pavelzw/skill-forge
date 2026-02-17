---
name: pydantic-evals
description: >-
  Guidelines for evaluating non-deterministic functions with pydantic-evals.
  Use when writing evals, defining datasets and cases, creating custom evaluators,
  or testing AI agent outputs with pydantic-evals.
license: MIT
---

# Evaluating Non-Deterministic Functions with pydantic-evals

pydantic-evals is a code-first framework for evaluating stochastic functions (LLM calls, agents, pipelines). It provides a structured way to define test cases, run them against a task function, and score results with evaluators.

Install: `pip install pydantic-evals` (or `pip install 'pydantic-evals[logfire]'` for Logfire integration).

## Data Model

The core data model is: **Dataset -> Cases -> Evaluators -> EvaluationReport**.

- A `Dataset` holds a list of `Case` objects and dataset-wide `Evaluator` instances.
- Each `Case` defines inputs, optional expected output, optional metadata, and optional case-specific evaluators.
- Calling `dataset.evaluate(task_fn)` runs the task against all cases and returns an `EvaluationReport`.

### Case

```python
from pydantic_evals import Case

case = Case(
    name="simple",                          # identifier (optional, but recommended)
    inputs="What is the capital of France?", # any type — passed to the task function
    expected_output="Paris",                 # optional — available inside evaluators via ctx.expected_output
    metadata={"difficulty": "easy"},         # optional — available inside evaluators via ctx.metadata
    evaluators=(MyEvaluator(),),             # optional — case-specific evaluators
)
```

`Case` is generic: `Case[InputsT, OutputT, MetadataT]`. Use dataclasses or Pydantic models for structured inputs/outputs.

### Dataset

```python
from pydantic_evals import Dataset

dataset = Dataset(
    cases=[case1, case2],
    evaluators=[GlobalEvaluator()],  # applied to every case
)
```

Key methods:

| Method | Description |
|--------|-------------|
| `await dataset.evaluate(task_fn)` | Run task against all cases (async) |
| `dataset.evaluate_sync(task_fn)` | Synchronous wrapper |
| `dataset.add_case(...)` | Add a case after construction |
| `dataset.add_evaluator(evaluator)` | Add a dataset-wide evaluator |
| `Dataset.from_file("cases.yaml")` | Load from YAML or JSON |
| `dataset.to_file("cases.yaml")` | Save to YAML or JSON |

`evaluate` / `evaluate_sync` accept `max_concurrency` to limit parallel case execution and `repeat` to run each case multiple times.

### EvaluatorContext

Every evaluator receives an `EvaluatorContext` with these fields:

| Field | Type | Description |
|-------|------|-------------|
| `inputs` | `InputsT` | The case inputs |
| `output` | `OutputT` | Actual task output |
| `expected_output` | `OutputT \| None` | Expected output from the case |
| `metadata` | `MetadataT \| None` | Case metadata |
| `name` | `str \| None` | Case name |
| `duration` | `float` | Task execution time in seconds |
| `span_tree` | `SpanTree` | OpenTelemetry spans recorded during execution |
| `attributes` | `dict` | Runtime attributes set via `set_eval_attribute` |
| `metrics` | `dict` | Runtime metrics set via `increment_eval_metric` |

## Writing Evaluators

Subclass `Evaluator` and implement `evaluate`. The method can be sync or async.

### Return Types

`evaluate` returns `EvaluatorOutput`, which is one of:

- **`bool`** — pass/fail
- **`int`** or **`float`** — numeric score
- **`EvaluationReason(value, reason)`** — score with explanation
- **`dict[str, ...]`** — multiple named scores from a single evaluator

```python
from dataclasses import dataclass
from pydantic_evals.evaluators import Evaluator, EvaluatorContext, EvaluationReason

@dataclass
class ContainsExpected(Evaluator[str, str]):
    """Check if the expected output is contained in the actual output."""

    def evaluate(self, ctx: EvaluatorContext[str, str]) -> EvaluationReason:
        if ctx.expected_output is None:
            return EvaluationReason(value=False, reason="No expected output provided")
        found = ctx.expected_output.lower() in ctx.output.lower()
        return EvaluationReason(
            value=found,
            reason=f"Expected '{ctx.expected_output}' {'found' if found else 'not found'} in output",
        )
```

### Built-in Evaluators

Import from `pydantic_evals.evaluators` or `pydantic_evals.evaluators.common`:

| Evaluator | Fields | Description |
|-----------|--------|-------------|
| `EqualsExpected()` | — | Exact match against `expected_output` |
| `Equals(value=...)` | `value` | Exact match against a fixed value |
| `Contains(value=...)` | `value`, `case_sensitive`, `as_strings` | Substring/membership check |
| `IsInstance(type_name=...)` | `type_name` | Output type check |
| `MaxDuration(seconds=...)` | `seconds` | Asserts task completed within time limit |
| `LLMJudge(rubric=...)` | `rubric`, `model`, `include_input`, `include_expected_output` | LLM-based evaluation against a rubric |
| `HasMatchingSpan(query=...)` | `query` | Checks OpenTelemetry span tree for a matching span |

### LLMJudge

Use `LLMJudge` to evaluate subjective qualities with an LLM:

```python
from pydantic_evals.evaluators import LLMJudge

judge = LLMJudge(
    rubric="The response should be a concise, factually correct answer.",
    model="openai:gpt-4o",
    include_input=True,
    include_expected_output=True,
)
dataset = Dataset(cases=cases, evaluators=[judge])
```

## Complete Example

```python
import asyncio
from dataclasses import dataclass

from pydantic_evals import Case, Dataset
from pydantic_evals.evaluators import Evaluator, EvaluatorContext, EvaluationReason


@dataclass
class QAInput:
    question: str


@dataclass
class QAOutput:
    answer: str


@dataclass
class AnswerContainsExpected(Evaluator[QAInput, QAOutput]):
    def evaluate(self, ctx: EvaluatorContext[QAInput, QAOutput]) -> EvaluationReason:
        if ctx.expected_output is None:
            return EvaluationReason(value=False, reason="No expected output")
        found = ctx.expected_output.answer.lower() in ctx.output.answer.lower()
        return EvaluationReason(value=found)


async def my_agent(inputs: QAInput) -> QAOutput:
    # Replace with your actual agent/LLM call
    return QAOutput(answer=f"The answer to '{inputs.question}' is 42.")


async def main():
    dataset = Dataset(
        cases=[
            Case(
                name="capital",
                inputs=QAInput(question="What is the capital of France?"),
                expected_output=QAOutput(answer="Paris"),
            ),
            Case(
                name="color",
                inputs=QAInput(question="What color is the sky?"),
                expected_output=QAOutput(answer="blue"),
            ),
        ],
        evaluators=[AnswerContainsExpected()],
    )

    report = await dataset.evaluate(my_agent)
    report.print(include_input=True, include_output=True)


if __name__ == "__main__":
    asyncio.run(main())
```

## YAML Datasets

Datasets can be loaded from and saved to YAML files. This separates test data from evaluation logic.

```python
# Save
dataset.to_file("my_cases.yaml")

# Load
dataset = Dataset[QAInput, QAOutput].from_file(
    "my_cases.yaml",
    custom_evaluator_types=(AnswerContainsExpected,),
)
```

Pass `custom_evaluator_types` when loading so pydantic-evals can deserialize custom evaluator references in the YAML.

## Reporting

`evaluate` / `evaluate_sync` return an `EvaluationReport`.

### EvaluationReport

| Field | Type | Description |
|-------|------|-------------|
| `name` | `str` | Experiment name |
| `cases` | `list[ReportCase]` | Successful case results |
| `failures` | `list[ReportCaseFailure]` | Cases where the task raised an exception |
| `analyses` | `list[ReportAnalysis]` | Report-level analyses (confusion matrices, precision-recall, etc.) |
| `experiment_metadata` | `dict` | Metadata passed to `evaluate(metadata=...)` |

### ReportCase

Each successful case result contains:

| Field | Type | Description |
|-------|------|-------------|
| `inputs` | `InputsT` | Case inputs |
| `output` | `OutputsT` | Task output |
| `expected_output` | `OutputsT \| None` | Expected output |
| `metadata` | `MetadataT \| None` | Case metadata |
| `scores` | `dict[str, float]` | Numeric evaluator results (from `float`/`int` returns) |
| `labels` | `dict[str, str]` | Categorical evaluator results (from `str` returns) |
| `assertions` | `dict[str, bool]` | Boolean evaluator results (from `bool` returns) |
| `metrics` | `dict[str, float]` | Runtime metrics from `increment_eval_metric` |
| `task_duration` | `float` | Time spent in the task function |
| `total_duration` | `float` | Time including evaluators |

### ReportCaseFailure

| Field | Type | Description |
|-------|------|-------------|
| `inputs` | `InputsT` | Case inputs |
| `expected_output` | `OutputsT \| None` | Expected output |
| `error_message` | `str` | Exception message |
| `error_stacktrace` | `str` | Full traceback |

### Printing and Rendering

```python
report = dataset.evaluate_sync(my_agent)
report.print(
    include_input=True,
    include_output=True,
    include_durations=False,
)
```

`print` outputs a formatted table showing each case, its inputs/outputs, evaluator scores, and pass/fail status. Use `report.render()` to get the formatted string without printing.

When using `repeat > 1`, access grouped results with `report.case_groups()` and aggregated statistics with `report.averages()`.

## Per-Case Evaluators

Cases can have their own evaluators in addition to dataset-wide ones. Use this when certain cases need specialized checks that don't apply globally.

```python
from pydantic_evals.evaluators import LLMJudge, MaxDuration

dataset = Dataset(
    cases=[
        Case(
            name="fast_lookup",
            inputs=QAInput(question="What is 2+2?"),
            expected_output=QAOutput(answer="4"),
            evaluators=(MaxDuration(seconds=1.0),),  # only this case must be fast
        ),
        Case(
            name="complex_reasoning",
            inputs=QAInput(question="Explain quantum entanglement simply."),
            expected_output=None,
            evaluators=(
                LLMJudge(rubric="The explanation should be accurate and accessible to a layperson."),
            ),
        ),
    ],
    evaluators=[AnswerContainsExpected()],  # applied to ALL cases
)
```

Dataset-wide evaluators run on every case. Case-specific evaluators run only on that case. Both sets of results appear in the report.

You can also add an evaluator to a specific case after construction:

```python
dataset.add_evaluator(MaxDuration(seconds=2.0), specific_case="fast_lookup")
```

## Report Evaluators

Report evaluators analyze results across all cases (e.g., confusion matrices, precision-recall). They run after all case-level evaluators finish.

```python
from dataclasses import dataclass
from pydantic_evals.evaluators import ReportEvaluator, ReportEvaluatorContext

@dataclass
class PassRate(ReportEvaluator[QAInput, QAOutput]):
    threshold: float = 0.8

    def evaluate(self, ctx: ReportEvaluatorContext[QAInput, QAOutput]) -> dict[str, float]:
        total = len(ctx.report.cases)
        passed = sum(1 for c in ctx.report.cases if c.assertions.get("AnswerContainsExpected"))
        rate = passed / total if total else 0.0
        return {"pass_rate": rate, "meets_threshold": float(rate >= self.threshold)}
```

Add report evaluators to the dataset:

```python
dataset = Dataset(
    cases=[...],
    evaluators=[AnswerContainsExpected()],
    report_evaluators=[PassRate(threshold=0.9)],
)
```


## Retries and Repeat

Control reliability of evaluations with `retry_task`, `retry_evaluators`, and `repeat`:

```python
report = await dataset.evaluate(
    my_agent,
    max_concurrency=5,         # run at most 5 cases in parallel
    repeat=3,                  # run each case 3 times, results grouped by case
    retry_task=2,              # retry the task up to 2 times on failure
    retry_evaluators=1,        # retry evaluators up to 1 time on failure
)
```

`repeat` is useful for measuring variance in non-deterministic outputs. Results for repeated runs are grouped under the original case name.


## Runtime Attributes and Metrics

Inside your task function, use `set_eval_attribute` and `increment_eval_metric` to record data accessible in evaluators:

```python
from pydantic_evals import set_eval_attribute, increment_eval_metric

async def my_agent(inputs: QAInput) -> QAOutput:
    set_eval_attribute("model_used", "gpt-4o")
    increment_eval_metric("llm_calls", 1)
    # ...
    return QAOutput(answer="...")
```

These values appear in `ctx.attributes` and `ctx.metrics` inside evaluators.

## Import Reference

```python
# Core dataset and case types
from pydantic_evals import Case, Dataset, set_eval_attribute, increment_eval_metric

# Evaluator base classes and types
from pydantic_evals.evaluators import (
    Evaluator,
    EvaluatorContext,
    EvaluatorOutput,
    EvaluationReason,
    ReportEvaluator,
    ReportEvaluatorContext,
)

# Built-in evaluators
from pydantic_evals.evaluators.common import (
    Equals,
    EqualsExpected,
    Contains,
    IsInstance,
    MaxDuration,
)

# LLM-as-judge evaluator
from pydantic_evals.evaluators import LLMJudge

# Span-based evaluation (requires logfire extra)
from pydantic_evals.evaluators import HasMatchingSpan
from pydantic_evals.otel import SpanQuery

# Dataset generation
from pydantic_evals.generation import generate_dataset
```

## Multi-Score Evaluators (Dict Returns)

When `evaluate` returns a `dict`, each key becomes a **separate named column** in the evaluation report. This is the most powerful return type — it lets a single evaluator produce multiple independent scores, assertions, or labels from one evaluation pass.

The report categorizes each value by its type:
- `bool` values go into `ReportCase.assertions`
- `int`/`float` values go into `ReportCase.scores`
- `str` values go into `ReportCase.labels`
- `EvaluationReason` values are unwrapped and categorized by their inner `.value` type

```python
@dataclass
class QualityEvaluator(Evaluator[QAInput, QAOutput]):
    """Single evaluator that produces multiple report columns."""

    def evaluate(self, ctx: EvaluatorContext[QAInput, QAOutput]) -> dict[str, EvaluationReason | bool | float]:
        output = ctx.output.answer

        # Each key becomes a separate column in the report
        return {
            "is_nonempty": len(output.strip()) > 0,                          # -> assertions
            "answer_length": float(len(output)),                             # -> scores
            "contains_expected": EvaluationReason(                           # -> assertions (bool value)
                value=ctx.expected_output is not None
                    and ctx.expected_output.answer.lower() in output.lower(),
                reason=f"Output: {output[:50]}",
            ),
            "verbosity": EvaluationReason(                                   # -> scores (float value)
                value=min(len(output) / 100, 1.0),
                reason="Normalized length score",
            ),
        }
```

Without dict returns, you would need four separate evaluator classes to produce four columns. With dict returns, one evaluator handles all related checks in a single pass.

By contrast, returning a single scalar (e.g. `return True`) uses the **evaluator class name** as the column name. You can override this by setting `evaluation_name` on the evaluator instance.

## Common Pitfalls

**Custom evaluators must be dataclasses.** The `@dataclass` decorator is required, not just subclassing `Evaluator`. Without it, serialization and construction break silently.

```python
# WRONG — missing @dataclass
class MyEval(Evaluator[str, str]):
    def evaluate(self, ctx: EvaluatorContext[str, str]) -> bool:
        return ctx.output == ctx.expected_output

# CORRECT
@dataclass
class MyEval(Evaluator[str, str]):
    def evaluate(self, ctx: EvaluatorContext[str, str]) -> bool:
        return ctx.output == ctx.expected_output
```

**Always handle `expected_output is None`.** Cases may omit `expected_output`. Evaluators that access `ctx.expected_output` without a None check will crash on those cases.

```python
@dataclass
class MyEval(Evaluator[str, str]):
    def evaluate(self, ctx: EvaluatorContext[str, str]) -> EvaluationReason:
        if ctx.expected_output is None:
            return EvaluationReason(value=False, reason="No expected output")
        # ... safe to use ctx.expected_output here
```

**The task function receives the full input object.** If `InputsT` is a dataclass, the function receives the whole dataclass instance — not unpacked fields. The function can be sync or async; both work with `evaluate` and `evaluate_sync`.

```python
@dataclass
class AgentInput:
    query: str
    context: str

# WRONG — unpacked fields
async def my_agent(query: str, context: str) -> str: ...

# CORRECT — receives the dataclass
async def my_agent(inputs: AgentInput) -> str:
    return f"Answer to {inputs.query} given {inputs.context}"
```
