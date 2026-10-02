# Template Method Examples

## 1) Multi-Format Document Mining

### Context
DOC, CSV, and PDF processors share extract -> analyze -> report flow with format-specific parsing.

### Design
- Base class defines template method: `processDocument()`.
- Abstract steps: `open()`, `parseRawData()`, `close()`.
- Default steps: `analyzeData()`, `generateReport()`.
- Optional hooks before/after analysis for specialized behavior.

### Outcome
Algorithm order is fixed while format-specific parsing remains customizable.

## 2) Game AI Turn Loop

### Context
Different factions follow same turn phases but differ in build/attack behavior.

### Pattern Use
- Template method: `turn()`.
- Shared default step: `collectResources()`.
- Abstract steps: `buildStructures()`, `buildUnits()`, `sendScouts()`, `sendWarriors()`.

### Outcome
New factions are added via subclass overrides without modifying turn orchestration.

## 3) ETL Pipeline Variants

### Context
Pipelines share validation and audit flow but vary in source extraction and transform logic.

### Pattern Use
- Base `runPipeline()` enforces `loadConfig -> validate -> extract -> transform -> persist -> audit`.
- Subclasses override `extract`/`transform`; defaults cover validation and auditing.

### Outcome
Compliance steps remain guaranteed while source-specific logic is isolated.

## 4) Quick Evaluation Prompts
- "Evaluate if this duplicated workflow should use Template Method or Strategy."
- "Refactor these similar classes into a base template with hooks."
- "Design abstract/default/hook step boundaries for this algorithm."
- "Compare Template Method vs Factory Method in this inheritance hierarchy."
