# Composite Examples

## 1) Product and Box Pricing Tree

### Context
Order contains simple `Product` items and nested `Box` containers.

### Design
- Component: `PricedItem` with `getPrice()`.
- Leaf: `Product` returns direct price.
- Container: `Box` iterates children, sums `getPrice()`, adds packaging cost.

### Outcome
Client computes total with one call on root item, regardless of nesting depth.

## 2) Graphics Grouping Editor

### Context
Editor supports dots/circles and grouped compound shapes.

### Design
- Component: `Graphic` with `move`, `draw`.
- Leaves: `Dot`, `Circle`.
- Composite: `CompoundGraphic` delegates move/draw to children recursively.

### Outcome
Grouping and ungrouping work without client branching by shape type.

## 3) Organization Access Scope Tree

### Context
Permissions are inherited across org -> team -> project -> resource.

### Pattern Use
- Component operation: `effectivePermissions()`.
- Leaf computes local permission set.
- Container merges own policy with children recursively.

### Outcome
Uniform policy evaluation across hierarchy with centralized merge rules.

## 4) Quick Evaluation Prompts
- "Evaluate whether this hierarchy should use Composite or plain collections."
- "Refactor this leaf/container type-checking code into Composite."
- "Design a cycle-safe Composite tree with add/remove constraints."
- "Compare Composite vs Decorator for this nested behavior problem."
