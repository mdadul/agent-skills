---
name: expo-modules-api
description: Builds, extends, and reviews Expo native modules with the Modules API (Swift/Kotlin ModuleDefinition DSL, expo-modules-core, views, events, Records, Android Package lifecycle listeners). Use when creating a local or published Expo module, writing native code in ios/ or android/, hooking MainActivity or Application without manual MainActivity edits, deep links, ReactActivityLifecycleListener, or bridging requireNativeModule to JavaScript events.
---

# Expo Modules API

## Purpose

The **Expo Modules API** is the recommended way to build **native modules** for Expo and React Native. It sits on [JSI](https://reactnative.dev/architecture/glossary#javascript-interfaces-jsi) and exposes a **declarative DSL** in Swift and Kotlin so one module definition stays consistent across iOS and Android.

Use this skill whenever someone is **building, extending, or debugging a module** — not only when they mention "Module API" by name. Typical work: new `expo-*` package, app-local module in `modules/`, adding native methods to an existing module, or shipping a **native view** to React.

**Read order:** workflows below → [REFERENCE.md](REFERENCE.md) for every DSL component → [EXAMPLES.md](EXAMPLES.md) for copy-paste starters.

## When to use

| Situation | Use this skill |
|-----------|----------------|
| `npx create-expo-module` or new `modules/my-lib` | Full scaffold + definition DSL |
| `ios/*Module.swift`, `android/**/*Module.kt` | `Module` + `ModuleDefinition` |
| JS can't find native method | `Name`, autolinking, rebuild native |
| Need Promise from native | `AsyncFunction` (not blocking `Function`) |
| Custom RN component with native UI | `View` + `ExpoView` + `Prop` / `EventDispatcher` |
| Native pushes updates to JS | `Events` + `sendEvent` or view callbacks |
| Options object from JS | `Record` + `@Field` |
| Platform-only lifecycle (camera, activity result) | Module DSL: [REFERENCE.md#lifecycle](REFERENCE.md#lifecycle) |
| Android deep link / `onCreate` without editing MainActivity | [Package lifecycle listeners](REFERENCE.md#android-lifecycle-listeners) |
| Bridge Android system events → JS | Observer + `Events` + `sendEvent` — [EXAMPLES.md#android-lifecycle-to-js](EXAMPLES.md#android-lifecycle-to-js) |

Official doc index: [llms.txt](https://docs.expo.dev/llms.txt). Getting started: [Expo Modules — Get Started](https://docs.expo.dev/modules/get-started/).

## What you are building

A standard Expo module usually contains:

```text
my-module/
├── expo-module.config.json    # module metadata for autolinking
├── src/index.ts               # JS/TS public API (requireNativeModule)
├── ios/MyModule.swift         # class MyModule: Module
├── android/.../MyModule.kt    # class MyModule : Module()
└── (optional) src/MyView.tsx  # requireNativeViewManager wrapper
```

**Core rule:** every exported native capability is declared inside `definition() -> ModuleDefinition { ... }`. JavaScript reaches it via `requireNativeModule('Name')` or `requireNativeViewManager('ViewName')` — names must match `Name("...")` in native code.

## Module types (pick a shape)

| Type | Native surface | JS entry |
|------|----------------|----------|
| **Headless API** | `Function`, `AsyncFunction`, `Property`, `Constant`, `Events` | `requireNativeModule('X')` |
| **Native view** | `View { Prop, Events, AsyncFunction }` + `ExpoView` subclass | `requireNativeViewManager('X')` → React component |
| **Hybrid** | Both module methods and `View` in same `Module` class | Module + view manager names may differ |
| **Events-only** | `Events` + observers + `sendEvent` | `addListener` / `useEvent` |

## Building a headless module (step-by-step)

### 1. Scaffold

- Prefer `npx create-expo-module@latest my-module` for publishable packages.
- For monorepos, follow [Expo monorepo / local modules](https://docs.expo.dev/modules/get-started/) docs; ensure `expo-module.config.json` exists and the app depends on the package.

### 2. Native module class

**Swift** — `public class MyModule: Module` with `public func definition() -> ModuleDefinition`.

**Kotlin** — `class MyModule : Module()` with `override fun definition() = ModuleDefinition { }`.

### 3. Declare the JS name

```swift
Name("MyModule")
```

```kotlin
Name("MyModule")
```

This string is what `requireNativeModule('MyModule')` uses.

### 4. Export behavior

| Need | DSL |
|------|-----|
| Sync, cheap, JS thread | `Function("method") { (arg: String) in ... }` |
| Async / I/O / heavy | `AsyncFunction("method") { ... }` |
| Cached read-once value | `Constant("version") { "1.0" }` |
| Live readable/writable field | `Property("enabled").get { }.set { }` |
| Push to JS listeners | `Events("onChange")` + `sendEvent(...)` |

**Threading default:** `Function` runs on the **JavaScript thread** and **blocks** the app until it returns. `AsyncFunction` returns a **Promise** and runs off the JS thread by default — use it for network, disk, long CPU, and most native SDK calls.

### 5. TypeScript public API

```ts
import { requireNativeModule } from 'expo-modules-core';

const Native = requireNativeModule('MyModule');

export async function doWork(input: string): Promise<string> {
  return Native.doWorkAsync(input);
}
```

Type event modules with `NativeModule<Events>` (see [EXAMPLES.md](EXAMPLES.md)).

### 6. Verify

- Rebuild native (`npx expo run:ios` / `run:android` or dev client).
- Call each export from JS; confirm Promise vs sync behavior.
- If listeners use battery/CPU, wire `OnStartObserving` / `OnStopObserving`.

## Building a native view module (step-by-step)

### 1. Native view class

- **Android:** must extend `ExpoView(context, appContext)`.
- **iOS:** extend `ExpoView` (extends `RCTView`) for `appContext` and style/accessibility helpers.

Do **not** change constructor parameters — `expo-modules-core` constructs the view.

### 2. Register in `definition()`

```swift
View(MyView.self) {
  Name("MyView")           // optional but recommended
  Prop("color") { view, color in ... }
  Events("onPress")
  AsyncFunction("focus") { view in view.becomeFirstResponder() }
}
```

### 3. View events

In the view class, declare `EventDispatcher` properties **with the same names** as in `Events("onPress")`:

```swift
class MyView: ExpoView {
  let onPress = EventDispatcher()
}
```

Invoke `onPress(["key": value])` from native; JS receives `event.nativeEvent`.

### 4. React usage

```tsx
const NativeView = requireNativeViewManager('MyView');
<NativeView onPress={(e) => {}} ref={ref} />
// ref.current?.focus() for view AsyncFunctions
```

View `AsyncFunction` runs on the **main/UI queue**; first parameter is always the **view instance**.

## Quick decisions

| Goal | Component | Notes |
|------|-----------|--------|
| Export module name | `Name("MyModule")` | Must match `requireNativeModule` |
| One-time constant | `Constant("key") { }` | Not deprecated `Constants` |
| Sync on JS thread | `Function` | Max **8** args; blocks UI if slow |
| Promise / background work | `AsyncFunction` | Prefer over `Function` for I/O |
| Main-thread async | `.runOnQueue(.main)` / `Queues.MAIN` | |
| Kotlin suspend | `AsyncFunction("x") Coroutine { }` | No `Promise` param in body |
| Module property | `Property` | `.get` / `.set` |
| React native component | `View` + `ExpoView` | |
| Ref imperative API | `AsyncFunction` inside `View` | |
| Module → JS broadcast | `Events` + `sendEvent` | |
| View → JS callback prop | `Events` + `EventDispatcher` | Not `sendEvent` |
| Typed options from JS | `Record` + `@Field` | |
| Custom type from JS (iOS) | `Convertible` | |
| Custom type from JS (Android) | `ModuleConverters` | |
| Optional vs undefined (exp.) | `ValueOrUndefined` | |
| Mutate JS object | `JavaScriptObject` in `Function` only | JS thread or crash |
| Android Activity without MainActivity edits | `Package` + `ReactActivityLifecycleListener` | See [REFERENCE.md#android-lifecycle-listeners](REFERENCE.md#android-lifecycle-listeners) |
| Application.onCreate globally | `ApplicationLifecycleListener` | Same Package class |

## Argument typing (module builders)

Prefer **typed bridges** over raw maps:

1. **Primitives** — `String`, `Int`, `Bool`, arrays, maps.
2. **`Record`** — JS objects with `@Field` defaults (best for options bags).
3. **`Enumerable` enums** — closed sets (encoding, quality, etc.).
4. **`Either`** — one of 2–4 types for overloaded JS args.
5. **Built-in convertibles** — `URL`, `UIColor`/`Color`, `CGPoint`, `Uri`, files — see [REFERENCE.md#built-in-convertibles](REFERENCE.md#built-in-convertibles).
6. **`Convertible` / `ModuleConverters`** — domain types (iOS/Android).
7. **`JavaScriptValue`** — escape hatch; sync + JS thread only.

## Platform lifecycle (summary)

Use lifecycle hooks instead of constructors when work depends on **React or Activity** being ready.

| Concern | iOS | Android (Module DSL) |
|---------|-----|------------------------|
| Module init | `OnCreate` | `OnCreate` |
| Module teardown | `OnDestroy` | `OnDestroy` |
| Foreground | `OnAppEntersForeground` | `OnActivityEntersForeground` |
| Background | `OnAppEntersBackground` | `OnActivityEntersBackground` |
| Activity result | — | `OnActivityResult` or **`RegisterActivityContracts`** |
| Deep link (module instance) | — | `OnNewIntent` in `definition()` |
| View destroyed | destructor | `OnViewDestroys` |

Full Module DSL list: [REFERENCE.md#lifecycle](REFERENCE.md#lifecycle).

## Android Package lifecycle listeners (when building modules)

Some Android hooks **cannot** be done only inside `ModuleDefinition` — especially early `Activity.onCreate`, custom `onBackPressed`, or `Application.onCreate` **without** asking users to paste code into `MainActivity.java` / `MainApplication.java`.

For those cases, implement Expo’s **`Package`** + listener pattern ([official guide](https://docs.expo.dev/modules/android-lifecycle-listeners/)):

| Need | Mechanism |
|------|-----------|
| `Activity.onCreate`, `onResume`, `onPause`, `onDestroy`, `onNewIntent`, `onBackPressed` | `Package.createReactActivityLifecycleListeners` → `ReactActivityLifecycleListener` |
| `Application.onCreate`, `onConfigurationChanged` | `Package.createApplicationLifecycleListeners` → `ApplicationLifecycleListener` |
| Send captured system data to JS | Singleton listener → **observer set** on module companion → `sendEvent` with `OnStartObserving` / `OnStopObserving` |
| Avoid leaks | **`WeakReference`** to module when sending events from observers |

**Choose Module DSL vs Package listener:**

| Situation | Prefer |
|-----------|--------|
| Logic runs while module exists; tied to `OnStartObserving` | `OnNewIntent`, `OnActivityResult`, etc. in `ModuleDefinition` |
| Must run at first Activity create before module loads | `ReactActivityLifecycleListener.onCreate` |
| Custom back button handling | `ReactActivityLifecycleListener.onBackPressed` |
| App-wide setup at cold start | `ApplicationLifecycleListener.onCreate` |
| Deep links / intents (production pattern) | Package listener + observer bridge (like `expo-linking`) |

**Not supported** on `ReactActivityLifecycleListener`: `onStart`, `onStop` (hooks come from `ReactActivityDelegate`, not `MainActivity`).

**Maintenance:** listener interfaces may change between SDKs — override **only** methods you need; do not implement every method.

Detail: [REFERENCE.md#android-lifecycle-listeners](REFERENCE.md#android-lifecycle-listeners). Full deep-link bridge: [EXAMPLES.md#android-lifecycle-to-js](EXAMPLES.md#android-lifecycle-to-js).

## Common mistakes when building modules

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `Cannot find native module` | Wrong `Name`, not linked, no rebuild | Match `Name`, check autolinking, rebuild |
| UI freezes on call | Heavy `Function` on JS thread | `AsyncFunction` |
| Crash from native | `JavaScriptObject` off JS thread | `Function` only, same thread |
| Events never fire | Missing `Events(...)` declaration | Add name before `sendEvent` |
| View callback undefined | No `EventDispatcher` on view | Match `Events` name to property |
| Android view won't mount | Doesn't extend `ExpoView` | Subclass `ExpoView` |
| Kotlin Promise fails | Wrong import | `expo.modules.kotlin.Promise` |
| Stale constant | Used `Property` for fixed value | Use `Constant` |

## Review checklist (before shipping a module)

- [ ] `Name` matches JS `requireNativeModule` / view manager
- [ ] No blocking `Function` for I/O, network, or disk
- [ ] `AsyncFunction` for promises; coroutines on Android where appropriate
- [ ] Records/enums for non-trivial JS objects
- [ ] `Events` + start/stop observing if native listeners cost resources
- [ ] View: `ExpoView`, `EventDispatcher` for callbacks, ref methods on main queue
- [ ] TypeScript types for public API and events
- [ ] Tested on **both** iOS and Android (lifecycle differs)
- [ ] No deprecated `Constants` (use `Constant`)
- [ ] Android: Package listeners registered if needed; `expo-module.config.json` lists module class
- [ ] Lifecycle → JS bridge uses observers + `WeakReference`; listeners removed in `OnStopObserving`

## Operating modes

### Authoring mode
1. Confirm module shape (headless / view / hybrid).
2. Implement `definition()` on both platforms with matching `Name` and method names.
3. Add TS wrapper in `src/index.ts`.
4. Rebuild native and run smoke tests from JS.

### Debug mode
1. Verify `Name` and autolinking.
2. Classify call as sync vs async and check thread.
3. For events, trace `Events` → observer setup → `sendEvent` / `EventDispatcher`.
4. Compare with [EXAMPLES.md](EXAMPLES.md) and reference modules (`expo-clipboard`, `expo-image-picker`).

## Additional resources

- [REFERENCE.md](REFERENCE.md) — complete DSL, argument types, guides
- [EXAMPLES.md](EXAMPLES.md) — end-to-end patterns for module authors
- [REFERENCE.md#android-lifecycle-listeners](REFERENCE.md#android-lifecycle-listeners) — Activity/Application hooks without MainActivity edits
- Reference implementations: `expo-linking`, `expo-clipboard`, `expo-crypto`, `expo-image-picker`, `expo-linear-gradient`, `expo-web-browser` in [expo/expo packages](https://github.com/expo/expo/tree/main/packages)
