# Flyweight Examples

## 1) Particle System / Game (Canonical RAM Problem)

### Before
```
class Particle {
    x: number       // extrinsic — changes every frame
    y: number       // extrinsic
    velocityX: number  // extrinsic
    velocityY: number  // extrinsic
    color: Color    // intrinsic — all bullets are the same color (500KB sprite)
    sprite: Texture // intrinsic — same 2MB texture per particle type
}
// 10,000 bullets × ~2MB sprite = ~20GB → crash
```

### After
- **Flyweight** (`ParticleType`): `name`, `color`, `sprite` — immutable, shared.
- **Context** (`Particle`): `x`, `y`, `velocityX`, `velocityY`, `type: ParticleType` — lightweight.
- **Factory** (`ParticleFactory`): pool keyed by `name`.

```typescript
class ParticleType {
    constructor(
        readonly name: string,
        readonly color: Color,
        readonly sprite: Texture  // 2MB — loaded once
    ) {}

    draw(canvas: Canvas, x: number, y: number) {
        // render sprite at (x, y)
    }
}

class ParticleFactory {
    private pool = new Map<string, ParticleType>();

    getType(name: string, color: Color, sprite: Texture): ParticleType {
        if (!this.pool.has(name))
            this.pool.set(name, new ParticleType(name, color, sprite));
        return this.pool.get(name)!;
    }
}

class Particle {
    constructor(
        public x: number,
        public y: number,
        public vx: number,
        public vy: number,
        readonly type: ParticleType  // shared reference, not a copy
    ) {}

    draw(canvas: Canvas) { this.type.draw(canvas, this.x, this.y); }
}
```

### Memory Impact
```
Before: 10,000 × 2MB sprite ≈ 20GB
After:  3 flyweights × 2MB + 10,000 × ~40 bytes ≈ 6MB + 400KB ≈ 6.4MB
```

---

## 2) Forest Rendering (GoF Tree Example)

### Design
- **Flyweight** (`TreeType`): `name`, `color`, `texture` — loaded once per species.
- **Context** (`Tree`): `x`, `y`, `type: TreeType`.
- **Factory** (`TreeFactory`): static pool keyed by `(name, color, texture)`.
- **Container** (`Forest`): holds `Tree[]`; calls `TreeFactory.getTreeType(...)` when planting.

```
Forest.plantTree(x=10, y=20, name="Oak", color=green, texture=oakTex)
  → factory finds or creates TreeType("Oak", green, oakTex)
  → creates Tree(x=10, y=20, type=oakFlyweight)
```

1,000,000 trees of 5 species → 5 flyweights + 1,000,000 tiny context objects.

---

## 3) Text Rendering — Character Glyphs

### Context
A text editor renders millions of characters. Each glyph (shape, font, size) is identical for every `'A'` in the same font — only the position on screen differs.

### Design
- **Flyweight** (`Glyph`): `char`, `font`, `size`, `bitmap` — immutable, shared per unique character+font combination.
- **Context** (`CharacterPosition`): `x`, `y`, `glyph: Glyph`.
- **Factory** (`GlyphCache`): pool keyed by `(char, font, size)`.

A 100,000-character document in one font needs at most ~95 unique glyph objects (printable ASCII), not 100,000.

---

## 4) CSS Class Sharing in UI Frameworks

### Context
A UI renders 50,000 table cells. Each cell has a style object: `{ fontSize: 14, color: '#333', padding: 4, fontWeight: 'normal' }`. Most cells share the same style.

### Design
- **Flyweight**: immutable style record keyed by a hash of its values.
- **Factory**: `StyleCache.get({ fontSize, color, padding, fontWeight })` — returns or creates.
- **Context**: each cell holds a reference to the shared style flyweight.

50,000 cells sharing 10 style variants → 10 style objects instead of 50,000.

---

## 5) Flyweight vs Singleton vs Object Pool

| | Flyweight | Singleton | Object Pool |
|---|---|---|---|
| **Instance count** | One per unique intrinsic state | Exactly one | Fixed or growing set |
| **Mutability** | Immutable | May be mutable | Mutable (recycled) |
| **Purpose** | Reduce RAM via sharing identical state | Single shared resource | Avoid GC / creation cost |
| **Identity** | Value-like (not entity) | Entity | Entity |
| **Returned to pool?** | No — shared indefinitely | N/A | Yes — after use |
| **Thread safety** | Inherent (immutable) | Must be managed | Must be managed |

---

## 6) When NOT to Use Flyweight

| Scenario | Why Flyweight doesn't help |
|---|---|
| 1,000 objects with unique state | N ≈ K — no sharing possible, just added complexity |
| Objects need to track mutable shared state | Mutability breaks the pattern's safety guarantee |
| Extrinsic state is large or expensive to compute | CPU cost of passing state offsets RAM savings |
| RAM is not the bottleneck | Premature optimization — profile first |

---

## 7) Quick Evaluation Prompts
- "My game is crashing due to RAM — would Flyweight help?"
- "How do I split this class into intrinsic and extrinsic state?"
- "Show me a Flyweight Factory in TypeScript for a particle system."
- "What's the difference between Flyweight and an Object Pool?"
- "Is Flyweight overkill for 10,000 objects?"
