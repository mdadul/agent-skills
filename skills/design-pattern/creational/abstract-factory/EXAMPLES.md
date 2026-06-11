# Abstract Factory Examples

## 1) Furniture Family Example

### Context
Product types: `Chair`, `Sofa`, `CoffeeTable`.
Variants: `Modern`, `Victorian`, `ArtDeco`.

### Design
- Abstract products: `Chair`, `Sofa`, `CoffeeTable`.
- Abstract factory: `FurnitureFactory` with `createChair`, `createSofa`, `createCoffeeTable`.
- Concrete factories: `ModernFurnitureFactory`, `VictorianFurnitureFactory`, `ArtDecoFurnitureFactory`.

### Outcome
Selecting one factory guarantees matching furniture style across all created products.

## 2) Cross-Platform UI Example

### Context
UI must create `Button` and `Checkbox` that match current OS style.

### Design
- Abstract products: `Button`, `Checkbox`.
- Abstract factory: `GUIFactory`.
- Concrete factories: `WinFactory`, `MacFactory`.
- App selects factory at startup using environment/config.

### Outcome
Client code stays unchanged while platform-specific variant changes by chosen factory.

## 3) Multi-Tenant Theme Kit Example

### Context
SaaS app supports tenant themes; each theme needs coordinated widgets.

### Pattern Use
- Product types: `PrimaryButton`, `Modal`, `FormField`.
- Variant factories: `DefaultThemeFactory`, `EnterpriseThemeFactory`.
- Composition root picks factory from tenant config.

### Outcome
No mixing of mismatched themed components at runtime.

## 4) Quick Evaluation Prompts
- "Evaluate whether this code should use Abstract Factory or Factory Method."
- "Map my product matrix and generate an Abstract Factory design."
- "Refactor these constructor-heavy UI builders into Abstract Factory step by step."
- "Show an Abstract Factory example in TypeScript with startup-time factory selection."
