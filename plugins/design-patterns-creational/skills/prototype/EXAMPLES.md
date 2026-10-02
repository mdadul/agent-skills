# Prototype Examples

## 1) Shape Hierarchy Clone Example

### Context
`Shape` base class with `Circle` and `Rectangle` subclasses.

### Design
- `Shape` defines clone contract.
- `Circle` and `Rectangle` copy parent and own fields in clone constructor or clone method.
- Client clones from `Shape` interface without checking concrete type.

### Outcome
Polymorphic cloning across hierarchy without concrete-type coupling in client logic.

## 2) Preconfigured Notification Templates

### Context
Notification objects have many settings (channels, retry policy, localization, metadata).

### Pattern Use
- Build canonical prototypes (`critical`, `transactional`, `marketing`).
- Clone template, mutate only request-specific fields.
- Avoid creating numerous subclasses for each preset.

### Outcome
Consistent defaults and reduced constructor/setup duplication.

## 3) Prototype Registry Example

### Context
Game entities need many preset configurations (`archer`, `tank`, `mage`).

### Pattern Use
- Registry maps key to prototype.
- Spawn pipeline requests key, registry returns clone.
- Clone policy deep-copies mutable combat stats, shares immutable assets.

### Outcome
Fast instance creation with controlled copy semantics.

## 4) Quick Evaluation Prompts
- "Assess if Prototype fits this copy-heavy module and define clone policy."
- "Refactor this manual deep-copy logic into Prototype with tests."
- "Design a prototype registry for these preset objects and show tradeoffs."
- "Compare Prototype vs Builder for this configuration workflow."
