---
name: pydantic-evals
description: >-
  Guidelines for evaluating non-deterministic functions with pydantic-evals.
  Use when writing evals, defining datasets and cases, creating custom evaluators,
  or testing AI agent outputs with pydantic-evals.
license: MIT
---

# Evaluating Non-Deterministic Functions with pydantic-evals

pydantic-evals is a code-first framework for evaluating stochastic functions (LLM calls, agents, pipelines). Define test cases, run them against a task function, and score results with evaluators.

Install: `pip install pydantic-evals` (or `pip install 'pydantic-evals[logfire]'` for Logfire integration).

## Import Reference

```python
from pydantic_evals import Case, Dataset, set_eval_attribute, increment_eval_metric
from pydantic_evals.evaluators import (
    Evaluator, EvaluatorContext, EvaluatorOutput, EvaluationReason,
    ReportEvaluator, ReportEvaluatorContext,
    LLMJudge, HasMatchingSpan,
)
from pydantic_evals.evaluators.common import Equals, EqualsExpected, Contains, IsInstance, MaxDuration
from pydantic_evals.otel import SpanQuery               # requires logfire extra
from pydantic_evals.generation import generate_dataset  # LLM-based dataset generation
```

## Data Model

**Dataset -> Cases -> Evaluators -> EvaluationReport.** A `Dataset` holds `Case` objects and dataset-wide evaluators. Calling `dataset.evaluate(task_fn)` runs the task against all cases and returns an `EvaluationReport`. Both `Case` and `Dataset` are generic: `Case[InputsT, OutputT, MetadataT]`.

### Case

```python
case = Case(
    name="simple",                           # identifier (optional, but recommended)
    inputs="What is the capital of France?", # any type — passed to the task function
    expected_output="Paris",                 # optional — available via ctx.expected_output
    metadata={"difficulty": "easy"},         # optional — available via ctx.metadata
    evaluators=(MyEvaluator(),),             # optional — case-specific evaluators
)
```

### Dataset

```python
dataset = Dataset(
    cases=[case1, case2],
    evaluators=[GlobalEvaluator()],          # applied to every case
    report_evaluators=[MyReportEvaluator()], # experiment-wide analysis (optional)
)
```

| Method | Description |
|--------|-------------|
| `await dataset.evaluate(task_fn)` | Run task against all cases (async) |
| `dataset.evaluate_sync(task_fn)` | Synchronous wrapper |
| `dataset.add_case(...)` | Add a case after construction |
| `dataset.add_evaluator(ev, specific_case=None)` | Add evaluator to all cases or a named case |
| `Dataset.from_file("cases.yaml")` | Load from YAML or JSON |
| `dataset.to_file("cases.yaml")` | Save to YAML or JSON |

### EvaluatorContext

Every evaluator receives an `EvaluatorContext`:

| Field | Type | Description |
|-------|------|-------------|
| `inputs` | `InputsT` | The case inputs |
| `output` | `OutputT` | Actual task output |
| `expected_output` | `OutputT | None` | Expected output from the case |
| `metadata` | `MetadataT | None` | Case metadata |
| `name` | `str | None` | Case name |
| `duration` | `float` | Task execution time in seconds |
| `span_tree` | `SpanTree` | OpenTelemetry spans recorded during execution |
| `attributes` | `dict` | Runtime attributes set via `set_eval_attribute` |
| `metrics` | `dict` | Runtime metrics set via `increment_eval_metric` |

## Writing Evaluators

Subclass `Evaluator` and implement `evaluate` (sync or async). **Must use `@dataclass` decorator.**

### Return Types

`evaluate` returns `EvaluatorOutput`:

- **`bool`** — pass/fail (stored in `ReportCase.assertions`)
- **`int`/`float`** — numeric score (stored in `ReportCase.scores`)
- **`str`** — label (stored in `ReportCase.labels`)
- **`EvaluationReason(value, reason)`** — any of the above with an explanation
- **`dict[str, ...]`** — multiple named columns from a single evaluator (see [Multi-Score Evaluators](#multi-score-evaluators-dict-returns))

Single-scalar returns use the **evaluator class name** as the report column name (override with `evaluation_name` field).

```python
@dataclass
class ContainsExpected(Evaluator[str, str]):
    def evaluate(self, ctx: EvaluatorContext[str, str]) -> EvaluationReason:
        if ctx.expected_output is None:
            return EvaluationReason(value=False, reason="No expected output provided")
        found = ctx.expected_output.lower() in ctx.output.lower()
        return EvaluationReason(value=found, reason=f"{'found' if found else 'not found'}")
```

### Built-in Evaluators

| Evaluator | Fields | Description |
|-----------|--------|-------------|
| `EqualsExpected()` | — | Exact match against `expected_output` |
| `Equals(value=...)` | `value` | Exact match against a fixed value |
| `Contains(value=...)` | `value`, `case_sensitive`, `as_strings` | Substring/membership check |
| `IsInstance(type_name=...)` | `type_name` | Output type check |
| `MaxDuration(seconds=...)` | `seconds` | Asserts task completed within time limit |
| `LLMJudge(rubric=...)` | `rubric`, `model`, `include_input`, `include_expected_output` | LLM-based evaluation against a rubric |
| `HasMatchingSpan(query=...)` | `query` (`SpanQuery`) | Checks OpenTelemetry span tree for a matching span |

## Multi-Score Evaluators (Dict Returns)

When `evaluate` returns a `dict`, each key becomes a **separate named column** in the report. This lets a single evaluator produce multiple independent scores, assertions, or labels from one pass. Values are categorized by type (`bool` -> assertions, `int`/`float` -> scores, `str` -> labels, `EvaluationReason` -> unwrapped by inner `.value` type).

```python
@dataclass
class QualityEvaluator(Evaluator[QAInput, QAOutput]):
    """Single evaluator that produces multiple report columns."""

    def evaluate(self, ctx: EvaluatorContext[QAInput, QAOutput]) -> dict[str, EvaluationReason | bool | float]:
        output = ctx.output.answer
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

## Per-Case Evaluators

Cases can carry their own evaluators via `evaluators=(...)`. Dataset-wide evaluators run on every case; case-specific ones run only on that case. Both appear in the report.

```python
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

# Or add to a specific case after construction:
dataset.add_evaluator(MaxDuration(seconds=2.0), specific_case="fast_lookup")
```

## Report Evaluators

Report evaluators analyze results across all cases after case-level evaluation finishes. They receive a `ReportEvaluatorContext` with access to `ctx.report.cases`.

```python
@dataclass
class PassRate(ReportEvaluator[QAInput, QAOutput]):
    threshold: float = 0.8

    def evaluate(self, ctx: ReportEvaluatorContext[QAInput, QAOutput]) -> dict[str, float]:
        total = len(ctx.report.cases)
        passed = sum(1 for c in ctx.report.cases if c.assertions.get("AnswerContainsExpected"))
        rate = passed / total if total else 0.0
        return {"pass_rate": rate, "meets_threshold": float(rate >= self.threshold)}

dataset = Dataset(cases=[...], evaluators=[...], report_evaluators=[PassRate(threshold=0.9)])
```

## Reporting

`evaluate` / `evaluate_sync` return an `EvaluationReport` containing:
- `cases: list[ReportCase]` — successful results, each with `scores` (float), `labels` (str), `assertions` (bool), `metrics`, `task_duration`, `total_duration`
- `ReportCase` also includes `inputs`, `output`, `expected_output`, and `metadata`
- `failures: list[ReportCaseFailure]` — failed cases with `error_message` and `error_stacktrace`
- `ReportCaseFailure` also includes `inputs` and `expected_output`
- `analyses: list[ReportAnalysis]` — report-level analyses (confusion matrices, precision-recall, etc.)

```python
report.print(include_input=True, include_output=True, include_durations=False)
report.render()          # returns formatted string instead of printing
report.case_groups()     # grouped results when using repeat > 1
report.averages()        # aggregated statistics when using repeat > 1
```

## YAML Datasets

```python
dataset.to_file("my_cases.yaml")

dataset = Dataset[QAInput, QAOutput].from_file(
    "my_cases.yaml",
    custom_evaluator_types=(AnswerContainsExpected,),  # required for custom evaluator deserialization
)
```

## Evaluate Options

```python
report = await dataset.evaluate(
    my_agent,
    max_concurrency=5,      # limit parallel case execution
    repeat=3,               # run each case N times, results grouped by case name
    retry_task=2,           # retry task on failure
    retry_evaluators=1,     # retry evaluators on failure
    metadata={"run": "v2"}, # experiment-level metadata
)
```

## Dataset Generation

```python
dataset = await generate_dataset(
    dataset_type=Dataset[QAInput, QAOutput],
    n_examples=10,
    model="openai:gpt-4o",
    extra_instructions="Focus on geography questions of varying difficulty.",
    path="generated_cases.yaml",  # optionally persist to file
)
```

Always review generated cases — treat them as a starting point, not ground truth.

## Span-Based Evaluation

Assert on internal agent behavior via OpenTelemetry traces (requires `logfire` extra):

```python
Case(
    name="uses_tool",
    inputs=QAInput(question="What's the weather in Paris?"),
    evaluators=(HasMatchingSpan(query=SpanQuery(name_contains="weather_api")),),
)
```

The full `span_tree` is also available in custom evaluators via `ctx.span_tree`.

## Runtime Attributes and Metrics

Record data inside the task function that evaluators can access via `ctx.attributes` and `ctx.metrics`:

```python
async def my_agent(inputs: QAInput) -> QAOutput:
    set_eval_attribute("model_used", "gpt-4o")
    increment_eval_metric("llm_calls", 1)
    return QAOutput(answer="...")
```

## Common Pitfalls

**Custom evaluators must use `@dataclass`.** Without it, serialization and construction break silently.

```python
# WRONG
class MyEval(Evaluator[str, str]):
    def evaluate(self, ctx: EvaluatorContext[str, str]) -> bool: ...

# CORRECT
@dataclass
class MyEval(Evaluator[str, str]):
    def evaluate(self, ctx: EvaluatorContext[str, str]) -> bool: ...
```

**Always handle `expected_output is None`.** Cases may omit it. Evaluators that access `ctx.expected_output` without a guard will crash.

**The task function receives the full input object**, not unpacked fields:

```python
# WRONG
async def my_agent(query: str, context: str) -> str: ...

# CORRECT — receives the dataclass
async def my_agent(inputs: AgentInput) -> str:
    return f"Answer to {inputs.query} given {inputs.context}"
```
