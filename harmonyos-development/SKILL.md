---
name: harmonyos-development
description: Comprehensive guide to HarmonyOS / HarmonyOS NEXT (鸿蒙) native app development using ArkTS and the ArkUI declarative UI framework. Use this skill when the user asks about 鸿蒙开发 / 华为开发者 / HarmonyOS, creating apps with DevEco Studio, ArkTS language, ArkUI components, the Stage model (UIAbility, ExtensionAbility, AbilityStage, WindowStage), state-management decorators (@State, @Prop, @Link, @Provide, @Consume, @Observed, @ObjectLink, @Watch, @Builder, @Styles), HarmonyOS Kits (Ability Kit, ArkUI, ArkGraphics, Network Kit, Media Kit, HiAI Foundation Kit, Location Kit, etc.), HAP/HSP/HAR packaging, app.json5/module.json5 configuration, distributed/atomic services (原子化服务/元服务), or the HarmonyOS sample code repository on developer.huawei.com/consumer/cn/samples.
---

# HarmonyOS (鸿蒙) Development

Covers HarmonyOS NEXT (API 12+) native app development — the Huawei mobile OS family that runs independently of Android (AOSP-free since HarmonyOS NEXT, released 2024). Primary language is **ArkTS** (a strict, statically-checked superset of TypeScript) and the primary UI framework is **ArkUI** (declarative, state-driven).

## When to use this skill

- Building, reading, or refactoring HarmonyOS / HarmonyOS NEXT apps (.hap / .hsp / .har modules)
- Writing ArkTS + ArkUI components with declarative `@Component`/`build()` syntax
- Questions about Stage model abilities, lifecycle, routing, or configuration files
- Debugging state-management decorators, rendering, or Observable propagation
- Choosing and calling HarmonyOS Kits (Ability, ArkUI, Network, Media, Location, HiAI, etc.)
- Building 原子化服务 (atomic services / meta-services) or cross-device distributed features
- Consulting the official samples catalog at https://developer.huawei.com/consumer/cn/samples/

## Platform snapshot

| Item | Value |
|---|---|
| OS | HarmonyOS NEXT (Pure HarmonyOS, AOSP-free) |
| Language | **ArkTS** (primary), C/C++ via NAPI, limited JS/TS |
| UI framework | **ArkUI** declarative (ArkUI-X for cross-platform) |
| Runtime | Ark Runtime, AOT-compiled by Ark Compiler |
| IDE | **DevEco Studio** (IntelliJ-based) |
| App model | **Stage model** (FA model is legacy — don't use in new apps) |
| Packaging | HAP (entry/feature), HSP (shared package), HAR (static archive) |
| Min API | API 9 for Stage model; API 12+ for HarmonyOS NEXT features |
| Sample catalog | https://developer.huawei.com/consumer/cn/samples/ |

## Project layout (Stage model)

```
MyApp/
├─ AppScope/
│  ├─ app.json5                 # global app config (bundleName, icon, label)
│  └─ resources/                # shared resources
├─ entry/                       # main HAP (entry module)
│  ├─ src/main/
│  │  ├─ ets/
│  │  │  ├─ entryability/EntryAbility.ets   # UIAbility subclass
│  │  │  ├─ entrybackupability/
│  │  │  └─ pages/Index.ets                 # ArkUI pages
│  │  ├─ resources/              # strings, colors, media, profiles
│  │  └─ module.json5            # module config, abilities, permissions
│  └─ build-profile.json5
├─ oh-package.json5              # ohpm dependencies
└─ build-profile.json5           # project-level
```

## The Stage model — core concepts

### Component types

- **UIAbility** — has UI, handles user interaction. Entry points. One instance per task.
- **ExtensionAbility** — background / extension scenarios:
  - `ServiceExtensionAbility` — background services (system apps)
  - `FormExtensionAbility` — home-screen widget (服务卡片)
  - `WorkSchedulerExtensionAbility` — deferred tasks
  - `InputMethodExtensionAbility`, `WallpaperExtensionAbility`, `BackupExtensionAbility`, etc.
- **AbilityStage** — module-level lifecycle container (one per HAP)
- **WindowStage** — window container scoped to a UIAbility

### UIAbility lifecycle

```ts
import { UIAbility, Want, AbilityConstant } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void { }
  onDestroy(): void { }
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/Index', (err) => { /* ... */ });
  }
  onWindowStageDestroy(): void { }
  onForeground(): void { }
  onBackground(): void { }
  onNewWant(want: Want, launchParam: AbilityConstant.LaunchParam): void { }
}
```

### Launching another ability

```ts
import { common, Want } from '@kit.AbilityKit';

const ctx = getContext(this) as common.UIAbilityContext;
const want: Want = {
  bundleName: 'com.example.app',
  abilityName: 'DetailAbility',
  parameters: { id: 42 }
};
ctx.startAbility(want);
// or startAbilityForResult for a return value
```

### module.json5 essentials

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",                // entry | feature | shared
    "srcEntry": "./ets/entryability/EntryAbility.ets",
    "deviceTypes": ["phone", "tablet", "2in1"],
    "abilities": [{
      "name": "EntryAbility",
      "srcEntry": "./ets/entryability/EntryAbility.ets",
      "startWindowIcon": "$media:startIcon",
      "startWindowBackground": "$color:start_window_background",
      "skills": [{
        "actions": ["action.system.home"],
        "entities": ["entity.system.home"]
      }]
    }],
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

## ArkTS language — strictness rules

ArkTS = TypeScript MINUS dynamic patterns, PLUS stricter static typing for AOT perf.

**Prohibited / changed vs TS:**
- No `any` / `unknown` as everyday escape hatches — declare real types
- No dynamic property add/delete on objects (`obj.foo = bar` on untyped object is a compile error)
- No `Function.prototype.bind`, `call`, `apply` with reshaped `this`
- No structural typing shortcuts — use nominal classes / interfaces
- No `Object.keys`/`Object.assign` for arbitrary reshaping
- `Record<K,V>` for true dictionary maps
- Use `class` / `interface` with explicit fields

```ts
// Prefer:
class User { constructor(public id: number, public name: string) {} }
// Over:
// const user: any = { id: 1, name: 'A' }
```

## ArkUI — declarative UI

### Custom component basics

```ts
@Entry                      // marks route/page entry
@Component
struct Index {
  @State count: number = 0;

  build() {
    Column({ space: 12 }) {
      Text(`Count: ${this.count}`)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
      Button('Increment')
        .onClick(() => { this.count++; })
    }
    .width('100%').height('100%').justifyContent(FlexAlign.Center)
  }
}
```

### State-management decorators

| Decorator | Scope | Purpose |
|---|---|---|
| `@State` | within component | Owned mutable state; triggers re-render |
| `@Prop` | parent → child | One-way copy (child has local copy) |
| `@Link` | parent ↔ child | Two-way binding (use `$var` to pass) |
| `@Provide` / `@Consume` | ancestor → descendant | Cross-level implicit binding by key |
| `@Observed` (class) + `@ObjectLink` (prop) | class instances | Observe changes to class properties |
| `@Watch('handler')` | any of above | Callback on value change |
| `@StorageLink` / `@StorageProp` | app-wide `AppStorage` | Global reactive state |
| `@LocalStorageLink` / `@LocalStorageProp` | page-scoped | Scoped reactive state |

**Passing `@Link`:**
```ts
@Entry @Component struct Parent {
  @State val: number = 0;
  build() { Child({ val: $val }) }     // $ prefix for @Link
}
@Component struct Child {
  @Link val: number;
  build() { Button(`${this.val}`).onClick(() => this.val++) }
}
```

**Observing class objects:**
```ts
@Observed class Task { constructor(public title: string, public done: boolean) {} }

@Component struct TaskRow {
  @ObjectLink task: Task;            // re-renders when task.title/done changes
  build() { Text(this.task.title) }
}
```

Arrays of `@Observed` instances require `@ObjectLink` in the row component — parent `@State tasks: Task[]` only reacts to array mutations (push/splice/reassign), not per-item changes.

### Builders, styles, extends

```ts
@Builder function Label(text: string) { Text(text).fontSize(16) }

@Styles function Card() { .padding(12).borderRadius(12).backgroundColor('#FFF') }

@Extend(Text) function title() { .fontSize(22).fontWeight(FontWeight.Bold) }

// Usage:
Text('Hello').title()
Column() { Label('x') } .Card()
```

### Navigation (recommended: Navigation component, not Router)

```ts
@Entry @Component struct App {
  @Provide('pathStack') pathStack: NavPathStack = new NavPathStack();
  build() {
    Navigation(this.pathStack) {
      Button('Go').onClick(() => this.pathStack.pushPath({ name: 'Detail', param: 42 }))
    }
    .navDestination((name: string, param: Object) => {
      if (name === 'Detail') Detail({ id: param as number })
    })
  }
}
```

The older `router` module (`@ohos.router`) still works but `Navigation` + `NavPathStack` is the modern API for HarmonyOS NEXT.

## HarmonyOS Kits (common imports)

```ts
import { UIAbility, Want, common, abilityAccessCtrl } from '@kit.AbilityKit';
import { window, display, promptAction, router } from '@kit.ArkUI';
import { http, webSocket, connection } from '@kit.NetworkKit';
import { media, image, camera } from '@kit.MediaLibraryKit' | '@kit.ImageKit' | '@kit.CameraKit';
import { geoLocationManager } from '@kit.LocationKit';
import { relationalStore, preferences, distributedKVStore } from '@kit.ArkData';
import { fileIo as fs, picker } from '@kit.CoreFileKit';
import { notificationManager } from '@kit.NotificationKit';
import { hiAIFoundation } from '@kit.HiAIVisionKit';
```

### HTTP request example

```ts
import { http } from '@kit.NetworkKit';

async function fetchJson(url: string): Promise<string> {
  const req = http.createHttp();
  try {
    const res = await req.request(url, {
      method: http.RequestMethod.GET,
      header: { 'Content-Type': 'application/json' },
      connectTimeout: 5000, readTimeout: 10000
    });
    return res.result as string;
  } finally {
    req.destroy();
  }
}
```

### Permissions

Declare in `module.json5` → `requestPermissions`. For user-granted permissions, request at runtime via `abilityAccessCtrl.createAtManager().requestPermissionsFromUser(context, [...])`.

### Persistent storage options

- **Preferences** — lightweight key-value (`@kit.ArkData` → `preferences`)
- **Relational Store** — SQLite (`relationalStore`)
- **Distributed KV Store** — cross-device sync
- **File I/O** — `@kit.CoreFileKit` `fileIo` / sandbox paths

## Atomic Services / Meta-Services (原子化服务 / 元服务)

Installation-free, small (≤10MB) apps launched from:
- Service cards (home-screen widgets built with `FormExtensionAbility`)
- Search, scan, NFC, cross-device continuation

Configured via `module.json5` with `"installationFree": true`. Build-time they produce `.app` packages containing multiple HAPs.

## Distributed features

- **Cross-device continuation** (流转) — `continueAbility` / `onContinue`/`onNewWant` lifecycle
- **Distributed data object** — synced state across devices
- **Distributed file system** — shared sandbox
- **Device manager** — discover trusted devices

## Packaging types

| Type | Purpose |
|---|---|
| **HAP** (Harmony Ability Package) | Runnable module: `entry` or `feature` type |
| **HSP** (Harmony Shared Package) | In-app shared module, dynamic-linked at runtime |
| **HAR** (Harmony Archive) | Static library, packaged into HAP at build time |

Distribute via AppGallery Connect (华为应用市场).

## Sample catalog (developer.huawei.com/consumer/cn/samples)

The official samples repo is organized by category. Typical categories include:

- **Getting Started / 起步** — First app, component basics, layout examples
- **ArkUI / 界面开发** — List, Grid, Swiper, Navigation, custom components, animations, WaterFlow, Tabs
- **Ability 框架** — UIAbility lifecycle, routing, startAbilityForResult, ExtensionAbility samples
- **Data Management / 数据管理** — Preferences, RDB, DataShare, distributed data object
- **File Management / 文件管理** — Sandbox IO, picker, media library access
- **Network & Connectivity / 网络** — HTTP, WebSocket, Bluetooth, Wi-Fi P2P, NFC
- **Media / 多媒体** — Audio playback/recording, camera, image, video
- **Graphics / 图形图像** — ArkGraphics 2D, XComponent + native OpenGL/Vulkan
- **AI / HiAI** — Image classification, OCR, ASR, TTS, speech recognition
- **Security / 安全** — HUKS key management, crypto, biometric auth
- **Distributed / 分布式** — Cross-device continuation, distributed scheduler, distributed KV
- **Atomic Services / 元服务** — Service card templates, form extension samples
- **Wearable / 穿戴** — HarmonyOS watch / band apps
- **Vehicle / 车机** — HarmonyOS cockpit samples
- **System / 系统** — Window management, notifications, input method, wallpaper

When helping the user find a sample, recommend they search the catalog by Kit name or by the API they need (e.g. "relationalStore", "FormExtensionAbility").

## Common gotchas

1. **Don't mix FA and Stage models** — FA is legacy; HarmonyOS NEXT only supports Stage.
2. **`this` in ArkUI callbacks** — arrow functions are required; regular `function () {}` loses `this`.
3. **`@State` on nested objects** — changes to nested props don't trigger updates; use `@Observed`/`@ObjectLink` or reassign the whole object.
4. **Array item updates** — replace the item (`arr[i] = newItem`) or use `@Observed` on the item class.
5. **Resource references** — use `$r('app.string.foo')`, `$r('app.media.icon')`, not string paths.
6. **`getContext(this)`** inside a component returns the `UIAbilityContext`; cast explicitly.
7. **Async in `build()`** is forbidden — load data in `aboutToAppear()` and store in `@State`.
8. **Permissions must be declared AND requested at runtime** for user-grant permissions.
9. **ohpm** is the package manager (similar to npm) — dependencies live in `oh-package.json5`.
10. **Preview on device** — DevEco Previewer doesn't fully simulate; always test on real HarmonyOS device or emulator.

## Debugging & tooling

- **DevEco Studio** — primary IDE, includes emulator, previewer, profiler, HiLog viewer
- **hdc** — Harmony Device Connector (like adb): `hdc shell`, `hdc file send`, `hdc hilog`
- **HiLog** — logging: `hilog.info(0x0001, 'TAG', 'message %{public}s', arg)`
- **Instruments: SmartPerf / DevEco Profiler** — CPU/GPU/memory/energy profiling

## Publishing

1. Obtain a HarmonyOS developer account, complete real-name verification
2. Generate signing certificate + profile in AppGallery Connect
3. Configure signing in `build-profile.json5`
4. Build release HAP/APP bundle: `hvigorw assembleApp`
5. Upload to **AppGallery Connect** for review

## Useful references

- Samples catalog: https://developer.huawei.com/consumer/cn/samples/
- Official docs: https://developer.huawei.com/consumer/cn/doc/
- ArkTS spec: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-overview-V5
- ArkUI component reference: search "ArkUI 声明式组件"
- DevEco Studio: https://developer.huawei.com/consumer/cn/deveco-studio/
