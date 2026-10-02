# Expo Modules API — Examples

Patterns for **building Expo native modules** from scratch. Pair with [SKILL.md](SKILL.md) workflows and [REFERENCE.md](REFERENCE.md).

---

## Package layout (typical module)

```text
expo-my-feature/
├── expo-module.config.json
├── package.json
├── src/
│   ├── index.ts              # public API
│   └── MyFeature.types.ts
├── ios/
│   └── MyFeatureModule.swift
└── android/src/main/java/expo/modules/myfeature/
    └── MyFeatureModule.kt
```

**`src/index.ts`** — always wrap native:

```ts
import { requireNativeModule } from 'expo-modules-core';

export default requireNativeModule('MyFeature');
```

---

## Minimal module {#minimal-module}

**Swift**

```swift
import ExpoModulesCore

public class MyModule: Module {
  public func definition() -> ModuleDefinition {
    Name("MyFirstExpoModule")

    Function("hello") { (name: String) in
      return "Hello \(name)!"
    }
  }
}
```

**Kotlin**

```kotlin
import expo.modules.kotlin.Module
import expo.modules.kotlin.modules.ModuleDefinition

class MyModule : Module() {
  override fun definition() = ModuleDefinition {
    Name("MyFirstExpoModule")

    Function("hello") { name: String ->
      return@Function "Hello $name!"
    }
  }
}
```

**TypeScript**

```ts
import { requireNativeModule } from 'expo-modules-core';

const MyModule = requireNativeModule<{ hello(name: string): string }>('MyFirstExpoModule');

export function greet(name: string): string {
  return MyModule.hello(name);
}
```

---

## AsyncFunction, Promise, and queues

**Swift — return value**

```swift
AsyncFunction("fetchData") { (url: String) async throws -> String in
  return try await load(url)
}
```

**Swift — explicit Promise**

```swift
AsyncFunction("fetchData") { (url: String, promise: Promise) in
  Task {
    do {
      promise.resolve(try await load(url))
    } catch {
      promise.reject(error)
    }
  }
}
```

**Swift — main queue**

```swift
AsyncFunction("updateUI") { () in
  // UI work
}.runOnQueue(.main)
```

**Kotlin — return value**

```kotlin
AsyncFunction("fetchData") { url: String ->
  return@AsyncFunction httpGet(url)
}
```

**Kotlin — coroutine**

```kotlin
import kotlinx.coroutines.delay

AsyncFunction("delay") Coroutine { message: String ->
  delay(1000)
  message
}
```

**Kotlin — Promise** (`expo.modules.kotlin.Promise`)

```kotlin
AsyncFunction("fetchData") { url: String, promise: Promise ->
  thread {
    try {
      promise.resolve(httpGet(url))
    } catch (e: Exception) {
      promise.reject(e)
    }
  }
}
```

**JavaScript**

```ts
const result = await MyModule.fetchData('https://example.com');
```

---

## Constant, Property, and lifecycle

```swift
public class SettingsModule: Module {
  private var enabled = false

  public func definition() -> ModuleDefinition {
    Name("Settings")

    Constant("maxItems") { 100 }

    Property("enabled")
      .get { self.enabled }
      .set { self.enabled = $0 }

    OnCreate {
      // setup when module loads — prefer over init
    }

    OnDestroy {
      // teardown
    }
  }
}
```

---

## Record and Enumerable (options object)

**Swift**

```swift
enum FileEncoding: String, Enumerable {
  case utf8, base64
}

struct FileReadOptions: Record {
  @Field var encoding: FileEncoding = .utf8
  @Field var position: Int = 0
  @Field var length: Int?
}

Function("readFile") { (path: String, options: FileReadOptions) -> String in
  read(path, options)
}
```

**Kotlin**

```kotlin
enum class FileEncoding(val value: String) : Enumerable {
  utf8("utf8"),
  base64("base64")
}

class FileReadOptions : Record {
  @Field val encoding: FileEncoding = FileEncoding.utf8
  @Field val position: Int = 0
  @Field val length: Int? = null
}

Function("readFile") { path: String, options: FileReadOptions ->
  read(path, options)
}
```

**TypeScript**

```ts
type FileReadOptions = {
  encoding?: 'utf8' | 'base64';
  position?: number;
  length?: number;
};

export function readFile(path: string, options?: FileReadOptions): string {
  return MyModule.readFile(path, options ?? {});
}
```

---

## Either (overloaded argument)

**Swift**

```swift
Function("setSize") { (size: Either<Int, [String: Double]>) in
  if let n: Int = size.get() {
    applyUniform(n)
  } else if let map = size.get() as [String: Double]? {
    applyMap(map)
  }
}
```

**Kotlin**

```kotlin
Function("setSize") { size: Either<Int, Map<String, Double>> ->
  size.get(Int::class)?.let { applyUniform(it) }
  size.get(Map::class.java)?.let { applyMap(it) }
}
```

---

## ValueOrUndefined (experimental)

**Swift**

```swift
Function("configure") { (timeout: ValueOrUndefined<Int>) in
  if timeout.isUndefined {
    useDefaultTimeout()
  } else if let value = timeout.optional {
    setTimeout(value)
  }
}
```

**JS**

```ts
MyModule.configure(undefined); // not provided
MyModule.configure(null);      // explicit null (with ValueOrUndefined<String?>)
MyModule.configure(5000);
```

---

## Module events (full) {#module-events-full}

**Swift**

```swift
let CLIPBOARD_CHANGED = "onClipboardChanged"

public class ClipboardModule: Module {
  public func definition() -> ModuleDefinition {
    Name("Clipboard")
    Events(CLIPBOARD_CHANGED)

    OnStartObserving(CLIPBOARD_CHANGED) {
      NotificationCenter.default.addObserver(
        self,
        selector: #selector(self.onClipboardChanged),
        name: UIPasteboard.changedNotification,
        object: nil
      )
    }

    OnStopObserving(CLIPBOARD_CHANGED) {
      NotificationCenter.default.removeObserver(
        self,
        name: UIPasteboard.changedNotification,
        object: nil
      )
    }
  }

  @objc private func onClipboardChanged() {
    sendEvent(CLIPBOARD_CHANGED, ["contentTypes": availableTypes()])
  }
}
```

**Kotlin**

```kotlin
const val CLIPBOARD_CHANGED = "onClipboardChanged"

class ClipboardModule : Module() {
  private val listener = ClipboardManager.OnPrimaryClipChangedListener {
    sendEvent(CLIPBOARD_CHANGED, bundleOf("contentTypes" to types()))
  }

  override fun definition() = ModuleDefinition {
    Name("Clipboard")
    Events(CLIPBOARD_CHANGED)

    OnStartObserving(CLIPBOARD_CHANGED) {
      clipboardManager?.addPrimaryClipChangedListener(listener)
    }

    OnStopObserving(CLIPBOARD_CHANGED) {
      clipboardManager?.removePrimaryClipChangedListener(listener)
    }
  }
}
```

**TypeScript**

```ts
import { requireNativeModule, NativeModule } from 'expo';

type ClipboardEvents = {
  onClipboardChanged(event: { contentTypes: string[] }): void;
};

declare class ClipboardModule extends NativeModule<ClipboardEvents> {}

const Clipboard = requireNativeModule<ClipboardModule>('Clipboard');

export function addClipboardListener(
  listener: (event: { contentTypes: string[] }) => void
) {
  return Clipboard.addListener('onClipboardChanged', listener);
}
```

---

## Native view (props, ref, events)

**Swift module + view**

```swift
public class CameraModule: Module {
  public func definition() -> ModuleDefinition {
    Name("ExpoCamera")
    View(CameraView.self) {
      Events("onCameraReady", "onPictureSaved")
      Prop("active") { (view: CameraView, active: Bool) in
        view.setActive(active)
      }
      AsyncFunction("takePicture") { (view: CameraView, options: PictureOptions) in
        view.takePicture(options)
      }
    }
  }
}

class CameraView: ExpoView {
  let onCameraReady = EventDispatcher()
  let onPictureSaved = EventDispatcher()

  func notifyReady() {
    onCameraReady(["message": "Camera was mounted"])
  }
}
```

**Kotlin**

```kotlin
class CameraModule : Module() {
  override fun definition() = ModuleDefinition {
    Name("ExpoCamera")
    View(CameraView::class) {
      Events("onCameraReady", "onPictureSaved")
      Prop("active") { view: CameraView, active: Boolean ->
        view.setActive(active)
      }
      AsyncFunction("takePicture") { view: CameraView, options: PictureOptions ->
        view.takePicture(options)
      }
    }
  }
}

class CameraView(context: Context, appContext: AppContext) :
  ExpoView(context, appContext) {
  val onCameraReady by EventDispatcher()
  val onPictureSaved by EventDispatcher()
}
```

**React**

```tsx
import { requireNativeViewManager } from 'expo-modules-core';
import { useRef, useEffect } from 'react';

const CameraView = requireNativeViewManager('ExpoCamera');

export function CameraScreen() {
  const ref = useRef<any>(null);

  useEffect(() => {
    ref.current?.takePicture({ quality: 0.8 });
  }, []);

  return (
    <CameraView
      ref={ref}
      active
      onCameraReady={(e) => console.log(e.nativeEvent)}
      onPictureSaved={(e) => console.log(e.nativeEvent)}
    />
  );
}
```

---

## RegisterActivityContracts (Android picker)

```kotlin
class ImagePickerModule : Module() {
  private lateinit var cameraLauncher: ActivityResultLauncher<CameraContractOptions>

  override fun definition() = ModuleDefinition {
    Name("ImagePicker")

    RegisterActivityContracts {
      cameraLauncher = registerForActivityResult(
        CameraContract(this@ImagePickerModule)
      ) { input, result ->
        handleResult(result, input.options)
      }
    }

    AsyncFunction("launchCameraAsync") { options: PickerOptions ->
      cameraLauncher.launch(CameraContractOptions(options))
    }
  }
}
```

Prefer this over raw `OnActivityResult` + `startActivityForResult` for new modules.

---

## Custom Convertible (iOS) and ModuleConverters (Android)

**iOS**

```swift
extension CMTime: @retroactive Convertible {
  public static func convert(from value: Any?, appContext: AppContext) throws -> CMTime {
    guard let seconds = value as? Double else {
      throw Conversions.ConvertingException<CMTime>(value)
    }
    return CMTime(seconds: seconds, preferredTimescale: .max)
  }
}
```

**Android**

```kotlin
class MyModule : Module() {
  override fun converters() = ModuleConverters {
    TypeConverter(CustomType::class)
      .from { n: Int -> CustomType.fromInt(n) }
      .from { s: String -> CustomType.parse(s) }
  }

  override fun definition() = ModuleDefinition {
    Name("MyModule")
    Function("process") { value: CustomType -> value.doSomething() }
  }
}
```

---

## Formatter — hide fields from JS (experimental)

**Swift**

```swift
Function("getUser") {
  let user = UserInfo(id: 1, email: "a@b.com", password: "secret")
  return user.format { f in
    f.property("password", keyPath: \.password).skip()
  }
}
```

**Kotlin**

```kotlin
Function("getUser") {
  val user = UserInfo(1, "a@b.com", "secret")
  formatter {
    property(UserInfo::password).skip()
  }.format(user)
}
```

---

## Hybrid module (API + view in one class)

```swift
public class MediaModule: Module {
  public func definition() -> ModuleDefinition {
    Name("ExpoMedia")

    AsyncFunction("getCacheSize") { () -> Int in
      cacheSize()
    }

    View(VideoView.self) {
      Prop("source") { (view: VideoView, url: URL) in view.load(url) }
      Events("onPlaybackStatus")
    }
  }
}
```

```ts
import { requireNativeModule, requireNativeViewManager } from 'expo-modules-core';

export const Media = requireNativeModule('ExpoMedia');
export const VideoView = requireNativeViewManager('VideoView');

export const getCacheSize = () => Media.getCacheSize();
```

---

## Android lifecycle listeners → JavaScript {#android-lifecycle-to-js}

Bridge Android **Activity** / **Application** events to JS without editing `MainActivity`. Based on [`expo-linking`](https://github.com/expo/expo/tree/main/packages/expo-linking). See [REFERENCE.md#android-lifecycle-listeners](REFERENCE.md#android-lifecycle-listeners).

### 1. Package registration

```kotlin
// DeepLinkHandlerPackage.kt
package expo.modules.deeplinkhandler

import android.content.Context
import expo.modules.core.interfaces.Package
import expo.modules.core.interfaces.ReactActivityLifecycleListener

class DeepLinkHandlerPackage : Package {
  override fun createReactActivityLifecycleListeners(
    activityContext: Context?
  ): List<ReactActivityLifecycleListener> {
    return listOf(DeepLinkHandlerActivityLifecycleListener())
  }
}
```

### 2. Activity lifecycle listener

```kotlin
// DeepLinkHandlerActivityLifecycleListener.kt
package expo.modules.deeplinkhandler

import android.app.Activity
import android.content.Intent
import android.net.Uri
import android.os.Bundle
import expo.modules.core.interfaces.ReactActivityLifecycleListener

class DeepLinkHandlerActivityLifecycleListener : ReactActivityLifecycleListener {
  override fun onCreate(activity: Activity?, savedInstanceState: Bundle?) {
    handleIntent(activity?.intent)
  }

  override fun onNewIntent(intent: Intent?): Boolean {
    handleIntent(intent)
    return true
  }

  private fun handleIntent(intent: Intent?) {
    val url: Uri? = intent?.data ?: return
    DeepLinkHandlerModule.initialUrl = url
    DeepLinkHandlerModule.urlReceivedObservers.forEach { observer ->
      observer(url)
    }
  }
}
```

### 3. Module with observers and events

```kotlin
// DeepLinkHandlerModule.kt
package expo.modules.deeplinkhandler

import android.net.Uri
import androidx.core.os.bundleOf
import expo.modules.kotlin.modules.Module
import expo.modules.kotlin.modules.ModuleDefinition
import java.lang.ref.WeakReference

class DeepLinkHandlerModule : Module() {
  companion object {
    var initialUrl: Uri? = null
    var urlReceivedObservers: MutableSet<((Uri) -> Unit)> = mutableSetOf()
  }

  private var urlReceivedObserver: ((Uri) -> Unit)? = null

  override fun definition() = ModuleDefinition {
    Name("DeepLinkHandler")

    Events("onUrlReceived")

    Function("getInitialUrl") {
      initialUrl?.toString()
    }

    OnStartObserving("onUrlReceived") {
      val weakModule = WeakReference(this@DeepLinkHandlerModule)
      val observer: (Uri) -> Unit = { uri ->
        weakModule.get()?.sendEvent(
          "onUrlReceived",
          bundleOf(
            "url" to uri.toString(),
            "scheme" to uri.scheme,
            "host" to uri.host,
            "path" to uri.path
          )
        )
      }
      urlReceivedObservers.add(observer)
      urlReceivedObserver = observer
    }

    OnStopObserving("onUrlReceived") {
      urlReceivedObservers.remove(urlReceivedObserver)
    }
  }
}
```

### 4. TypeScript module

```ts
// DeepLinkHandler.ts
import { requireNativeModule, NativeModule } from 'expo-modules-core';

export type DeepLinkEvent = {
  url: string;
  scheme?: string;
  host?: string;
  path?: string;
};

type DeepLinkHandlerModuleEvents = {
  onUrlReceived(event: DeepLinkEvent): void;
};

declare class DeepLinkHandlerNativeModule extends NativeModule<DeepLinkHandlerModuleEvents> {
  getInitialUrl(): string | null;
}

const DeepLinkHandler =
  requireNativeModule<DeepLinkHandlerNativeModule>('DeepLinkHandler');

export default DeepLinkHandler;
```

### 5. React hook

```tsx
// useDeepLinkHandler.ts
import { useEffect, useState } from 'react';
import DeepLinkHandler, { DeepLinkEvent } from './DeepLinkHandler';

export function useDeepLinkHandler() {
  const [initialUrl] = useState<string | null>(DeepLinkHandler.getInitialUrl());
  const [event, setEvent] = useState<DeepLinkEvent | null>(null);

  useEffect(() => {
    const subscription = DeepLinkHandler.addListener('onUrlReceived', setEvent);
    return () => subscription.remove();
  }, []);

  return {
    initialUrl,
    url: event?.url ?? initialUrl,
    event,
  };
}
```

### 6. expo-module.config.json

```json
{
  "platforms": ["android"],
  "android": {
    "modules": ["expo.modules.deeplinkhandler.DeepLinkHandlerModule"]
  }
}
```

### Application lifecycle listener (optional)

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
    // App-wide setup at Application.onCreate
  }
}
```

---

## Reference modules on GitHub

Study real modules when building yours:

| Package | Notes |
|---------|--------|
| `expo-linking` | Android Package lifecycle + deep link bridge |
| `expo-clipboard` | Events + observers |
| `expo-image-picker` | Activity contracts, async |
| `expo-linear-gradient` | Native view + props |
| `expo-crypto` | Headless sync/async API |
| `expo-web-browser` | Activity / browser flows |
| `expo-localization` | Constants + functions |

https://github.com/expo/expo/tree/main/packages
