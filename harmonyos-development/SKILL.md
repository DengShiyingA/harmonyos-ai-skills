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
| Language | **ArkTS** (primary), **Cangjie** (beta), C/C++ via NAPI |
| UI framework | **ArkUI** declarative (ArkUI-X for cross-platform) |
| Compiler | **ArkCompiler** — AOT to native machine code; LiteActor concurrency |
| Package manager | **ohpm** — `oh-package.json5`; registry at DevEco Service (OHPM Central) |
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
import { window, display, promptAction } from '@kit.ArkUI';
import { http, webSocket, connection } from '@kit.NetworkKit';
import { photoAccessHelper } from '@kit.MediaLibraryKit';   // photo/video picker
import { fileIo as fs } from '@kit.CoreFileKit';
import { geoLocationManager } from '@kit.LocationKit';
import { relationalStore, preferences, distributedKVStore } from '@kit.ArkData';
import { notificationManager } from '@kit.NotificationKit';
import { speechRecognizer } from '@kit.CoreSpeechKit';       // ASR / TTS
import { hilog } from '@kit.PerformanceAnalysisKit';
```

### Full Kit catalog — official SDK categories (developer.huawei.com/consumer/cn/sdk/)

**应用框架 Application Framework**

| Kit | Import key | Purpose |
|---|---|---|
| Ability Kit | `AbilityKit` | UIAbility, ExtensionAbility, Want, context, routing |
| Accessibility Kit | `AccessibilityKit` | Screen reader, universal design, a11y services |
| ArkData | `ArkData` | relationalStore, preferences, distributedKVStore |
| ArkTS | — | Language spec & Ark compiler tooling |
| ArkUI | `ArkUI` | window, display, promptAction, Navigation, components |
| ArkWeb | `ArkWeb` | WebView / Web component embedding |
| Background Tasks Kit | `BackgroundTasksKit` | Deferred/scheduled background work, transient tasks |
| Core File Kit | `CoreFileKit` | fileIo, picker, sandbox paths |
| Form Kit | `FormKit` | FormExtensionAbility, home-screen service cards/widgets |
| IME Kit | `IMEKit` | Input method engine development |
| IPC Kit | `IPCKit` | Inter-process communication (Parcel, RemoteObject) |
| Localization Kit | `LocalizationKit` | i18n, l10n, RTL, pseudo-localization |

**应用服务 Application Services**

| Kit | Import key | Purpose |
|---|---|---|
| Account Kit | `AccountKit` | One-click Huawei ID login, OAuth 2.0 |
| Ads Kit | `AdsKit` | Advertising SDK (banner, native, interstitial) |
| App Linking Kit | `AppLinkingKit` | Deep links, deferred deep links |
| Calendar Kit | `CalendarKit` | Calendar events, reminders |
| Contacts Kit | `ContactsKit` | Contact read/write/search |
| IAP Kit | `IAPKit` | In-app purchases (consumable, non-consumable, subscription) |
| Live View Kit | `LiveViewKit` | Live activities on lock screen / notification center |
| Location Kit | `LocationKit` | geoLocationManager, geofencing, geocoding |
| Map Kit | `MapKit` | Huawei Maps SDK, routing, place search |
| Notification Kit | `NotificationKit` | Local + push notifications, badges |
| Payment Kit | `PaymentKit` | Huawei Pay transaction processing |
| Push Kit | `PushKit` | Push notification delivery service |
| Scan Kit | `ScanKit` | QR/barcode scanning |
| Share Kit | `ShareKit` | Cross-app content sharing |

**系统 System**

| Kit | Import key | Purpose |
|---|---|---|
| Asset Store Kit | `AssetStoreKit` | Secure credential/secret storage |
| Basic Services Kit | `BasicServicesKit` | Battery, vibration, thermal, brightness, clipboard |
| Connectivity Kit | `ConnectivityKit` | Bluetooth, Wi-Fi, NFC, USB, hotspot |
| Crypto Architecture Kit | `CryptoArchitectureKit` | Encryption, key management, certificates |
| Distributed Service Kit | `DistributedServiceKit` | deviceManager, cross-device discovery |

**媒体 Media**

| Kit | Import key | Purpose |
|---|---|---|
| Audio Kit | `AudioKit` | Audio playback, recording, routing, focus |
| AVCodec Kit | `AVCodecKit` | Hardware-accelerated encode/decode (H.264, H.265, AAC…) |
| Camera Kit | `CameraKit` | Camera preview, photo capture, video recording |
| Image Kit | `ImageKit` | Image decoding, transformation, EXIF |
| Media Kit | `MediaKit` | AVPlayer, AVRecorder (unified playback/recording) |
| Media Library Kit | `MediaLibraryKit` | photoAccessHelper, media library CRUD |
| Scan Kit | `ScanKit` | QR/barcode scanning |

**图形 Graphics**

| Kit | Import key | Purpose |
|---|---|---|
| ArkGraphics 2D | `ArkGraphics2D` | 2D Canvas drawing, effects, blur, shadow |
| ArkGraphics 3D | `ArkGraphics3D` | 3D scene graph, glTF rendering |
| XComponent | (ArkUI built-in) | Native OpenGL ES / Vulkan surface via NAPI |

**AI**

| Kit | Import key | Purpose |
|---|---|---|
| Core Speech Kit | `CoreSpeechKit` | On-device ASR / TTS engine |
| Core Vision Kit | `CoreVisionKit` | OCR, face detection, image segmentation, super-resolution |
| Intents Kit | `IntentsKit` | Intent recognition (30+ domains, 60+ built-in intents) |
| MindSpore Lite | `MindSporeLiteKit` | Edge model inference (Caffe/TF/ONNX/MindIR) |
| Natural Language Kit | `NaturalLanguageKit` | Word segmentation, NER, keyword extraction |

**Performance & DevTools**

| Kit | Import key | Purpose |
|---|---|---|
| Performance Analysis Kit | `PerformanceAnalysisKit` | hilog, HiAppEvent, crash analysis |

### Photo/video picker (correct API — `picker.PhotoViewPicker` is deprecated)

```ts
import { photoAccessHelper } from '@kit.MediaLibraryKit';

const phAccessHelper = photoAccessHelper.getPhotoAccessHelper(context);
const picker = new photoAccessHelper.PhotoViewPicker();
const result = await picker.select({
  MIMEType: photoAccessHelper.PhotoViewMIMETypes.IMAGE_TYPE,
  maxSelectNumber: 9
});
const uris: string[] = result.photoUris;
```

### CoreSpeechKit — ASR audio format requirements

```ts
import { speechRecognizer } from '@kit.CoreSpeechKit';

const engine = await speechRecognizer.createEngine({
  language: 'zh-CN',
  online: 0   // 0=on-device, 1=cloud
});
// Audio MUST be: PCM, 16kHz, mono, 16-bit
// Chunk size: 640 or 1280 bytes, write every 20ms or 40ms
engine.startListening({
  sessionId: `asr_${Date.now()}`,
  audioInfo: {
    audioType: 'pcm',
    sampleRate: 16000,
    soundChannel: 1,
    sampleBit: 16
  }
});
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

The official samples repo is organized by the following **10 official categories** (as shown on the HarmonyOS developer samples page):

| 类别 | Category | What it covers |
|---|---|---|
| **HarmonyOS特征** | HarmonyOS Features | Platform-differentiating capabilities: 一多 (one-develop multi-deploy) responsive layout, 分布式流转 cross-device continuation, 原子化服务 / 服务卡片 atomic services & service cards, 超级终端 super device, 方舟编译器 Ark compiler features, 鸿蒙智联 HarmonyOS Connect |
| **技术质量** | Technical Quality | Performance optimization (startup time, frame rate, memory), stability (crash/ANR), power consumption, security hardening, code quality, HiAppEvent / HiLog diagnostics, DFX capabilities |
| **应用框架开发** | Application Framework | UIAbility / ExtensionAbility lifecycle, Ability routing & want, AppStartup, state management (@State/@Link/@Provide/@Observed), ArkUI components & layouts, Navigation / NavPathStack, Router, window management, internationalization, resource management, notifications, background tasks |
| **系统开发** | System Development | Power/battery APIs, telephony (call/SMS), account system, device management, settings, accessibility, input method, wallpaper, system apps, low-level OS integration |
| **媒体开发** | Media Development | AVPlayer / AVRecorder audio & video playback/recording, camera (CameraKit), image codec (ImageKit), media library, AVSession, audio focus, DRM, HDR |
| **图形开发** | Graphics Development | ArkGraphics 2D, XComponent + native OpenGL ES / Vulkan, 3D rendering, Canvas, animations & transitions, effects, WebGL, SVG, drawing APIs |
| **应用服务开发** | Application Service | Location (LocationKit / geoLocationManager), maps, push notifications, payments, account sign-in (Huawei ID), sharing, scanning, health/fitness, wallet, in-app purchase |
| **AI功能开发** | AI Function Development | HiAI Foundation Kit, CoreVisionKit (image classification, OCR, face detection), SpeechKit (ASR/TTS), natural language, MindSpore Lite on-device inference, AI image enhancement, generative models |
| **应用专项测试** | Application Testing | Unit testing (ohUnit / Hypium), UI automation testing, performance testing, compatibility testing, monkey / stress testing, smartperf profiling, DevEco Testing |
| **开发工具** | Development Tools | DevEco Studio plugins & workflows, hvigor build scripts, ohpm package management, hdc device connector, Previewer, Profiler, Code Linter, debugging & logging tools |

> Browse the live catalog at https://developer.huawei.com/consumer/cn/samples/ — filter by category or search by Kit name / API (e.g. "relationalStore", "FormExtensionAbility", "AVPlayer").

**When helping the user pick a sample:**
1. Identify which of the 10 categories the feature belongs to
2. Narrow by the specific Kit (AbilityKit, MediaKit, HiAIVisionKit, LocationKit, etc.)
3. Match by API name or scenario (e.g. "service card" → 应用框架开发 + FormExtensionAbility)

## Multi-device (一多) responsive layout

HarmonyOS supports a single codebase across phone, tablet, 2-in-1, watch, smartscreen, and automotive. Use breakpoints in **vp** (not px).

### Breakpoints

| Breakpoint | vp width | Target |
|---|---|---|
| xs | < 320 | watch |
| sm | 320–600 | phone portrait |
| md | 600–840 | phone landscape / small tablet |
| lg | 840–1440 | tablet / 2-in-1 |
| xl | ≥ 1440 | large screen |

**Always convert px → vp before comparing:**
```ts
import { display } from '@kit.ArkUI';
function getBreakpoint(): string {
  const dm = display.getDefaultDisplaySync();
  const w = dm.width / dm.densityPixels; // vp
  if (w < 600) return 'sm';
  if (w < 840) return 'md';
  return 'lg';
}
```

### GridRow / GridCol (recommended for responsive grids)

```ts
GridRow({ columns: { sm: 4, md: 8, lg: 12 }, gutter: { x: 12 } }) {
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) { LeftPanel() }
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) { RightPanel() }
}
```

### Foldable / hover mode
Listen for display change and re-check breakpoint:
```ts
display.on('change', (_id: number) => { this.bp = getBreakpoint(); });
display.off('change');
```

## sys.symbol — icon glyph system

HarmonyOS Symbol is a 1500+ vector icon font with multi-layer color and 7 animation types.

```ts
SymbolGlyph($r('sys.symbol.bell_fill'))
  .fontSize(24)
  .fontColor([Color.Blue, Color.Green])
  .symbolEffect(new BounceSymbolEffect(EffectScope.WHOLE), true)
```

**Confirmed working names** (SDK 6.0.1): `xmark` · `plus` · `minus` · `checkmark` · `chevron_right` · `chevron_left` · `star` · `star_fill` · `bell` · `bell_fill` · `doc` · `video` · `mic` · `mic_fill` · `clock` · `trash` · `pencil` · `image` · `person`

**Names that do NOT exist** (common mistakes): `photo` · `doc_richtext` · `sparkles` · `checklist`

Find valid names: https://developer.huawei.com/consumer/cn/design/harmonyos-symbol

## DevEco Studio 6.x — Project Setup

DevEco Studio 6.x requires **additional Hvigor infrastructure files**. Without them the IDE shows "工程结构及配置需要升级".

### hvigorfile.ts (root + each module)

```ts
// root
import { appTasks } from '@ohos/hvigor-ohos-plugin';
export default { system: appTasks, plugins:[] }

// entry/hvigorfile.ts
import { hapTasks } from '@ohos/hvigor-ohos-plugin';
export default { system: hapTasks, plugins:[] }
```

### hvigor/wrapper/hvigor-config.json5

```json5
{
  "hvigorVersion": "5.0.0",
  "dependencies": {
    "@ohos/hvigor-ohos-plugin": "5.0.0"
  }
}
```

### build-profile.json5 — SDK version placement

SDK versions go in **one place only** — either under `app` OR inside each `product`, not both. Mixing them causes "Sync Failed".

**Recommended workflow:** Always let DevEco Studio generate the scaffold (`新建项目` → Empty Ability), then copy your source files in.

## ArkTS strict-mode compiler errors (SDK 6.0.1)

### No object literals as types
```ts
// ❌
parseOutput(): { text: string; tags: string[] } { ... }
// ✅
interface ParsedOutput { text: string; tags: string[]; }
parseOutput(): ParsedOutput { ... }
```

### No `any` / `unknown`
```ts
const task = JSON.parse(rawStr) as AgentTask;  // ✅ cast immediately
```

### `navDestination` requires a `@Builder` function reference
```ts
// ❌ inline lambda
.navDestination((name, param) => { if (name==='X') MyPage() })
// ✅ top-level @Builder
@Builder function PageRouter(name: string) { if (name==='X') MyPage() }
Navigation(stack) { ... }.navDestination(PageRouter)
```

### `@Entry` build() root must be a container
Wrap in `Stack()`, `Column()`, or `Row()` — a custom component alone is not a container.

### `display.width` is pixels, not vp
```ts
function isTablet(): boolean {
  const dm = display.getDefaultDisplaySync();
  return (dm.width / dm.densityPixels) >= 840;
}
```

### All user_grant permissions need `reason` + `usedScene`
```json5
{
  "name": "ohos.permission.READ_MEDIA",
  "reason": "$string:permission_media_reason",
  "usedScene": { "abilities": ["EntryAbility"], "when": "inuse" }
}
```
`INTERNET` is system_grant — no extra fields needed.

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

## 仓颉 (Cangjie) — alternative language (Beta)

| | ArkTS | Cangjie |
|---|---|---|
| Status | **Production** (use this now) | **Beta / developer preview** |
| Base | TypeScript superset | New language (Huawei-designed) |
| Key feature | Familiar TS syntax, strict AOT typing | Concurrent GC, embedded AgentDSL for AI |
| Use for | All current HarmonyOS NEXT apps | Experimentation only until stable |

## ArkCompiler — runtime details

- **AOT mode** — static type info generates optimized native machine code; no JIT warm-up
- **LiteActor concurrency** — lightweight actor model; shared immutable objects avoid copy overhead
- **Taskpool / Worker** — two concurrency APIs:
  - `taskpool.execute(task)` — fire-and-forget parallel tasks (preferred)
  - `new worker.ThreadWorker(script)` — long-lived background threads

```ts
import { taskpool } from '@kit.ArkTS';

@Concurrent
function heavyCalc(n: number): number { return n * n; }

const result = await taskpool.execute(heavyCalc, 42);
```

## OHPM — package manager

```bash
ohpm install @ohos/axios          # install a library
ohpm install                      # install all from oh-package.json5
```

**oh-package.json5 example:**
```json5
{
  "name": "entry",
  "dependencies": {
    "@ohos/axios": "^2.0.0"
  }
}
```

OHPM Central Repository: https://developer.huawei.com/consumer/cn/deveco-service

## Debugging & tooling

- **DevEco Studio** — primary IDE, includes emulator, previewer, profiler, HiLog viewer
- **hdc** — Harmony Device Connector (like adb): `hdc shell`, `hdc file send`, `hdc hilog`
- **HiLog** — logging: `hilog.info(0x0001, 'TAG', 'message %{public}s', arg)`
- **Instruments: SmartPerf / DevEco Profiler** — CPU/GPU/memory/energy profiling
- **DevEco Testing** — UI automation, performance testing, monkey/stress, compatibility

## Publishing

1. Obtain a HarmonyOS developer account, complete real-name verification
2. Generate signing certificate + profile in AppGallery Connect
3. Configure signing in `build-profile.json5`
4. Build release HAP/APP bundle: `hvigorw assembleApp`
5. Upload to **AppGallery Connect** for review

## Best practices (official)

Official best-practice docs at: https://developer.huawei.com/consumer/cn/best-practices

Key topics: ArkTS高性能编程, 布局优化, 组件绘制优化, 内存分析, 功耗优化, 应用安全编码, ArkWeb安全, 自由流转/跨端迁移, NDK跨语言调用, 行业场景解决方案 (新闻阅读 / 直播连麦 / 影音娱乐)

## Useful references

- Official docs home: https://developer.huawei.com/consumer/cn/doc/
- Getting started guide: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide
- ArkTS language: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts
- ArkTS high-perf programming: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-high-performance-programming
- ArkUI framework: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui
- State management: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-state-management-overview
- Navigation & routing: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-set-navigation-routing
- Animations: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-use-animation
- Concurrency: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-concurrency
- ArkData (data mgmt): https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/data-mgmt-overview
- Samples catalog: https://developer.huawei.com/consumer/cn/samples/
- SDK / Kit overview: https://developer.huawei.com/consumer/cn/sdk
- Best practices: https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-best-practices-overview
- DevEco Studio: https://developer.huawei.com/consumer/cn/deveco-studio/
- DevEco Testing: https://developer.huawei.com/consumer/cn/deveco-testing
- OHPM package registry: https://developer.huawei.com/consumer/cn/deveco-service
- Design resources: https://developer.huawei.com/consumer/cn/design/resource
- HarmonyOS Symbol icons: https://developer.huawei.com/consumer/cn/design/harmonyos-symbol
- Codelabs (hands-on tutorials): https://developer.huawei.com/consumer/cn/codelabsPortal/serviceTypes
- Quick-start Codelab: https://developer.huawei.com/consumer/cn/codelabsPortal/getstarted/101718800110527001
- Knowledge map: https://developer.huawei.com/consumer/cn/app/knowledge-map
- AppGallery Connect: https://developer.huawei.com/consumer/cn/agconnect
- App review policy: https://developer.huawei.com/consumer/cn/doc/app/50000
- HarmonyOS developer certification: https://developer.huawei.com/consumer/cn/training/dev-cert-detail/101666948302721398
- Cangjie beta signup: https://developer.huawei.com/consumer/cn/activityDetail/cangjie-beta
