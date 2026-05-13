# Structure Fingerprint v0.1

A minimal specification for representing **multi-layer evidence** about a text artifact by separating:

- proof-oriented provenance
- inference-oriented structural similarity
- risk and uncertainty signals

This specification does **not** claim absolute authorship or absolute origin.
Instead, it defines a reproducible evidence object for reasoning about:

- lineage
- proximity
- origin candidacy
- downstream trace-based allocation

---

## Overview

Historically, proving “who said it first” was difficult in many cases,
especially when content was copied, paraphrased, partially quoted, or reposted
without durable provenance.

`Structure Fingerprint v0.1` is a minimal specification for handling that problem
as an engineering task.

The core idea is simple:

- **proof-oriented evidence** and
- **inference-oriented evidence**

must be stored and validated separately.

In other words, this specification distinguishes between:

- evidence that is closer to **proof**
- evidence that is closer to **inference**

This separation makes it possible to design trace systems without collapsing
provenance, structural similarity, and uncertainty into a single opaque score.

---

## Design Goals

This specification aims to provide:

1. a minimal, reproducible structure for text-oriented fingerprint objects
2. strict separation between proof, inference, and risk layers
3. JSON Schema validation for structural correctness
4. semantic validation for meaning-level consistency
5. a clean base for future trace, lineage, and allocation protocols

---

## Non-Goals

This specification does **not** attempt to provide:

- absolute authorship proof
- legal truth by itself
- a universal scoring formula for allocation
- a complete discourse parser or style engine
- a final standard for provenance interoperability

It is a **minimal foundation**, not a total solution.

---

## Core Principles

### 1. Separation of layers

Proof-oriented evidence and inference-oriented evidence MUST be stored and
validated in separate namespaces.

### 2. Deterministic recomputation

A fingerprint SHOULD be reproducible from the same input text under the same:

- canonicalization profile
- model versions

### 3. No single source of truth

A structure fingerprint does not assert absolute authorship or absolute origin.
It represents structured evidence for:

- lineage
- proximity
- origin candidacy

### 4. Explicit uncertainty

Uncertainty and attack-risk signals MUST be recorded explicitly and MUST NOT be
silently merged into provenance evidence.

---

## Object Model

A `Structure Fingerprint` object is organized into five top-level namespaces:

- `meta`
- `canon`
- `proof`
- `inference`
- `risk`

### Layer summary

#### `meta`

Object metadata such as:

- fingerprint ID
- creation time
- spec version
- source URI
- producer system

#### `canon`

Canonicalization context and canonical text hash, used to support
reproducibility.

#### `proof`

Proof-oriented evidence, such as:

- hard-binding hashes
- provenance manifests
- publication metadata
- signing status

#### `inference`

Inference-oriented structural signals, such as:

- style embedding references
- discourse graph references
- rhetorical operator distributions
- explanation spans

#### `risk`

Risk and uncertainty signals, such as:

- obfuscation risk
- impersonation risk
- paraphrase resilience
- confidence intervals
- model disagreement

---

## Repository Structure

```text
.
├── README.md
├── schemas/
│   └── structure-fingerprint-v0.1.schema.json
├── examples/
│   ├── structure-fingerprint.sample.json
│   └── invalid/
│       ├── missing-required-meta.schema-fail.json
│       ├── invalid-fingerprint-id.schema-fail.json
│       ├── span-order.semantic-fail.json
│       └── interval-order.semantic-fail.json
└── .github/
    └── workflows/
        └── validate-specs.yml
Specification Files
Main files
schemas/structure-fingerprint-v0.1.schema.json
JSON Schema (Draft 2020-12) for validating a Structure Fingerprint object
examples/structure-fingerprint.sample.json
canonical positive sample
.github/workflows/validate-specs.yml
CI workflow for schema and semantic validation
Negative test files
examples/invalid/*.schema-fail.json
samples that MUST fail at the schema layer
examples/invalid/*.semantic-fail.json
samples that MUST pass the schema layer but fail at the semantic layer
Minimal Field Structure

At minimum, a valid object contains:

meta
canon
proof
inference
risk
Minimal conceptual form
{
  "meta": {},
  "canon": {},
  "proof": {},
  "inference": {},
  "risk": {}
}

The detailed constraints are defined in:

schemas/structure-fingerprint-v0.1.schema.json
Schema Usage

This repository provides a JSON Schema and validation workflow for
Structure Fingerprint v0.1.

The goal is not to assert absolute authorship or absolute origin, but to
validate a structured evidence object that separates:

proof-oriented evidence
inference-oriented evidence
risk and uncertainty signals
Files
schemas/structure-fingerprint-v0.1.schema.json
JSON Schema (Draft 2020-12) for the Structure Fingerprint object
examples/structure-fingerprint.sample.json
canonical valid sample
examples/invalid/
intentionally invalid examples for negative testing
.github/workflows/validate-specs.yml
CI workflow for schema and semantic validation
What is validated

The validation pipeline checks two layers.

1. Schema validation

This layer validates the structural shape of the object:

JSON syntax validity
required fields
type constraints
enums
patterns
URI / date-time formats where declared
namespace separation of:
meta
canon
proof
inference
risk
2. Semantic validation

This layer validates cross-field meaning and internal consistency:

end_char >= start_char for explanation spans
interval.upper >= interval.lower
rhetorical operator weights are approximately 1.0
canon.canonical_text_hash and proof.hard_binding.content_hash
are checked for consistency when both are present
suspicious combinations such as very high confidence with high attack-risk
confidence values outside declared intervals
time-order anomalies such as publication_time > created_at
Local validation

Install dependencies:

python -m pip install --upgrade pip
pip install jsonschema

Run basic schema validation locally:

python - <<'PY'
import json
from pathlib import Path
from jsonschema import Draft202012Validator

schema_path = Path("schemas/structure-fingerprint-v0.1.schema.json")
sample_path = Path("examples/structure-fingerprint.sample.json")

with schema_path.open("r", encoding="utf-8") as f:
    schema = json.load(f)

with sample_path.open("r", encoding="utf-8") as f:
    sample = json.load(f)

validator = Draft202012Validator(schema)
errors = sorted(validator.iter_errors(sample), key=lambda e: list(e.absolute_path))

if errors:
    print("Schema validation failed:")
    for error in errors:
        path = ".".join(str(p) for p in error.absolute_path) or "<root>"
        print(f"- {path}: {error.message}")
    raise SystemExit(1)

print("Schema validation passed.")
PY
Design note

Passing validation does not imply absolute authorship proof.

This specification is designed around strict separation of layers:

proof.* for proof-oriented evidence
inference.* for inference-oriented structural signals
risk.* for attack-risk and uncertainty
canon.* for reproducible canonicalization context
meta.* for object metadata

Allocation, royalty, or downstream scoring decisions should be computed
outside the fingerprint object itself.

Negative Test Cases

This repository includes intentionally invalid examples under:

examples/invalid/

These files verify that the validation pipeline correctly rejects invalid
Structure Fingerprint objects.

Purpose

Negative tests make the boundary of the specification explicit.

They confirm that:

malformed objects fail at the schema layer
structurally valid but semantically inconsistent objects fail at the
semantic layer
the validator distinguishes between shape errors and meaning errors
Naming Convention

Invalid example files MUST follow one of these naming conventions:

*.schema-fail.json
*.semantic-fail.json
*.schema-fail.json

A file with .schema-fail. in its name is expected to fail during
JSON Schema validation.

Typical causes:

missing required fields
invalid patterns
invalid enum values
wrong data types
forbidden additional properties

Examples:

missing-required-meta.schema-fail.json
invalid-fingerprint-id.schema-fail.json
*.semantic-fail.json

A file with .semantic-fail. in its name is expected to pass
JSON Schema validation but fail during semantic validation.

Typical causes:

end_char < start_char
interval.upper < interval.lower
inconsistent cross-field values
contradictory metadata relationships

Examples:

span-order.semantic-fail.json
interval-order.semantic-fail.json
Validation expectations

The CI pipeline enforces the following rules:

examples/structure-fingerprint.sample.json MUST pass:
schema validation
semantic validation
Every file matching *.schema-fail.json MUST fail:
schema validation
Every file matching *.semantic-fail.json MUST:
pass schema validation
fail semantic validation

If an invalid test passes unexpectedly, the workflow fails.

If a semantic-fail test already fails at the schema layer, the workflow also
fails, because the test is misclassified.

Authoring guidance

When adding a new invalid example:

use .schema-fail. only when the object is expected to fail schema validation
use .semantic-fail. only when the object is expected to satisfy the schema
but violate semantic rules
keep each invalid sample focused on one main failure condition where possible
prefer small, minimal examples over large mixed-error cases

This keeps failures interpretable and regression checks sharp.

CI

GitHub Actions validates both the positive sample and the negative test set.

What CI checks

The workflow in .github/workflows/validate-specs.yml performs:

required file existence checks
JSON syntax checks
Draft 2020-12 schema validation
semantic validation for the valid sample
negative testing for invalid examples
Positive path

The canonical sample:

examples/structure-fingerprint.sample.json

must pass:

schema validation
semantic validation
Negative path

The invalid samples under:

examples/invalid/

must fail according to their filename class:

.schema-fail. → fail schema validation
.semantic-fail. → pass schema validation, fail semantic validation
Failure behavior

The workflow fails when:

a required file is missing
the schema JSON is invalid
the valid sample fails validation
a schema-fail test unexpectedly passes schema validation
a semantic-fail test unexpectedly passes semantic validation
a semantic-fail test fails too early at the schema layer
Why CI is split this way

JSON Schema is excellent at validating shape, type, and declared constraints.

It is less suited for higher-order meaning constraints such as:

span ordering
interval ordering
approximate weight totals
suspicious cross-field combinations

For that reason, this repository uses a two-stage validation model:

Schema layer for structural correctness
Semantic layer for meaning-level consistency

This separation improves:

debugging clarity
validator maintainability
auditability
extensibility for future rules
Practical implication

A sample can be:

schema-invalid
schema-valid but semantic-invalid
fully valid

That distinction is intentional and is part of the design of this repository.

Validation Philosophy

This repository is intentionally designed so that:

shape is validated by JSON Schema
meaning is validated by semantic checks
allocation is handled downstream
truth claims are never silently inferred from structural similarity alone

This helps preserve interpretability and auditability.

A structurally similar text is not automatically proof of origin.

A provenance chain is not automatically proof of authorship.

A confidence score is not automatically truth.

Each layer has its own role.

Intended Use

Structure Fingerprint v0.1 is intended as a minimal foundation for:

trace protocols
lineage analysis
origin-candidate ranking
structural similarity comparison
provenance-aware downstream allocation systems

It is especially suited for controlled or semi-controlled environments such as:

note/blog corpora
repository-based text archives
closed publication pools
experimental trace and allocation systems
Limitations

This specification has important limitations.

1. No absolute origin proof

This object does not prove absolute authorship or absolute first utterance.

2. Inference remains probabilistic

Style, discourse, and rhetorical signals remain inference-oriented and may be
affected by:

paraphrase
obfuscation
impersonation
model bias
language-specific ambiguity
3. Semantic checks are incomplete by design

The semantic layer currently captures only a small set of consistency checks.
Future versions may add stronger cross-field constraints.

4. Allocation is out of scope

This repository does not define a binding allocation formula.
It only prepares the evidence object that a downstream allocation layer may use.

Future Work

Possible next steps include:

comparison-result-v0.1.schema.json
lineage relation objects
stronger semantic rule sets
versioned canonicalization profiles
language-specific discourse profiles
provenance bridge profiles
allocation-readiness guidance
adversarial robustness testing
Versioning

Current version:

0.1.0

Version changes should be recorded whenever any of the following changes:

canonicalization logic
required fields
schema rules
semantic validation behavior
model-version assumptions
namespace conventions
Status

Draft.

This repository is currently a minimal engineering baseline for future
trace-oriented protocols.

License

Unless otherwise specified, this repository is intended to be published under:

CC0-1.0 for the specification text
implementation-specific licenses as needed for code or tooling

Adjust as appropriate for your actual repository policy.
