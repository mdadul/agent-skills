# Expo Modules API — Reference

Complete API reference for **building native modules**. The Modules API is a JSI abstraction with a Swift/Kotlin DSL. Official: [Module API](https://docs.expo.dev/modules/module-api/).

---

## Definition components {#definition-components}

Every module class implements `definition()` returning `ModuleDefinition { }`.

### `Name` {#name}

Sets the JavaScript module name.

```swift
Name("MyModuleName")
```

```kotlin
Name("MyModuleName")
```

Can be inferred from class name; **set explicitly** for clarity and stable JS imports.

---

### `Constant` {#constant}

Single property computed **once** on first access, then cached.

```swift
Constant("PI") { Double.pi }
```

```kotlin
Constant("PI") { Math.PI }
```

---

### `Constants` {#constants}

**Deprecated** — use [`Constant`](#constant) per key.

```swift
Constants(["PI": Double.pi])
Constants { ["PI": Double.pi] }
```

```kotlin
Constants("PI" to kotlin.math.PI)
Constants { mapOf("PI" to kotlin.math.PI) }
```

---

### `Function` {#function}

**Synchronous** native function on the **JavaScript thread**. Blocks JS until return. Up to **8 arguments** (Swift/Kotlin generics limit).

```swift
Function("mySyncFunction") { (message: String) in
  return message
}
```

```kotlin
Function("mySyncFunction") { message: String ->
  return@Function message
}
```

```js
import { requireNativeModule } from 'expo-modules-core';
const MyModule = requireNativeModule('MyModule');
MyModule.mySyncFunction('bar');
```

Use only for **fast** work. For I/O, network, filesystem, or long CPU → [`AsyncFunction`](#asyncfunction).

---

### `AsyncFunction` {#asyncfunction}

Always returns a **Promise**. Native body runs **off the JS thread** by default.

**Resolve behavior:**
- Return value → resolved Promise.
- Throw → rejected Promise.
- Last parameter `Promise` → wait for manual resolve/reject.

```swift
AsyncFunction("myAsyncFunction") { (message: String) in
  return message
}

AsyncFunction("myAsyncFunction") { (message: String, promise: Promise) in
  promise.resolve(message)
}
```

```kotlin
AsyncFunction("myAsyncFunction") { message: String ->
  return@AsyncFunction message
}

// Promise from expo.modules.kotlin (NOT expo.modules.core)
AsyncFunction("myAsyncFunction") { message: String, promise: Promise ->
  promise.resolve(message)
}
```

```js
await MyModule.myAsyncFunction('bar');
```

**Queue override:**

```swift
AsyncFunction("work") { ... }.runOnQueue(.main)
```

```kotlin
AsyncFunction("work") { ... }.runOnQueue(Queues.MAIN)
```

**Kotlin coroutines** — suspend body; no `Promise` argument; canceled when module deallocates:

```kotlin
AsyncFunction("suspendFunction") Coroutine { message: String ->
  delay(5000)
  return@Coroutine message
}
```

---

### `Property` {#property}

Like `Object.defineProperty` on the module object.

**Read-only:**

```swift
Property("foo") { return "bar" }
```

```kotlin
Property("foo") { return@Property "bar" }
```

**Read/write:**

```swift
Property("foo")
  .get { return "bar" }
  .set { (newValue: String) in /* store */ }
```

```kotlin
Property("foo")
  .get { return@get "bar" }
  .set { newValue: String -> /* store */ }
```

```js
MyModule.foo;
MyModule.foo = 'foobar';
```

---

### `View` {#view}

Exports a native view to React Native. Allowed children: [`Prop`](#prop), [`Events`](#events), [`GroupView`](#groupview), [`AsyncFunction`](#view-asyncfunction).

```swift
View(UITextView.self) {
  Prop("text") { (view: UITextView, text: String) in view.text = text }
  AsyncFunction("focus") { (view: UITextView) in view.becomeFirstResponder() }
}
```

```kotlin
View(TextView::class) {
  Prop("text") { view: TextView, text: String -> view.text = text }
  AsyncFunction("focus") { view: TextView -> view.requestFocus() }
}
```

- **Android:** view class must extend [`ExpoView`](#expoview).
- **iOS:** extending `ExpoView` recommended; SwiftUI via `UIHostingController` until native SwiftUI support lands.

View [`AsyncFunction`](#view-asyncfunction): attached to **ref**, view as **first arg**, **main queue** default.

---

### `Events` {#events}

Declares event names the module or view may emit.

```swift
Events("onCameraReady", "onPictureSaved")
```

```kotlin
Events("onCameraReady", "onPictureSaved")
```

Use inside [`View`](#view) for view callback props. See [Sending events](#sending-events) and [View callbacks](#view-callbacks).

---

### `OnStartObserving` / `OnStopObserving` {#event-observing}

Scoped by event name — run when first listener added / all listeners removed.

```swift
OnStartObserving("onURLReceived") { /* start */ }
OnStopObserving("onURLReceived") { /* stop */ }
```

```kotlin
OnStartObserving("onURLReceived") { /* start */ }
OnStopObserving("onURLReceived") { /* stop */ }
```

Use to start/stop expensive native observers only while JS listens.

---

### Lifecycle listeners {#lifecycle}

| Listener | Platform | When |
|----------|----------|------|
| `OnCreate` | Both | After module init (prefer over constructor) |
| `OnDestroy` | Both | Before module dealloc |
| `OnAppContextDestroys` | Both | App context dealloc |
| `OnAppEntersForeground` | iOS | App entering foreground |
| `OnAppEntersBackground` | iOS | App entering background |
| `OnAppBecomesActive` | iOS | App active after foreground |
| `OnActivityEntersForeground` | Android | Activity resumed |
| `OnActivityEntersBackground` | Android | Activity paused |
| `OnActivityDestroys` | Android | Activity destroyed |
| `OnActivityResult` | Android | `startActivityForResult` result |
| `OnNewIntent` | Android | New intent (e.g. deep link) |
| `OnUserLeavesActivity` | Android | User sent app to background (Home) |
| `RegisterActivityContracts` | Android | Type-safe activity results |

**Android activity result (legacy):**

```kotlin
AsyncFunction("pick") { /* startActivityForResult */ }

OnActivityResult { activity, payload ->
  val requestCode = payload.requestCode
  val resultCode = payload.resultCode
  val data = payload.data  // Intent?
}
```

**Android activity result (modern):** see [RegisterActivityContracts](#registeractivitycontracts) in [EXAMPLES.md](EXAMPLES.md).

**Android deep link (Module DSL only):**

```kotlin
OnNewIntent { intent ->
  val data = intent.data
}
```

For deep links and early Activity hooks **without editing MainActivity**, use [Android Package lifecycle listeners](#android-lifecycle-listeners) instead.

---

## View definition components {#view-definition}

Only inside `View { }`.

### `Name` (view) {#view-name}

```swift
Name("MyViewName")
```

### `Prop` {#prop}

Setter when React re-renders. Optional default when JS passes `null`.

```swift
Prop("background") { (view: UIView, color: UIColor) in
  view.backgroundColor = color
}
Prop("background", UIColor.black) { (view, color) in ... }
```

```kotlin
Prop("background") { view: View, @ColorInt color: Int ->
  view.setBackgroundColor(color)
}
Prop("background", Color.BLACK) { view, color -> ... }
```

**Not supported:** function-typed props (callbacks) — use [`Events`](#events) + [`EventDispatcher`](#view-callbacks).

### `PropGroup` {#propgroup}

**Android only.** Batch props with shared setter (pair or index). Used by CSS internals; rarely needed in app modules.

### `OnViewDidUpdateProps` {#onviewdidupdateprops}

Called after all props applied.

```swift
OnViewDidUpdateProps { (view: MyView) in }
```

### `OnViewDestroys` {#onviewdestroys}

**Android only** — view no longer used by RN.

### `AsyncFunction` (in View) {#view-asyncfunction}

```swift
View(MyView.self) {
  AsyncFunction("displayMessage") { (view: MyView, message: String) in
    view.displayMessage(message)
  }
}
```

```js
ref.current?.displayMessage('hi');
```

### `GroupView` {#groupview}

**Android only** — `ViewGroup` child management: `AddChildView`, `GetChildCount`, `GetChildViewAt`, `RemoveChildView`, `RemoveChildViewAt`.

---

## Argument types {#argument-types}

### Primitives {#primitives}

| Swift | Kotlin |
|-------|--------|
| `Bool`, `Int`/`Int8`–`Int64`, `UInt` variants, `Float32`, `Double`, `String` | `Boolean`, `Int`, `Long`, `Float`, `Double`, `String`, `Pair` |
| Arrays, dictionaries, optionals | Arrays, maps, optionals |

### Convertibles {#convertibles}

Native types built from JS shapes (e.g. `CGPoint` from `{x,y}` or `[x,y]`).

**iOS — `Convertible` protocol:**

```swift
extension CMTime: @retroactive Convertible {
  public static func convert(from value: Any?, appContext: AppContext) throws -> CMTime {
    if let seconds = value as? Double {
      return CMTime(seconds: seconds, preferredTimescale: .max)
    }
    throw Conversions.ConvertingException<CMTime>(value)
  }
}
```

**Android — `ModuleConverters`:**

```kotlin
override fun converters() = ModuleConverters {
  TypeConverter(CustomType::class)
    .from { n: Int -> CustomType.fromInt(n) }
    .from { s: String -> CustomType.parse(s) }
}
```

Runtime tries each `.from` until one matches.

### Built-in convertibles {#built-in-convertibles}

| Native (iOS) | TypeScript |
|--------------|------------|
| `URL` | `string` (no scheme → file URL) |
| `CGFloat` | `number` |
| `CGPoint` | `{x,y}` or `[x,y]` |
| `CGSize` | `{width,height}` or `[w,h]` |
| `CGVector` | `{dx,dy}` or `[dx,dy]` |
| `CGRect` | object or `[x,y,w,h]` |
| `CGColor` / `UIColor` | `#RRGGBB`, `#RRGGBBAA`, `#RGB`, `#RGBA`, CSS named colors, `"transparent"` |
| `Data` | `Uint8Array` (SDK 50+) |

| Native (Android) | TypeScript |
|------------------|------------|
| `java.net.URL` | `string` (scheme required) |
| `Uri` / `URI` | `string` (scheme required) |
| `File` / `Path` | file path `string` |
| `Color` | same hex/named as iOS |
| `Pair<A,B>` | `[a, b]` |
| `ByteArray` | `Uint8Array` (SDK 50+) |
| `BooleanArray` | `boolean[]` |
| `IntArray`, etc. | `number[]` |
| `Duration` | seconds as `number` (SDK 52+) |

### Records {#records}

Typed JS objects with validation and defaults.

```swift
struct FileReadOptions: Record {
  @Field var encoding: String = "utf8"
  @Field var position: Int = 0
  @Field var length: Int?
}
```

```kotlin
class FileReadOptions : Record {
  @Field val encoding: String = "utf8"
  @Field val position: Int = 0
  @Field val length: Int? = null
}
```

### Formatter {#formatter}

**Experimental** — transform Record before sending to JS: `.map`, `.skip`, conditional `.skip { }`.

```swift
return user.format { f in
  f.property("password", keyPath: \.password).skip()
}
```

### Enums {#enums}

Must conform to **`Enumerable`** with primitive raw value.

```swift
enum FileEncoding: String, Enumerable { case utf8, base64 }
```

```kotlin
enum class FileEncoding(val value: String) : Enumerable {
  utf8("utf8"), base64("base64")  // constructor param MUST be `value`
}
```

### Either {#either}

```swift
Function("foo") { (bar: Either<String, Int>) in
  if let s: String = bar.get() { }
  if let i: Int = bar.get() { }
}
```

```kotlin
Function("foo") { bar: Either<String, Int> ->
  bar.get(String::class)?.let { }
  bar.get(Int::class)?.let { }
}
```

Types: `Either<A,B>`, `EitherOfThree`, `EitherOfFour`.

### ValueOrUndefined {#valueorundefined}

**Experimental** — preserves JS `undefined` vs `null` vs value (`isUndefined`, `optional`).

### JavaScript values {#javascript-values}

`JavaScriptValue`, `JavaScriptObject`, `JavaScriptFunction<ReturnType>` — **only in sync `Function`**, **JS thread only** (otherwise crash).

```swift
Function("mutateMe") { (jsObject: JavaScriptObject) in
  jsObject.setProperty("expo", value: "modules")
}
```

---

## Native classes {#native-classes}

### `Module` {#module-class}

```swift
class MyModule: Module {
  func definition() -> ModuleDefinition { ... }
  // sendEvent("onX", ["key": value])
  // appContext: AppContext
}
```

`sendEvent(eventName, payload)` — `Map`/`Bundle` (Android), `[String: Any?]` (iOS).

### `AppContext` {#appcontext}

| Property | Description |
|----------|-------------|
| `constants` | Legacy constants interface |
| `permissions` | Permissions manager |
| `activityProvider` | Activity provider |
| `reactContext` | React application context (Android) |
| `hasActiveReactInstance` | RN instance alive |
| `utilities` | Legacy utilities |

### `ExpoView` {#expoview}

Base for exported views; provides `appContext`.

```swift
class LinearGradientView: ExpoView {}
```

```kotlin
class LinearGradientView(context: Context, appContext: AppContext) :
  ExpoView(context, appContext)
```

Do not change constructor signature.

---

## Guides {#guides}

### Sending events (module-level) {#sending-events}

1. `Events("onClipboardChanged")`
2. `OnStartObserving` / `OnStopObserving` to register native observers
3. `sendEvent("onClipboardChanged", payload)` from module instance
4. JS: `requireNativeModule('Clipboard').addListener(...)` or `useEvent` / `useEventListener`

Modules extend `EventEmitter`. Type with `NativeModule<Events>`.

Full clipboard-style example: [EXAMPLES.md#module-events-full](EXAMPLES.md#module-events-full).

### View callbacks {#view-callbacks}

1. `Events("onCameraReady")` inside `View { }`
2. View class: property `onCameraReady` = `EventDispatcher()` (**same name**)
3. `onCameraReady(["message": "ok"])` from native
4. JS: `<CameraView onCameraReady={(e) => e.nativeEvent} />`

Do **not** use `sendEvent` for view-bound events.

Android: non-object payloads may appear as `{ payload: value }`.

---

## Android Package lifecycle listeners {#android-lifecycle-listeners}

Hook into Android **Activity** and **Application** without copying code into `MainActivity` / `MainApplication`. Official: [Android lifecycle listeners](https://docs.expo.dev/modules/android-lifecycle-listeners/).`

**Prerequisite:** Expo module or library with Expo Modules API integrated ([overview](https://docs.expo.dev/modules/overview)).

### Overview

1. Create a class implementing [`Package`](https://github.com/expo/expo/blob/main/packages/expo-modules-core/android/src/main/java/expo/modules/core/interfaces/Package.java).
2. Return listeners from `createReactActivityLifecycleListeners` and/or `createApplicationLifecycleListeners`.
3. Optionally bridge to JS via module `Events` + observer pattern (listeners are **singletons**, separate from module instances).

### `ReactActivityLifecycleListener` (Activity) {#react-activity-lifecycle-listener}

Hooks React Native’s `ReactActivity` via `ReactActivityDelegate` (similar to Activity lifecycle, not identical to `MainActivity`).

**Supported callbacks:**

| Callback | Use |
|----------|-----|
| `onCreate(activity, savedInstanceState)` | First create; read `intent` (cold-start deep link) |
| `onResume(activity)` | Foreground |
| `onPause(activity)` | Background |
| `onDestroy(activity)` | Cleanup |
| `onNewIntent(intent)` | New intent while running; return `true` if handled |
| `onBackPressed()` | Return `true` to consume back press |

**Not supported:** `onStart`, `onStop` — `ReactActivityDelegate` does not expose them.

**Register via Package:**

```kotlin
// MyLibPackage.kt
package expo.modules.mylib

import android.content.Context
import expo.modules.core.interfaces.Package
import expo.modules.core.interfaces.ReactActivityLifecycleListener

class MyLibPackage : Package {
  override fun createReactActivityLifecycleListeners(
    activityContext: Context
  ): List<ReactActivityLifecycleListener> {
    return listOf(MyLibReactActivityLifecycleListener())
  }
}
```

**Listener — override only what you need:**

```kotlin
class MyLibReactActivityLifecycleListener : ReactActivityLifecycleListener {
  override fun onCreate(activity: Activity?, savedInstanceState: Bundle?) {
    activity?.intent?.data?.let { handleDeepLink(it.toString()) }
  }

  override fun onResume(activity: Activity) {
    trackAppStateChange("active")
  }

  override fun onPause(activity: Activity) {
    trackAppStateChange("inactive")
  }

  override fun onDestroy(activity: Activity) {
    cleanup()
  }

  override fun onNewIntent(intent: Intent?): Boolean {
    intent?.data?.let {
      handleDeepLink(it.toString())
      return true
    }
    return false
  }

  override fun onBackPressed(): Boolean {
    return handleCustomBackNavigation()
  }
}
```

### `ApplicationLifecycleListener` (Application) {#application-lifecycle-listener}

**Supported callbacks:**

| Callback | Use |
|----------|-----|
| `onCreate(application)` | App-wide init |
| `onConfigurationChanged` | Locale, UI mode, etc. |

```kotlin
class MyLibPackage : Package {
  override fun createApplicationLifecycleListeners(
    context: Context
  ): List<ApplicationLifecycleListener> {
    return listOf(MyLibApplicationLifecycleListener())
  }
}

class MyLibApplicationLifecycleListener : ApplicationLifecycleListener {
  override fun onCreate(application: Application) {
    doSomeSetupInApplicationOnCreate(application)
  }
}
```

Override **only** required methods — interfaces may change between Expo SDK releases (new methods added; old ones `@Deprecated`).

### Lifecycle listener → JavaScript event flow {#lifecycle-to-js}

Listeners are **singletons**; modules are per-instance. Typical bridge (pattern from `expo-linking`):

```text
Android intent / lifecycle
  → ReactActivityLifecycleListener (singleton)
  → notify module companion observers (Uri → Unit)
  → Module.sendEvent (WeakReference to module)
  → JS addListener / hook
```

Steps:

1. **System integration** — listener captures intents / lifecycle in `onCreate` / `onNewIntent`.
2. **Observer pattern** — `companion object` holds `initialUrl` + `MutableSet<(Uri) -> Unit>` observers.
3. **Event bridging** — module `OnStartObserving` registers observer that calls `sendEvent`.
4. **Memory** — `WeakReference<Module>` inside observer lambda.
5. **TypeScript** — `NativeModule<Events>` + optional React hook.

Full code: [EXAMPLES.md#android-lifecycle-to-js](EXAMPLES.md#android-lifecycle-to-js).

### Module DSL vs Package listeners {#lifecycle-dsl-vs-package}

| Use Module DSL (`OnNewIntent`, etc.) | Use Package listeners |
|--------------------------------------|------------------------|
| Handler tied to module lifecycle | Early `onCreate` before module ready |
| Simpler one-file module | Custom `onBackPressed` |
| | `Application.onCreate` |
| | Production deep-link capture (`expo-linking` style) |

### `expo-module.config.json` {#expo-module-config-android}

Register the Kotlin module class:

```json
{
  "platforms": ["android"],
  "android": {
    "modules": ["expo.modules.deeplinkhandler.DeepLinkHandlerModule"]
  }
}
```

### Known issues {#android-lifecycle-known-issues}

| Issue | Detail |
|-------|--------|
| No `onStart` / `onStop` | Hooks attach to `ReactActivityDelegate`, not `MainActivity` |
| Interface stability | May change per SDK; implement only needed methods |

---

## Limits and pitfalls {#limits}

| Rule | Detail |
|------|--------|
| Max 8 args | Per `Function` / `AsyncFunction` |
| `Function` blocks JS | Never for I/O or long work |
| `Constants` | Deprecated |
| Kotlin `Promise` | `expo.modules.kotlin.Promise` |
| View prop callbacks | Not supported — use events |
| `JavaScriptObject` | Sync + JS thread only |
| SwiftUI views | `UIHostingController` workaround |
