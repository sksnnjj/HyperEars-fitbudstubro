# HyperEars 适配补丁 —— Edifier FitBuds Turbo

> 将文件放置到完整仓库后，替换/新增对应源文件即可。所有补丁均在现有 HyperEars main 分支
> （最新提交 2026-08-31，v2.6.0 之后）基础上编写。

## 改动清单

| 文件 | 操作 | 说明 |
|------|------|------|
| `integration/src/main/java/dev/hyperears/integration/EdifierFitBudsTurboAdapter.kt` | **新增** | 具体型号适配器 |
| `integration/src/main/java/dev/hyperears/integration/EarbudAdapter.kt` | **修改** | 在 `initialRegistrations` 注册一行 |
| `integration/src/test/java/dev/hyperears/integration/EdifierFitBudsTurboAdapterTest.kt` | **新增** | 单元测试（可选） |

---

## 1. 新增适配器

把 `EdifierFitBudsTurboAdapter.kt` 放到：
```
integration/src/main/java/dev/hyperears/integration/EdifierFitBudsTurboAdapter.kt
```

## 2. 注册到适配器注册表

编辑 `integration/src/main/java/dev/hyperears/integration/EarbudAdapter.kt`，
在 `initialRegistrations` 的 Edifier 组中插入一行。

**改动前：**
```kotlin
add(Registration(edifierGroup, ::EdifierW860NBProAdapter))
add(Registration(edifierGroup, ::EdifierEvoProAdapter))
add(Registration(edifierGroup, ::EdifierFitClipUltraAdapter))
add(Registration(edifierGroup, ::EdifierHeadphonesAdapter))
add(Registration(edifierGroup, ::EdifierEarbudAdapter))
```

**改动后：**
```kotlin
add(Registration(edifierGroup, ::EdifierW860NBProAdapter))
add(Registration(edifierGroup, ::EdifierEvoProAdapter))
add(Registration(edifierGroup, ::EdifierFitClipUltraAdapter))
add(Registration(edifierGroup, ::EdifierFitBudsTurboAdapter))   // <-- 新增（精确匹配优先于家族）
add(Registration(edifierGroup, ::EdifierHeadphonesAdapter))
add(Registration(edifierGroup, ::EdifierEarbudAdapter))
```

> 注册顺序很重要：`EdifierFitBudsTurboAdapter` 是 `EXACT_MATCH`，必须排在
> `EdifierHeadphonesAdapter` / `EdifierEarbudAdapter`（`FAMILY_MATCH`）**之前**，
> 否则 `EdifierEarbudAdapter` 的 `matches()` 会先命中家族匹配，精确型号永不生效。

## 3. （可选）单元测试

参考 `EdifierEarbudAdapter` 家族已有测试（如 `EarbudAdapterHierarchyTest`），验证：
- 设备名 `Edifier FitBuds Turbo` → 命中 `EdifierFitBudsTurboAdapter`（ID `edifier-fitbuds-turbo`）
- 设备名 `Edifier FitClip Ultra` → **不**命中 FitBuds Turbo（命中 FitClip Ultra）
- 未命名的普通 Edifier 耳机 → 回退到家族匹配

---

## 关于游戏模式的 MiLink 卡片

FitBuds Turbo **同时具备 ANC 和游戏模式**，这在当前代码库的 Edifier 家族里没有先例：

- `FitClipUltraGameModeMiLinkCardAdapter` 用 GAME_MODE 卡片（因为它**无 ANC**，游戏模式正好占用
  ANC 卡片槽位）。
- `EdifierFourModeMiLinkCardAdapter` 用 FOUR_MODE 卡片（四模式：降噪/关闭/通透/抗风噪）。

FitBuds Turbo 的首选呈现是 **FOUR_MODE 四模式卡**（覆盖其核心卖点）。游戏模式的 `SetGameMode`
控制和 `0x08/0x09` 协议链路已经包含在本适配器内，但**是否需要/如何在同一卡片上叠加游戏模式行，
属于 MiLink 桥接层（`system-module/hook`）的 UI 设计问题**，必须在实机验证 ANC 后另行评估。

实机验证时请确认：ANC 四模式卡正常工作后，游戏模式开关是否需要在卡片上单独呈现。
