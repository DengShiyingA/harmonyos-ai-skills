---
name: harmonyos-development
description: >
  HarmonyOS 6.0.2 / NEXT (鸿蒙) development knowledge — ArkTS, ArkUI, Stage model,
  50+ Kit APIs, state management, navigation, animation, concurrency, testing, security.

  Trigger: HarmonyOS, ArkTS, ArkUI, .ets, DevEco Studio, Stage model, UIAbility,
  @State, @Prop, @Link, @Provide, @Observed, @ObjectLink, @ComponentV2, @ObservedV2,
  @Trace, @Monitor, StateStore, NavPathStack, LazyForEach, @Reusable, animateTo,
  TaskPool, Worker, @Concurrent, @Sendable, module.json5, oh-package.json5, OHPM,
  sys.symbol, HAP, HSP, relationalStore, preferences, notificationManager,
  backgroundTaskManager, abilityAccessCtrl, arkxtest, BlurStyle, systemMaterialEffect,
  鸿蒙, 鸿蒙开发, 鸿蒙NEXT, 方舟语言, 状态管理, 导航路由, 懒加载, 组件复用,
  鸿蒙权限, 鸿蒙测试, 液态玻璃, 沉浸光感, 折叠屏, 鸿蒙入门
---

# HarmonyOS (鸿蒙) Development

Covers HarmonyOS 6.0.2(22) / HarmonyOS NEXT native app development — the Huawei mobile OS family that runs independently of Android (AOSP-free since HarmonyOS NEXT, released 2024). Primary language is **ArkTS** (a strict, statically-checked superset of TypeScript) and the primary UI framework is **ArkUI** (declarative, state-driven). HarmonyOS 6.0 adds system-level "Immersive Light Perception" visual effects (液态玻璃/沉浸光感视效).

## Platform snapshot

| Item | Value |
|---|---|
| OS | **HarmonyOS 6.0.2(22)** (current stable, Pure HarmonyOS, AOSP-free) |
| Language | **ArkTS** (primary), **Cangjie** (beta), C/C++ via NAPI |
| UI framework | **ArkUI** declarative (ArkUI-X for cross-platform) |
| Compiler | **ArkCompiler** — AOT to native machine code; LiteActor concurrency |
| Package manager | **ohpm** — `oh-package.json5`; registry at DevEco Service (OHPM Central) |
| IDE | **DevEco Studio 6.0.2 Release** (Hvigor 6.22.x, IntelliJ-based) |
| App model | **Stage model** (FA model is legacy — don't use in new apps) |
| Packaging | HAP (entry/feature), HSP (shared package), HAR (static archive), atomic .app |
| Recommended API | **API 20+ (strongly recommend API 22)** |
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

### Naming conventions (official coding style guide)

| Element | Convention | Example |
|---|---|---|
| Classes, Structs, Enums | **UpperCamelCase** | `PersonInfo`, `ColorType` |
| Variables, Parameters, Methods | **lowerCamelCase** | `userName`, `getUserInfo()` |
| Constants | **UPPER_SNAKE_CASE** | `MAX_VALUE`, `DEFAULT_TIMEOUT` |
| Boolean variables | Prefix with `is`, `has`, `can` | `isVisible`, `hasPermission` |

Formatting: 2-space indent, max 120 chars/line, K&R braces, always use `{}` for if/for/while.

### High-performance ArkTS rules (from official best practices)

1. Use `const` for unchanging values — enables engine optimization
2. Never mix int and float in same variable — `let n = 1; n = 1.1;` causes boxing overhead
3. Use TypedArrays (`Int8Array`, `Float32Array`) for numerical computation
4. Avoid sparse arrays — `arr[9999] = 0` forces hash-table storage (much slower)
5. Don't mix types in arrays — `[1, "a", 2]` deoptimizes
6. Cache property lookups outside hot loops
7. Avoid exception throwing in perf-critical loops — use sentinel values
8. Minimize closures in hot paths — pass variables via function params instead
9. Use Array methods (`forEach`, `map`, `filter`, `reduce`) — internally optimized
10. Keep `build()` pure and declarative — no side effects, load data in `aboutToAppear()`

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

### Component lifecycle callbacks

**All `@Component`:**

| Callback | When | Notes |
|---|---|---|
| `aboutToAppear()` | After created, BEFORE `build()` | Can change state here; changes apply in first `build()` |
| `onDidBuild()` | After `build()` completes | Do NOT change state or call `animateTo()`. API 12+ |
| `aboutToDisappear()` | Before destruction | Do NOT change state (especially `@Link`). No async/await |
| `aboutToReuse(params)` | Reusable component re-added from cache | Update state with new params. API 10+ |
| `aboutToRecycle()` | Component moving to reuse cache | Release heavy resources. API 10+ |

**Only `@Entry`:**

| Callback | When |
|---|---|
| `onPageShow()` | Each time page is displayed |
| `onPageHide()` | Each time page is hidden |
| `onBackPress(): boolean` | User taps Back. Return `true` to override default |

**Execution order (cold start):**
`Parent aboutToAppear → Parent build → Parent onDidBuild → Child aboutToAppear → Child build → Child onDidBuild → onPageShow`

### Layout containers

| Container | When to use | Performance |
|---|---|---|
| `Column` / `Row` | Linear arrangement | **Best** — single-pass layout |
| `Stack` | Overlapping / stacking | Good |
| `Flex` | Items need stretch/shrink | **Slower** — extra pass for flexGrow/flexShrink |
| `RelativeContainer` | Complex layouts, avoid deep nesting | Good — flat structure |
| `GridRow` / `GridCol` | Responsive multi-device grids | Good |
| `List` | Scrollable list with recycling | Best for long lists (with `LazyForEach`) |

**Column/Row alignment:**
- Main axis: `justifyContent(FlexAlign.Start | .Center | .End | .SpaceBetween | .SpaceAround | .SpaceEvenly)`
- Cross axis: Column → `alignItems(HorizontalAlign.Start | .Center | .End)`, Row → `alignItems(VerticalAlign.Top | .Center | .Bottom)`

**Stack:** `alignContent` takes 9 positions (TopStart, Top, TopEnd, Start, Center, End, BottomStart, Bottom, BottomEnd). `zIndex` controls layer order.

**Flex vs Column/Row:** Flex requires re-layout for `flexShrink`/`flexGrow`. Always prefer `Column`/`Row` when you don't need flex behavior.

**RelativeContainer:** Use `__container__` as anchor ID for the container itself. Each child needs `.id()`. Set `.alignRules({ top: { anchor: 'id', align: VerticalAlign.Bottom } })`.

### Performance-critical patterns

**`LazyForEach` for large lists** (only renders visible items):
```ts
class MyDataSource implements IDataSource {
  private data: string[] = []
  totalCount(): number { return this.data.length }
  getData(index: number): string { return this.data[index] }
  registerDataChangeListener(listener: DataChangeListener): void { /* ... */ }
  unregisterDataChangeListener(listener: DataChangeListener): void { /* ... */ }
}

List() {
  LazyForEach(this.dataSource, (item: string, index: number) => {
    ListItem() { Text(item) }
  }, (item: string) => item)  // key generator — MUST produce unique keys
}
.cachedCount(5)  // preload 5 items off-screen
```

**`@Reusable` components** (69% faster component creation):
```ts
@Reusable
@Component
struct MyListItem {
  @State message: Message = new Message('default')

  aboutToReuse(params: Record<string, ESObject>) {
    this.message = params.message as Message
  }
  aboutToRecycle() { /* release heavy resources */ }
  build() { Text(this.message.value) }
}
```
Rules: only works within same parent; don't nest `@Reusable` inside `@Reusable`; combine with `LazyForEach`.

**Layout performance rules:**
- Max 3 levels of nesting — each level adds layout cost
- Use `if/else` over `.visibility()` — hidden components still participate in layout
- Use `RelativeContainer` to flatten deep Row/Column/Flex hierarchies (documented 26% improvement)
- Set explicit dimensions on `List` inside `Scroll` — without them ALL children load at once
- Avoid `@StorageLink` for frequently-changing data — propagates to all subscribers

### Animation

**Explicit animation (`animateTo`)** — state changes in closure animate:
```ts
this.getUIContext()?.animateTo({
  duration: 300,
  curve: Curve.EaseOut,
  onFinish: () => { console.info('done') }
}, () => {
  this.width = 200  // this change animates
})
```

**Property animation (`.animation()`)** — implicit, applied to preceding attributes:
```ts
Text('Hello')
  .width(this.myWidth)
  .animation({ duration: 500, curve: Curve.EaseIn })
```

**Curve values:** `Curve.Linear | .Ease | .EaseIn | .EaseOut | .EaseInOut | .FastOutSlowIn | .Friction | .Sharp | .Smooth`

**Spring curves (string):** `'spring(velocity,mass,stiffness,damping)'`, `'springMotion(response,dampingFraction)'`, `'responsiveSpringMotion(response,dampingFraction)'`

**Shared element transition:**
```ts
// Bind same geometryTransition ID on source and target
Image($r('app.media.photo')).geometryTransition('picture')
// Wrap state change in animateTo
this.getUIContext()?.animateTo({ duration: 300 }, () => { this.isExpanded = !this.isExpanded })
```

### HarmonyOS 6.0 visual effects (沉浸光感视效 / 液态玻璃)

HarmonyOS 6.0 (API 23) introduces system-level "Immersive Light Perception" visual effects. Users enable via Settings → Desktop & Personalization → Immersive Light Effect (强/均衡/弱). Developers achieve similar effects through these ArkUI attributes:

**BlurStyle enum (API 9–11):**

| Name | Since | Level |
|---|---|---|
| `Thin` / `Regular` / `Thick` | API 9 | Material blur |
| `BACKGROUND_THIN` / `BACKGROUND_REGULAR` / `BACKGROUND_THICK` / `BACKGROUND_ULTRA_THICK` | API 10 | Depth-of-field (min→max) |
| `COMPONENT_ULTRA_THIN` / `COMPONENT_THIN` / `COMPONENT_REGULAR` / `COMPONENT_THICK` / `COMPONENT_ULTRA_THICK` | API 11 | Component-level material |
| `NONE` | API 10 | No blur |

**backgroundBlurStyle (API 9+):**
```ts
Column() { /* content */ }
  .backgroundBlurStyle(BlurStyle.Thin, {
    colorMode: ThemeColorMode.LIGHT,     // SYSTEM | LIGHT | DARK
    adaptiveColor: AdaptiveColor.DEFAULT, // DEFAULT | AVERAGE
    scale: 1.0                            // 0.0–1.0 (blur intensity)
  })
```

**foregroundBlurStyle (API 10+):**
```ts
Image($r('app.media.photo'))
  .foregroundBlurStyle(BlurStyle.Regular)
```

**backgroundEffect (API 11+) — fine-grained control:**
```ts
Column() { /* content */ }
  .backgroundEffect({
    radius: 20,          // blur radius
    saturation: 15,      // [0, 50] recommended
    brightness: 0.6,     // [0, 2] recommended
    color: '#80FFFFFF'   // mask color
  })
```

**blur / backdropBlur (API 7+) — numeric radius:**
```ts
Column() { /* content */ }
  .backdropBlur(20, { grayscale: [30, 50] })  // background blur
  .blur(10)                                    // foreground blur
```

**backgroundBrightness (API 12+):**
```ts
Column() { /* content */ }
  .backgroundBrightness({ rate: 0.5, lightUpDegree: 0.2 })
```

**Visual effect filters (API 12+):**
```ts
import { uiEffect } from '@kit.ArkGraphics2D';

const blurFilter = uiEffect.createFilter().blur(10);
Column() { /* content */ }
  .backgroundFilter(blurFilter)    // background filter
  .foregroundFilter(blurFilter)    // content filter
```

**pointLight (API 11+, System API only — NOT available for third-party apps):**
```ts
// System apps only! Supports: Image, Column, Flex, Row, Stack, Button, Toggle
Flex()
  .pointLight({
    lightSource: { positionX: '50%', positionY: '50%', positionZ: 80, intensity: 2, color: Color.White },
    illuminated: IlluminatedType.BORDER,  // NONE | BORDER | CONTENT | BORDER_CONTENT
    bloom: 0.5                            // luminous intensity 0–1
  })
```
Up to 12 light sources can illuminate a single component. HarmonyOS 6.0 adds dual-edge flow light and UV background flow light effects.

**systemMaterialEffect (HDS layer, API 23+, HarmonyOS-only SDK):**
```ts
import { hdsMaterial } from '@kit.ArkUI';

Column() { /* content */ }
  .systemMaterialEffect({
    materialType: hdsMaterial.MaterialType.ADAPTIVE,
    materialLevel: hdsMaterial.MaterialLevel.ADAPTIVE
  })
```
Note: `hdsMaterial` is part of the closed-source HarmonyOS Design System (HDS), not OpenHarmony. Requires HarmonyOS 6.0 SDK (API 23+).

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

**State management performance rules (from official docs):**
- Minimize state scope: only `@State` variables that directly affect UI
- `@Prop` creates **deep copy** on every update — for large objects, prefer `@Link` (by reference) or `@ObjectLink`
- `@Link` is preferred for inter-component communication — avoids unnecessary re-renders
- `@Observed` + `@ObjectLink` for nested objects — fine-grained property observation
- `@ObjectLink` is **READ-ONLY** — cannot reassign whole object (`this.task = new Task()` breaks binding)
- Avoid `@StorageLink` for frequently-changing data — global state changes propagate to ALL subscribers
- **Observation depth (V1):** `@State`/`@Prop`/`@Link` observe ONLY first-level properties. Nested changes are NOT detected. Array: only push/splice/reassign/length, NOT item mutations.

### V2 state decorators (API 12+)

| V1 | V2 replacement | Change |
|---|---|---|
| `@Component` | `@ComponentV2` | Clearer semantics |
| `@State` | `@Local` | Cannot be initialized externally — internal state only |
| `@Observed` + `@ObjectLink` | `@ObservedV2` + `@Trace` | **Deep observation** across multiple nested levels |
| `@Watch` | `@Monitor('prop')` | More precise deep listener |
| `AppStorage` | `AppStorageV2` | Unified with `@ObservedV2` + `@Trace` |

```ts
@ObservedV2
class UserInfo {
  @Trace name: string = '';    // changes to this trigger UI refresh
  @Trace age: number = 0;     // changes to this trigger UI refresh
  address: string = '';        // NO @Trace → changes do NOT trigger refresh
}
```
Rules: `@ObservedV2` and `@Trace` must be used together (either alone has no effect). Only `@Trace`-decorated properties participate in UI rendering.

### StateStore — global state management (2026, officially recommended for mid-large apps)

Separates state logic from UI components entirely. Works with `@ObservedV2` + `@Trace`.

```ts
import { StateStore } from '@kit.ArkUI';

@ObservedV2
class CounterStore {
  @Trace count: number = 0;

  increment(): void {
    this.count++;
  }
}

// Create global store (do this once, e.g. in EntryAbility or top-level)
const counterStore = StateStore.createStore(new CounterStore());

// In any component — read state
@Entry
@Component
struct CounterPage {
  build() {
    Column() {
      Text(`Count: ${counterStore.getState().count}`)
      Button('Add').onClick(() => {
        counterStore.getState().increment();
      })
    }
  }
}
```

**When to use:** Multiple pages/components share the same state; state logic is complex; need thread-safe updates (TaskPool workers can safely update StateStore).

Docs: https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-global-state-management-state-store

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

**NavPathStack full API:**
```ts
// Push
pathStack.pushPath({ name: 'Page', param: data });
pathStack.pushPathByName('Page', data);
pathStack.pushPathByName('Page', data, (popInfo) => {
  console.info('Pop result: ' + JSON.stringify(popInfo.result));
});

// Pop, Replace, Remove
pathStack.pop();
pathStack.replacePath({ name: 'Page', param: data });
pathStack.removeIndex(0);
pathStack.movePageToTop('Page');

// Query
pathStack.getParamByIndex(index);
pathStack.getParamByName('Page');
pathStack.getAllPathName();
pathStack.size();
```

**Route interception:**
```ts
pathStack.setInterception({
  willShow: (from, to, operation) => { /* validate/redirect */ },
  didShow: (from, to, operation) => { /* analytics */ }
});
```

Display modes: **Stack** (single column), **Split** (two columns), **Auto** (adaptive, 600vp threshold).

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

Official samples: https://developer.huawei.com/consumer/cn/samples/

## sys.symbol — icon glyph system

HarmonyOS Symbol is a 1500+ vector icon font with multi-layer color and 7 animation types.

```ts
SymbolGlyph($r('sys.symbol.bell_fill'))
  .fontSize(24)
  .fontColor([Color.Blue, Color.Green])
  .symbolEffect(new BounceSymbolEffect(EffectScope.WHOLE), true)
```

**Confirmed working names** (SDK 6.0.1): `xmark` · `plus` · `minus` · `checkmark` · `chevron_right` · `chevron_left` · `star` · `star_fill` · `bell` · `bell_fill` · `doc` · `video` · `mic` · `mic_fill` · `clock` · `trash` · `pencil` · `camera` · `person`

**Names that do NOT exist** (common mistakes): `photo` · `doc_richtext` · `sparkles` · `checklist` · `image` (use `doc` or `camera` instead) · `location_fill` (use text label instead)

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
11. **Navigation has no `hideSideBar`** — use `.hideBackButton(true)` instead.
12. **`promptAction.showToast()` is deprecated** — use `getUIContext().getPromptAction().showToast(...)` instead; wrap in try-catch for safety.
13. **Floating FAB button blocks last list item** — use `Navigation.menus()` for primary action buttons, or add bottom padding to List equal to FAB height + margin.

## Stability — crash types and error handling

| Type | Description |
|---|---|
| **JS_ERROR** | ArkTS/JS runtime exceptions (most common) — `TypeError: Cannot read property 'x' of undefined` |
| **CPP_CRASH** | Native C/C++ crash (SIGSEGV, SIGABRT) |
| **APP_FREEZE** | Main thread blocked >6s (ANR equivalent). Root causes: thread locks (57%), system resources (14%), heavy main-thread work (9%) |
| **OOM** | Out-of-memory kill |

**Global error handler:**
```ts
import { errorManager } from '@kit.AbilityKit';

const observer: errorManager.ErrorObserver = {
  onUnhandledException(errMsg: string): void {
    console.error('Uncaught: ' + errMsg);
  },
  onException?(errObject: Error): void {  // API 10+
    console.error(errObject.name + ': ' + errObject.message);
  }
};
const observerId = errorManager.on('error', observer);
```

**Crash event subscription (HiAppEvent):**
```ts
import { hiAppEvent } from '@kit.PerformanceAnalysisKit';

hiAppEvent.addWatcher({
  name: "crashWatcher",
  appEventFilters: [{
    domain: hiAppEvent.domain.OS,
    names: [hiAppEvent.event.APP_CRASH]
  }],
  onReceive: (domain, appEventGroups) => { /* process crash */ }
});
```

## Background Tasks — 4 types

| Type | API | Duration | Use case |
|---|---|---|---|
| **Transient** | `requestSuspendDelay` | ~3 min max | Save data, upload logs |
| **Continuous** | `startBackgroundRunning` | Unlimited (needs notification) | Music, navigation, recording |
| **Deferred** | `workScheduler.startWork` | System-determined | Timed sync, cleanup |
| **Agent Reminders** | `ReminderRequestTimer` | System-managed | Alarms, timers |

**Continuous task example:**
```ts
import { backgroundTaskManager } from '@kit.BackgroundTasksKit';
import { wantAgent, WantAgent } from '@kit.AbilityKit';

// module.json5: "abilities": [{ "backgroundModes": ["audioRecording"] }]
// permission: ohos.permission.KEEP_BACKGROUND_RUNNING

const info: wantAgent.WantAgentInfo = {
  wants: [{ bundleName: 'com.example.app', abilityName: 'MainAbility' }],
  actionType: wantAgent.OperationType.START_ABILITY,
  requestCode: 0, actionFlags: [wantAgent.WantAgentFlags.UPDATE_PRESENT_FLAG]
};
const agent = await wantAgent.getWantAgent(info);
backgroundTaskManager.startBackgroundRunning(
  this.context, backgroundTaskManager.BackgroundMode.AUDIO_RECORDING, agent
);
```

9 background modes: `dataTransfer` · `audioPlayback` · `audioRecording` · `location` · `bluetoothInteraction` · `multiDeviceConnection` · `taskKeeping` (2-in-1 only)

**Deferred task frequency by user activity:** Active=2h, Frequent=4h, Regular=24h, Rare=48h, Never used=prohibited.

## Security coding rules (from official best practices)

1. Set `exported: false` for non-interactive abilities
2. Validate all parameters crossing trust boundaries (Want intents, rpc.RemoteObject)
3. Use parameterized queries — never string concat for SQL
4. Replace HTTP with HTTPS; validate SSL certificates
5. Never store personal data in clipboard
6. Use `Asset Store Kit` for sensitive short data (passwords, tokens)
7. Avoid passing personal data through implicit intents
8. Use code obfuscation for production builds
9. Use precise `InputType` (`.USER_NAME`, `.Password`) for system-level protection
10. Never use debug signatures for production releases

**Permission check + request pattern:**
```ts
import { abilityAccessCtrl, bundleManager } from '@kit.AbilityKit';

// Check
const atManager = abilityAccessCtrl.createAtManager();
const bundleInfo = await bundleManager.getBundleInfoForSelf(
  bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION);
const status = await atManager.checkAccessToken(
  bundleInfo.appInfo.accessTokenId, 'ohos.permission.CAMERA');

// Request (if user previously rejected, use requestPermissionOnSetting instead)
atManager.requestPermissionsFromUser(context,
  ['ohos.permission.CAMERA', 'ohos.permission.MICROPHONE']);
```

**Data encryption levels:** EL1 (device-level) → EL2 (user-level, default) → EL3 (accessible while locked) → EL4 (inaccessible when locked)

## Testing — arkxtest framework

Package: `@ohos/hypium` (Mocha-style). Three sub-frameworks: **JsUnit** (unit), **UiTest** (UI automation), **PerfTest** (performance).

### JsUnit

```ts
import { describe, it, expect, beforeAll, beforeEach, afterEach, afterAll } from '@ohos/hypium';

export default function abilityTest() {
  describe('MyTestSuite', () => {
    beforeAll(() => { /* once before all */ });
    beforeEach(() => { /* before each */ });
    afterEach(() => { /* after each */ });
    afterAll(() => { /* once after all */ });

    it('sync_test', 0, () => {
      expect(1 + 1).assertEqual(2);
    });

    it('async_test', 0, async (done: Function) => {
      let result = await someAsyncOp();
      expect(result).assertContain('expected');
      done();
    });
  });
}
```

**Key assertions:** `assertEqual(v)` · `assertContain(v)` · `assertTrue()` · `assertFalse()` · `assertNull()` · `assertUndefined()` · `assertNaN()` · `assertInstanceOf(type)` · `assertThrowError(fn)` · `assertDeepEquals(v)` · `assertClose(v, tolerance)` · `assertLarger(v)` · `assertLess(v)` · `not()` (negation) · `assertPromiseIsResolved()` · `assertPromiseIsRejected()`

Test files in `entry/src/ohosTest/ets/test/`. For UI automation, see the **UiTest** section below.

**仓颉 (Cangjie)** is Huawei's new language (beta) — use ArkTS for all production apps until Cangjie is stable.

## ArkCompiler — runtime details

- **AOT mode** — static type info generates optimized native machine code; no JIT warm-up
- **LiteActor concurrency** — Actor model with isolated memory per thread; communication via message passing

### TaskPool vs Worker

| Feature | TaskPool | Worker |
|---|---|---|
| Thread management | Automatic (create/reuse/destroy) | Manual lifecycle |
| Max threads | Auto-scaled to physical cores | Max 64 per process |
| Task duration limit | **3 minutes** (excluding async I/O) | Unlimited |
| Priority | HIGH / MEDIUM / LOW / IDLE | API 18+ only |
| Cancellation | Supported | Not supported |
| Thread reuse | Yes | No |
| Task groups | Yes | No |
| Delayed/periodic | Yes | No |

**Use Worker when:** tasks exceed 3 minutes, need persistent state, or strongly associated synchronous tasks.

### @Concurrent rules
- **Required** on all TaskPool functions, **only in `.ets` files**
- Allowed: regular functions, async functions
- **Prohibited:** arrow functions, class methods, anonymous functions, generator functions
- **No closure variables** — cannot reference outer scope; only local vars, params, and imports
- Cannot call other same-file functions (closure violation) — must import them

```ts
import { taskpool } from '@kit.ArkTS';

@Concurrent
function heavyCalc(n: number): number { return n * n; }

const result = await taskpool.execute(heavyCalc, 42);

// With priority
const task = new taskpool.Task(heavyCalc, 42);
await taskpool.execute(task, taskpool.Priority.HIGH);

// Delayed and periodic
taskpool.executeDelayed(heavyCalc, 2000, 42);      // after 2s
taskpool.executePeriodically(heavyCalc, 5000, 42);  // every 5s

// TaskGroup — execute multiple tasks as a group
const group = new taskpool.TaskGroup();
group.addTask(heavyCalc, 10);
group.addTask(heavyCalc, 20);
group.addTask(heavyCalc, 30);
const results = await taskpool.execute(group);  // returns array of results
```

**Long-time tasks:** async code (Promise/IO) in TaskPool has NO time limit (only CPU-bound sync code is capped at 3 minutes). HarmonyOS 6.0 officially supports long-running async tasks in TaskPool.

### @Sendable — shared-heap reference passing
Objects on SharedHeap (process-level, all threads can access) — **100x faster** than serialization for 1MB data.

```ts
@Sendable
class SharedData {
  value: number = 0;  // must be explicitly initialized
}
```

**Constraints:** Can only inherit from Sendable classes. Property types limited to: primitives, Sendable classes, `@arkts.collections` containers. Cannot add/delete properties at runtime. No computed properties. No `#` private (use `private` keyword).

### Worker pattern
```ts
// Main thread
const worker = new worker.ThreadWorker('entry/ets/workers/myWorker.ets');
worker.postMessage({ type: 'compute', data: payload });
worker.onmessage = (e) => { /* handle result */ };
worker.terminate();

// Worker thread (myWorker.ets)
const workerPort = worker.workerPort;
workerPort.onmessage = (e) => {
  const result = processData(e.data);
  workerPort.postMessage(result);
};
```

**Both TaskPool and Worker:** Cannot access AppStorage or UI libraries from worker threads. Different thread contexts prevent context object sharing.

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

## Key references

- Official docs: https://developer.huawei.com/consumer/cn/doc/
- Best practices: https://developer.huawei.com/consumer/cn/best-practices
- Samples: https://developer.huawei.com/consumer/cn/samples/
- Codelabs: https://developer.huawei.com/consumer/cn/codelabsPortal/serviceTypes
- ArkTS guide: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts
- ArkUI guide: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui
- State management: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-state-management-overview
- Navigation: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-set-navigation-routing
- Concurrency: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-concurrency
- SDK / Kits: https://developer.huawei.com/consumer/cn/sdk
- OHPM registry: https://developer.huawei.com/consumer/cn/deveco-service
- DevEco Studio: https://developer.huawei.com/consumer/cn/deveco-studio/
- Symbol icons: https://developer.huawei.com/consumer/cn/design/harmonyos-symbol
- AppGallery Connect: https://developer.huawei.com/consumer/cn/agconnect
- StateStore: https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-global-state-management-state-store

## API 21 (SDK 6.0.1) — confirmed compile errors and fixes

These errors were verified against a real Mate 70 Pro build. All entries below caused `ArkTS Compiler Error` at `assembleDevHqf`.

### `DataChangeListener` requires both new and deprecated method names

In API 21, any class that `implements DataChangeListener` must include ALL of:
```ts
// New names (current)
onDataReloaded(): void
onDataAdd(index: number): void
onDataDelete(index: number): void
onDataChange(index: number): void
onDataMove(from: number, to: number): void
// Deprecated aliases — still required by the interface in API 21
onDataAdded(index: number): void
onDataDeleted(index: number): void
onDataChanged(index: number): void
onDataMoved(from: number, to: number): void
onDatasetChange(dataOperations: DataOperation[]): void
```
This applies to every class including test stubs (`NoopListener`, etc).

### `notificationManager.addSlot()` — pass `SlotType`, not `NotificationSlot`

```ts
// ❌ API 9 style
await notificationManager.addSlot({ type: SlotType.SOCIAL_COMMUNICATION, desc: '...' })
// ✅ API 12+ style
await notificationManager.addSlot(notificationManager.SlotType.SOCIAL_COMMUNICATION)
```

Also: always annotate the constant explicitly to prevent type narrowing to literal type:
```ts
// ❌ infers as SlotType.SOCIAL_COMMUNICATION — not assignable to SlotType
const SLOT_ID = notificationManager.SlotType.SOCIAL_COMMUNICATION
// ✅
const SLOT_ID: notificationManager.SlotType = notificationManager.SlotType.SOCIAL_COMMUNICATION
```

### `NotificationRequest.slotType` — incompatible `SlotType` modules

`NotificationRequest.slotType` is typed as `@ohos.notification.SlotType` (old module), while `notificationManager.SlotType` is from `@ohos.notificationManager` (new module). They are nominally different types — assigning new to old is a compile error. **Just omit `slotType`** (it is optional):
```ts
const request: notificationManager.NotificationRequest = {
  id: NOTIFICATION_ID,
  // slotType omitted — routes to the slot created by addSlot() automatically
  content: { ... }
}
```
`NotificationSlotLevel` does not exist on `notificationManager` in API 21 — remove any usage.

### `Permissions` type — `@ohos.bundleManager` not importable

`import type { Permissions } from '@ohos.bundleManager'` fails with "Cannot find module". `bundleManager` is accessible only via `@kit.AbilityKit`. Workaround: declare a local subset type whose values are all valid `Permissions` literals:
```ts
// ✅ subtype of Permissions — assignable to the SDK's Permissions parameter
type AppPermission = 'ohos.permission.CAMERA' | 'ohos.permission.APPROXIMATELY_LOCATION'

async function hasPermission(name: AppPermission): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager()
  const bundleInfo = await bundleManager.getBundleInfoForSelf(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION)
  const status = atManager.checkAccessTokenSync(bundleInfo.appInfo.accessTokenId, name)
  return status === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
}
```

### `requestPermissionsFromUser` — `void & Promise` overload intersection

The function has both a callback overload (returns `void`) and a Promise overload (returns `Promise<PermissionRequestResult>`). TypeScript sometimes intersects them to `void & Promise<…>`, causing `.authResults` to not exist. Additionally, `abilityAccessCtrl.PermissionRequestResult` is **not exported** from the namespace in API 21.

Fix: declare the result interface locally and cast:
```ts
interface PermResult {
  authResults: abilityAccessCtrl.GrantStatus[]
}

async function requestPermission(ctx: common.UIAbilityContext, name: AppPermission): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager()
  const result = await (atManager.requestPermissionsFromUser(ctx, [name]) as Promise<PermResult>)
  return result.authResults[0] === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
}
```

### `getContext(this)` is deprecated

Replace in all component methods:
```ts
// ❌
const ctx = getContext(this) as common.UIAbilityContext
// ✅
const ctx = this.getUIContext().getHostContext() as common.UIAbilityContext
```
Module-level (non-component) functions must receive `ctx` as a parameter instead.

## Multi-device / foldable screen adaptation (API 21)

### Breakpoint detection

```ts
import { display } from '@kit.ArkUI'

export const BP_SM = 'sm'   // < 600 vp  — phone portrait
export const BP_MD = 'md'   // < 840 vp  — phone landscape / small tablet
export const BP_LG = 'lg'   // >= 840 vp — large tablet / foldable unfolded

export function getBreakpoint(): string {
  try {
    const d = display.getDefaultDisplaySync()
    const widthVp = d.width / d.densityPixels
    if (widthVp < 600) return BP_SM
    if (widthVp < 840) return BP_MD
    return BP_LG
  } catch (e) {
    return BP_SM
  }
}
```

### Foldable screen — listen for fold/unfold events

```ts
// display.on callback receives (id: number) — NOT empty params
private displayListener: (id: number) => void = (_id: number) => {
  this.breakpoint = getBreakpoint()
}

aboutToAppear(): void {
  this.breakpoint = getBreakpoint()
  display.on('change', this.displayListener)
}

aboutToDisappear(): void {
  display.off('change', this.displayListener)
}
```

### Responsive layout switching (if/else, not .visibility)

Always use `if/else` to switch between phone and tablet layouts. `.visibility(Visibility.None)` hides but still lays out — wastes resources and can cause measurement bugs.

```ts
if (this.breakpoint === BP_SM) {
  // Phone: single-column List + LazyForEach
  List({ space: 8 }) { LazyForEach(...) }
} else {
  // Tablet/foldable: GridRow with responsive columns
  Scroll() {
    GridRow({
      columns: { sm: 1, md: 2, lg: 3 },
      gutter: { x: 12, y: 12 },
      breakpoints: { value: ['600vp', '840vp'] }
    }) {
      ForEach(this.gridItems, (item) => { GridCol() { MyCard({ record: item }) } })
    }
  }
}
```

### `GridRow`/`GridCol` does not support `LazyForEach`

`GridRow`/`GridCol` is a responsive layout container, not a lazy loader. Use `ForEach` inside it. To keep the grid reactive to data changes, maintain a `@State gridItems: T[]` that is updated via a `DataChangeListener`:

```ts
class GridRefreshListener implements DataChangeListener {
  private updateFn: () => void
  constructor(fn: () => void) { this.updateFn = fn }
  // Must implement ALL methods — see API 21 DataChangeListener note above
  onDataReloaded(): void { this.updateFn() }
  onDataAdd(_i: number): void { this.updateFn() }
  onDataAdded(_i: number): void { this.updateFn() }
  // ... all other methods
  onDataChange(_i: number): void { /* @ObjectLink handles per-item updates */ }
  onDataChanged(_i: number): void { }
}

// In @Entry @Component:
@State gridItems: MyRecord[] = []
private gridListener = new GridRefreshListener(() => {
  this.gridItems = [...this.dataSource.getAllData()]
})
aboutToAppear() { this.dataSource.registerDataChangeListener(this.gridListener) }
aboutToDisappear() { this.dataSource.unregisterDataChangeListener(this.gridListener) }
```

### Share breakpoint via `@Provide`/`@Consume`

Declare in the `@Entry` root, consume in any `NavDestination` child:
```ts
// Root
@Provide('breakpoint') breakpoint: string = BP_SM

// Detail page (NavDestination)
@Consume('breakpoint') breakpoint: string
```
`@Provide`/`@Consume` works across `Navigation`/`NavDestination` because they are in the same component subtree.

## Location Kit (geoLocationManager)

```ts
import { geoLocationManager } from '@kit.LocationKit'

// Permission required: ohos.permission.APPROXIMATELY_LOCATION (user_grant)
// module.json5: reason + usedScene required

const pos = await geoLocationManager.getCurrentLocation({
  priority: geoLocationManager.LocationRequestPriority.FIRST_FIX,
  scenario: geoLocationManager.LocationRequestScenario.UNSET,
  timeoutMs: 10000
})

const addresses = await geoLocationManager.getAddressesFromLocation({
  latitude: pos.latitude,
  longitude: pos.longitude,
  maxItems: 1
})

// Build readable location string from GeoAddress fields:
// administrativeArea (省) → subAdministrativeArea (市) → locality (区) → subLocality → placeName
```

## Notification Kit (notificationManager)

```ts
import { notificationManager } from '@kit.NotificationKit'

// 1. Create slot (call once at app init)
await notificationManager.addSlot(notificationManager.SlotType.SOCIAL_COMMUNICATION)

// 2. Check and request notification enable
const enabled = await notificationManager.isNotificationEnabled()
if (!enabled) await notificationManager.requestEnableNotification()  // deprecated but functional

// 3. Publish
const request: notificationManager.NotificationRequest = {
  id: 1001,
  // OMIT slotType — see API 21 type mismatch note above
  content: {
    notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
    normal: { title: '标题', text: '正文', additionalText: '副文本' }
  },
  deliveryTime: Date.now(),
  showDeliveryTime: true,
  tapDismissed: true
}
await notificationManager.publish(request)

// 4. Cancel
await notificationManager.cancel(1001)
```

## relationalStore (SQLite)

```ts
import { relationalStore } from '@kit.ArkData'
import { common } from '@kit.AbilityKit'

// Init (call in aboutToAppear / UIAbility.onCreate)
const store = await relationalStore.getRdbStore(ctx, {
  name: 'my_db.db',
  securityLevel: relationalStore.SecurityLevel.S1
})
await store.executeSql('CREATE TABLE IF NOT EXISTS records (...)')

// Insert
const values: relationalStore.ValuesBucket = { id: '1', title: 'Hello' }
await store.insert('records', values)

// Query (ordered)
const predicates = new relationalStore.RdbPredicates('records')
predicates.orderByDesc('timestamp')
const rs = await store.query(predicates, ['id', 'title', 'timestamp'])
while (rs.goToNextRow()) {
  const id = rs.getString(rs.getColumnIndex('id'))
}
rs.close()

// Update
const predicates2 = new relationalStore.RdbPredicates('records')
predicates2.equalTo('id', '1')
await store.update({ title: 'Updated' }, predicates2)

// Delete
const predicates3 = new relationalStore.RdbPredicates('records')
predicates3.equalTo('id', '1')
await store.delete(predicates3)
```

## preferences (key-value settings)

```ts
import { preferences } from '@kit.ArkData'

const store = await preferences.getPreferences(ctx, 'user_settings')
// Read (with default)
const val = (await store.get('sort_order', 'time_desc')) as string
// Write + flush
await store.put('sort_order', 'time_asc')
await store.flush()
```

## UiTest — common patterns (arkxtest)

```ts
import { Driver, ON, Component } from '@ohos.UiTest'
import AbilityDelegatorRegistry from '@ohos.app.ability.abilityDelegatorRegistry'

const DELEGATOR = AbilityDelegatorRegistry.getAbilityDelegator()

// Launch app before tests
beforeAll(async (done: Function) => {
  await DELEGATOR.startAbility({ bundleName: 'com.example.app', abilityName: 'EntryAbility' })
  await new Promise(r => setTimeout(r, 2000))  // wait for UI to render
  done()
})

it('example', 0, async (done: Function) => {
  const driver = Driver.create()
  
  // Find by text / type / content description
  const btn = await driver.findComponent(ON.text('发布'))
  const input = await driver.findComponent(ON.type('TextInput'))
  const items = await driver.findComponents(ON.type('SymbolGlyph'))
  
  // Actions — ALL must be awaited
  await btn.click()
  await input.inputText('hello')
  
  // Wait after state-changing actions
  await new Promise(r => setTimeout(r, 800))
  
  // Component refs become stale after state changes — re-find
  const btnAfter = await driver.findComponent(ON.text('发布'))
  
  // Get bounds for position-based selection
  const bounds = await btn.getBounds()  // { top, left, right, bottom }
  
  // Navigate back
  await driver.pressBack()
  
  done()
})
```

Key rules:
- Every UiTest API call inside `it()` must be `await`ed
- After any click/input that changes state, component references may be stale — always re-`findComponent`
- Tests only run on real device or emulator — **Previewer does not support UiTest**
- `Driver.create()` fresh per `it` block (don't share across tests)
- Use `sleep` / `setTimeout` after navigation to let the new page render before asserting
