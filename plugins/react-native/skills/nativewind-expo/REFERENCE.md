# NativeWind v4 — Optional Setup Reference

## Table of Contents
1. [TypeScript Setup](#typescript-setup)
2. [Dark Mode](#dark-mode)
3. [useColorScheme() API](#usecolorscheme-api)
4. [Custom Fonts](#custom-fonts)
5. [NX Monorepo](#nx-monorepo)

---

## TypeScript Setup

Add the NativeWind type reference so TypeScript recognises `className` on React Native components.

### Option A — nativewind.d.ts (recommended)

Create `nativewind.d.ts` in the project root:

```ts title="nativewind.d.ts"
/// <reference types="nativewind/types" />
```

### Option B — tsconfig.json include

```json title="tsconfig.json"
{
  "compilerOptions": { ... },
  "include": ["nativewind-env.d.ts", "**/*.ts", "**/*.tsx"]
}
```

where `nativewind-env.d.ts` contains the triple-slash directive above.

---

## Dark Mode

### System preference (automatic)

No extra config needed. Use `dark:` variants directly:

```tsx
<Text className="text-black dark:text-white">Hello</Text>
```

Expo requires `userInterfaceStyle: "automatic"` in `app.json`:

```json
{
  "expo": {
    "userInterfaceStyle": "automatic"
  }
}
```

### Read current scheme

```tsx
import { useColorScheme } from "nativewind";

function MyComponent() {
  const { colorScheme } = useColorScheme();
  return <Text>{colorScheme === "dark" ? "Dark" : "Light"}</Text>;
}
```

### Manual toggle

```tsx
import { colorScheme } from "nativewind";

function toggleTheme(current: "light" | "dark") {
  colorScheme.set(current === "light" ? "dark" : "light");
}
```

Always offer a "System" option alongside manual Light / Dark choices so users can revert to device preference.

---

## useColorScheme() API

`useColorScheme()` is imported from `nativewind` and provides unified access to the device color scheme.

| Value | Description |
|-------|-------------|
| `colorScheme` | The current active color scheme (`"light"` \| `"dark"` \| `null`) |
| `setColorScheme` | Override the scheme — accepts `"light"`, `"dark"`, or `"system"` |
| `toggleColorScheme` | Toggle between `"light"` and `"dark"` |

> **Important:** `setColorScheme` and `toggleColorScheme` require `darkMode: "class"` in `tailwind.config.js`. They throw if `darkMode` is `"media"` (the Tailwind default).

### tailwind.config.js prerequisite

```js title="tailwind.config.js"
module.exports = {
  darkMode: "class",  // required for setColorScheme / toggleColorScheme
  // ...
};
```

### Usage example

```tsx
import { useColorScheme } from "nativewind";
import { Text } from "react-native";

function ThemeToggle() {
  const { colorScheme, setColorScheme } = useColorScheme();

  return (
    <Text
      onPress={() => setColorScheme(colorScheme === "light" ? "dark" : "light")}
    >
      {`Current scheme: ${colorScheme}`}
    </Text>
  );
}
```

### `setColorScheme` vs `colorScheme.set()`

Both are exported from `nativewind` and do the same thing. `useColorScheme()` is the hook form (use inside components); `colorScheme.set()` is the imperative form (use outside React, e.g. in event handlers or storage callbacks).

---

## Custom Fonts

### 1. Install expo-font

```bash
# npm
npm install expo-font

# bun
bun add expo-font
```

### 2. Add font files

Place OTF or TTF files in `assets/fonts/`. Use static weight files — variable fonts are not supported by React Native.

File names must match the font's PostScript name (visible in Font Book on macOS or fontdrop.info).

### 3. Register via app.json (preferred)

```json title="app.json"
{
  "expo": {
    "plugins": [
      [
        "expo-font",
        {
          "fonts": [
            "./assets/fonts/Inter-Regular.otf",
            "./assets/fonts/Inter-Bold.otf",
            "./assets/fonts/Inter-Medium.otf"
          ]
        }
      ]
    ]
  }
}
```

### 4. Map in tailwind.config.js

```js title="tailwind.config.js"
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        inter: ["Inter-Regular"],
        "inter-bold": ["Inter-Bold"],
        "inter-medium": ["Inter-Medium"],
      },
    },
  },
};
```

### 5. Use in components

```tsx
<Text className="font-inter">Regular</Text>
<Text className="font-inter-bold">Bold</Text>
```

> React Native ignores fallback fonts — always put the exact PostScript name as the first (and only) entry in the array.

### Common pitfalls

| Problem | Fix |
|---------|-----|
| Works on Android, not iOS | File name doesn't match PostScript name |
| Font not found at runtime | Check `app.json` plugin config |
| `font-bold` has no effect | `font-bold` sets `fontWeight`, not `fontFamily` — use `font-inter-bold` |

---

## NX Monorepo

Prerequisites: Expo project already set up in NX using `@nx/expo`. Skip the standard `metro.config.js` step from the main install guide.

### metro.config.js for NX

NX's `withNxMetro` returns a Promise, so `withNativeWind` must be chained:

```js title="metro.config.js"
const { withNativeWind } = require("nativewind/metro");
const { withNxMetro } = require("@nx/expo");
const { getDefaultConfig } = require("expo/metro-config");
const { mergeConfig } = require("metro-config");

const defaultConfig = getDefaultConfig(__dirname);
const customConfig = {
  // your project-specific overrides
};

module.exports = withNxMetro(mergeConfig(defaultConfig, customConfig), {
  // NX options
}).then((config) => withNativeWind(config, { input: "./global.css" }));
```

All other steps (Babel, global.css import, app.json) remain the same as the standard Expo install.

### References
- [NX React Native docs](https://nx.dev/recipes/react/react-native)
- [NX Expo plugin](https://nx.dev/nx-api/expo)
- [Expo monorepo guide](https://docs.expo.dev/guides/monorepos/)
