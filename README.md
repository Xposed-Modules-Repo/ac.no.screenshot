# 隐屏 / HideScreen

[English](#english) | [中文](#chinese)

源码：https://github.com/zjkkzk/-HideScreen
Source:https://github.com/zjkkzk/-HideScreen
---

<a id="chinese"></a>
## 中文说明

### 简介

**隐屏（HideScreen）** 是一个基于 Xposed / LSPosed 的轻量级 Android 模块。  
它在截图、录屏及屏幕投射过程中，使作用域内的应用界面**从最终画面中消失**，而应用本身在设备上的显示和交互完全不受影响。

模块的实现完全依赖 Android 原生安全机制 —— 通过设置 `SurfaceControl.setSkipScreenshot(true)`，让系统合成画面时自动跳过指定窗口。模块本身**不修改应用内容、不注入视图、不持续运行**，仅在窗口创建时一次性标记，性能极佳，耗电可忽略不计。

<div align="center">
  <img src="/app/1.jpg" width="45%" alt="截图效果 - 1" />
  <img src="/app/2.jpg" width="45%" alt="截图效果 - 2" />
</div>

---

### 功能特点

- ✅ 截图、录屏、投射结果中完全隐藏目标应用界面
- ✅ 应用本地显示与交互保持原样，不受任何影响  
- ~~✅ 不影响物理显示输出（USB / HDMI 投射画面正常显示）~~  
- ✅ 无需任何配置、无后台服务、无用户界面  
- ✅ 极简设计：一次设置，永久生效  
- ✅ 兼容 Android 6.0 ～ 15+，覆盖绝大多数定制 ROM  
- ✅ 适配所有通过 `WindowManager` 添加的窗口（包括输入法、对话框、Toast 等）

---

### 技术原理与优化

本模块的设计哲学为：**只做一件事，做到极致**。

- **核心机制**  
  在 `ViewRootImpl` 窗口生命周期的关键节点，通过反射调用系统隐藏 API `SurfaceControl.Transaction.setSkipScreenshot(true)`，将防截屏标志写入 SurfaceFlinger。系统截图与录屏服务会自动遵守该标志，无需额外拦截或修改系统行为。

- **关键优化**  
  - **反射预热与缓存**：所有反射成员仅在首次使用时加载一次，后续直接复用，避免重复查找带来的性能波动。  
  - **Surface 字段智能缓存**：自动探测设备上实际存放 `SurfaceControl` 的字段并记住，后续窗口直接命中，彻底消除无效的反射遍历。  
  - **事务提交去重**：每个窗口的生命周期内，防截屏标志只会被设置一次，即便每帧回调也不会重复提交 Binder 事务，将额外开销降至理论最低。  
  - **多版本 API 降级链**：按优先级尝试不同签名的 `setSkipScreenshot` 及 `setSecure` 方法，确保从 Android 6.0 到最新版本的广泛兼容性。  
  - **资源安全释放**：事务完成后主动调用 `close()` 方法，及时释放底层 Binder 引用，避免任何潜在的系统资源堆积。  
  - **弱引用容器**：窗口标记全部存储在弱引用集合中，当窗口被系统销毁时，相关记录自动清理，无需手动维护，无内存泄漏风险。  
  - **线程安全与防重入**：采用合适的同步包装及保护机制，即使在多线程回调的场景下也能保持稳定运行。

- **性能与功耗**  
  模块只在窗口创建、旋转或重建时进行极短的事务提交，其余时间完全静默。在日常使用中，其对 CPU 和电池的影响微乎其微，完全可以忽略不计。即便在低端设备上，也不会产生任何可感知的卡顿或发热。

---

### 使用环境

- Android 6.0 ～ 15+（实际效果取决于系统实现）  
- 已激活的 Xposed 框架（原版、EdXposed、LSPosed 均可）  
- 模块仅作用于被勾选的第三方应用

---

### 使用方法

1. 安装模块 APK  
2. 在 LSPosed 管理器中启用模块  
3. **仅勾选需要隐藏界面内容的应用**  
4. 强制停止并重新打开目标应用

启用后，被选中的应用在进行系统截图、录屏或屏幕投射时，其界面将不出现在结果中。

---

### ⚠️ 重要警告

**切勿将本模块作用于以下目标：**

- 系统界面（System UI）  
- 系统框架（android）或核心服务（system_server）  
- 任何系统级应用（如设置、电话、启动器等）

若错误地将模块应用于系统组件，可能导致：

- 系统截图、录屏功能彻底失效  
- 界面黑屏、无法唤醒或持续卡在开机动画  
- 系统不稳定甚至无法启动

**本模块仅设计用于第三方普通应用，请务必在 LSPosed 作用域中仔细甄别。**

---

### 注意事项

- 本项目主要用于技术研究、隐私保护及合规测试场景。  
- 由于各厂商 ROM 对 `setSkipScreenshot` 的支持程度不同，模块在某些深度定制的系统上可能效果受限（已知问题极少）。  
- 使用者应遵守当地法律法规，不得将本模块用于非法用途。  
- 使用本模块所造成的任何后果由使用者自行承担。

---

### 致谢

本模块的核心思路受 [Transparent Screenshot](https://github.com/Dszsu/Transparent_screenshot) 启发，在此向原作者的开源精神表示感谢。  
当前版本已在性能、兼容性、资源管理等方面进行了彻底重构，具体差异详见[技术原理与优化](#技术原理与优化)章节。

---

### 开源许可

本项目基于 [MIT License](LICENSE) 发布。

---

<a id="english"></a>
## English

###概述

**HideScreen** is a lightweight Xposed / LSPosed module for Android.  
It hides the target application's UI from screenshots, screen recordings, and screen casting results, while the app remains fully visible and interactive on the physical device.

The module leverages Android's native secure surface flag — `SurfaceControl.setSkipScreenshot(true)` — to instruct SurfaceFlinger to exclude specific windows from the final composed output. No content modification, view injection, or background service is involved. Performance is excellent, and power consumption is negligible.

<div align="center">
  <img src="/app/1.jpg" width="45%" alt="Screenshot 1" />
  <img src="/app/2.jpg" width="45%" alt="Screenshot 2" />
</div>

---

### Features

- ✅ Hides app UI from screenshots, recordings, and casting results  
- ✅ No impact on local display or touch interaction  
- ✅ Physical display output (USB / HDMI) remains unaffected  
- ✅ Zero configuration, no UI, no background services  
- ✅ Minimalistic design: one‑time flag, permanent effect  
- ✅ Supports Android 6.0 through 15+, works on most custom OEM ROMs  
- ✅ Covers all windows added via `WindowManager` (including IME, dialogs, toasts)

---

### Technical Highlights & Optimizations

The module follows a single‑responsibility principle: **do one thing exceptionally well**.

- **Core Approach**  
  Hooks into `ViewRootImpl` lifecycle methods (`setView`, `relayoutWindow`, `performTraversals`) and applies the screenshot‑skip flag via reflective calls to `SurfaceControl.Transaction`. The system screenshot / screen recording pipeline respects this flag automatically.

- **Key Optimizations**  
  - **Reflection pre‑warming & caching** – All reflection members are resolved once and reused, eliminating repeated lookup overhead.  
  - **Intelligent Surface field caching** – The actual `SurfaceControl` field used by the device is detected and memorized, avoiding unnecessary reflection traversal.  
  - **Transaction deduplication** – Each window is marked only once per lifecycle, preventing redundant Binder IPC calls and reducing overhead to the absolute minimum.  
  - **Multi‑version API fallback chain** – Attempts different signatures of `setSkipScreenshot` and `setSecure` in optimal order, ensuring broad compatibility from Android 6.0 to the latest versions.  
  - **Resource cleanup** – Actively calls `close()` on the transaction object to release underlying Binder references promptly.  
  - **Weak reference containers** – Window markers are stored in weak‑reference sets, allowing automatic cleanup when windows are destroyed with zero memory leaks.  
  - **Thread safety & reentrancy guard** – Proper synchronization and protective mechanisms ensure stability even under concurrent callbacks.

- **Performance & Power**  
  The module performs a brief transaction only when a window is created, rotated, or rebuilt. The rest of the time it stays completely idle. Its impact on CPU and battery is so minimal that it is entirely unnoticeable in daily use, even on low‑end devices.

---

### Requirements

- Android 6.0 – 15+ (actual behavior depends on system implementation)  
- An active Xposed framework (original, EdXposed, or LSPosed)  
- The module scope should be limited to third‑party applications only

---

### Usage

1. Install the module APK  
2. Enable the module in LSPosed Manager  
3. **Select only the third‑party apps** you wish to hide from screenshots / recordings  
4. Force‑stop and relaunch the target apps

Once enabled, selected apps will no longer appear in system screenshots, screen recordings, or casting outputs.

---

### ⚠️ Important Warning

**Do NOT apply this module to the following:**

- System UI  
- Android framework (`android`) or `system_server`  
- Any system‑level packages (Settings, Phone, Launcher, etc.)

Applying the module to system components may cause:

- Complete breakage of screenshot / recording functionality  
- Black screens, unresponsive UI, or boot loops  
- System instability or failure to start

This module is **strictly designed for ordinary third‑party applications**. Choose scopes carefully within LSPosed.

---

### Disclaimer

- This project is intended for technical research, privacy protection, and compliance testing.  
- Effectiveness may vary on heavily customized OEM ROMs due to differing support for `setSkipScreenshot`.  
- Users are responsible for complying with local laws and regulations.  
- The author assumes no liability for any consequences resulting from the use of this module.

---

### Acknowledgments

The core idea of this module was inspired by [Transparent Screenshot](https://github.com/Dszsu/Transparent_screenshot). We appreciate the original author's open-source spirit.  
The current version has been completely refactored in terms of performance, compatibility, and resource management. See [Technical Highlights](#technical-highlights--optimizations) for detailed differences.

---

### License

This project is licensed under the [MIT License](LICENSE).
