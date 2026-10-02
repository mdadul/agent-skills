---
name: nativewind-expo
description: Installs and configures NativeWind v4 (Tailwind CSS for React Native) in an Expo project, covering package installation, Tailwind config, Babel preset, Metro config, global CSS import, and app.json bundler switch. Use when the user asks to add NativeWind, set up Tailwind CSS in Expo, configure nativewind, or integrate Tailwind with a React Native / Expo project.
---

# NativeWind v4 — Expo Installation

## Purpose
Walk through every required step to add NativeWind v4 to an existing Expo project, producing a working Tailwind CSS setup for iOS, Android, and web.

## When To Use
- "add NativeWind to my Expo project"
- "set up Tailwind CSS in Expo / React Native"
- "configure nativewind v4"
- "install nativewind with Expo"

## Inputs
- An existing Expo project (SDK 49+)
- Package manager preference (npm / yarn / pnpm / bun)

## Workflow

### Step 1 — Install packages

Pick the command that matches the project's package manager:

```bash
# npm
npm install nativewind tailwindcss react-native-reanimated react-native-safe-area-context

# yarn
yarn add nativewind tailwindcss react-native-reanimated react-native-safe-area-context

# pnpm
pnpm add nativewind tailwindcss react-native-reanimated react-native-safe-area-context

# bun
bun add nativewind tailwindcss react-native-reanimated react-native-safe-area-context
```

### Step 2 — Create tailwind.config.js

Run the Tailwind init command:

```bash
npx tailwindcss init
```

Then replace the generated file with the NativeWind preset and content paths that match the project:

```js title="tailwind.config.js"
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./App.{js,jsx,ts,tsx}",
    "./app/**/*.{js,jsx,ts,tsx}",
    "./components/**/*.{js,jsx,ts,tsx}",
  ],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

> Adjust the `content` globs to match the actual source layout (e.g. `./src/**/*.{js,jsx,ts,tsx}`).

### Step 3 — Create global.css

Create `global.css` in the project root:

```css title="global.css"
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Step 4 — Configure Babel

Edit `babel.config.js`:

```js title="babel.config.js"
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      ["babel-preset-expo", { jsxImportSource: "nativewind" }],
      "nativewind/babel",
    ],
  };
};
```

### Step 5 — Configure Metro

Create or update `metro.config.js`:

```js title="metro.config.js"
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: "./global.css" });
```

### Step 6 — Import global.css in the entry file

Add the import to the root entry file. For **Expo Router** projects this is `app/_layout.tsx`; for bare Expo projects it is `App.tsx` (or `App.js`):

```tsx
import "./global.css";
```

### Step 7 — Switch the web bundler in app.json

```json title="app.json"
{
  "expo": {
    "web": {
      "bundler": "metro"
    }
  }
}
```

### Step 8 — Clear caches and restart

```bash
npx expo start --clear
```

### Step 9 — Verify

Replace the root view with a quick smoke test:

```tsx
import { Text, View } from "react-native";

export default function App() {
  return (
    <View className="flex-1 items-center justify-center bg-white">
      <Text className="text-2xl font-bold text-blue-500">NativeWind works!</Text>
    </View>
  );
}
```

If the text renders blue and centered, the setup is complete.

## Output Format
After completing all steps, confirm:
- [ ] `nativewind` and `tailwindcss` appear in `package.json` dependencies
- [ ] `tailwind.config.js` contains `presets: [require("nativewind/preset")]`
- [ ] `global.css` exists with all three `@tailwind` directives
- [ ] `babel.config.js` uses `jsxImportSource: "nativewind"` and `"nativewind/babel"`
- [ ] `metro.config.js` wraps config with `withNativeWind`
- [ ] `global.css` is imported in the entry file
- [ ] `app.json` sets `web.bundler` to `"metro"`

## Optional Topics
See [REFERENCE.md](./REFERENCE.md) for:
- TypeScript setup
- Dark mode (system preference + manual toggle)
- Custom fonts with expo-font
- NX monorepo wiring

## Common Pitfalls
| Problem | Fix |
|---------|-----|
| Classes compile but styles don't apply | Confirm `global.css` is imported in the entry file |
| Metro error on startup | Delete `.expo/` and node_modules cache, re-run `npx expo start --clear` |
| Web styles missing | Ensure `app.json` has `"bundler": "metro"` under `expo.web` |
| TypeScript errors on `className` | Add `/// <reference types="nativewind/types" />` — see REFERENCE.md |
| Monorepo: `withNativeWind` not a function | Use promise-chain pattern — see REFERENCE.md |
