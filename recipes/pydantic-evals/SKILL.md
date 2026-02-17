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

`evaluate` / `evaluate_sync` return an `EvaluationReport` with a `print` method:

```python
report = dataset.evaluate_sync(my_agent)
report.print(
    include_input=True,
    include_output=True,
    include_durations=False,
)
```

This prints a formatted table showing each case, its inputs/outputs, evaluator scores, and pass/fail status.

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
