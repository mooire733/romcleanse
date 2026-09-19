# 未知厂商 / 未知包的处置（证据驱动 triage）

> 适用：ROM 签名落空、或包不在任何厂商 reference 表里。原则：**证据不足的包只提案，不执行**。

## 1. ROM 兜底识别

```
adb shell getprop | grep -Ei 'product\.brand|product\.manufacturer|build\.display\.id'
```
brand 落到哪家就用哪家 reference 的 NEVER 清单 + 保守模式；brand 也不明 → 只用通用 AOSP 手段。

## 2. 包分类打分信号（按强度排序）

### 信号 A：doze 白名单自加白（强证据，实测广告框架特征）
```
adb shell dumpsys deviceidle whitelist | grep '^user'
```
非系统包出现在 user 段 = 该应用给自己争取了后台唤醒特权。广告/追踪框架的典型行为。命中 → T1 候选。

### 信号 B：包名关键词（中证据）
```
grep -iE '\.ad|\.ads|advert|adsdk|analytics|track|push|mcs|midas|market|appstore|gamecenter|game|hybrid|pictorial|magazine|wallet|clean|news|browser|theme|ugd|ab$|ota|report|logger|stats'
```
命中关键词 ≠ 一定是广告（browser/weather/news 可能是用户在用的功能件）→ 降为「T2 候选，提案时说明用途猜测」。

### 信号 C：installer 直方图（中证据，识别预装来源）
```
adb shell pm list packages -3 -i | grep -oE 'installer=[^ ]*' | sort | uniq -c | sort -rn
```
installer 为厂商商店/换机助手（如 com.miui.huanji）的 -3 包 = 预装垃圾链。installer=null 的 -3 包多为出厂预置，逐个列出给用户看。

### 信号 D：运行时行为（弱证据）
- 注册 BOOT_COMPLETED：`cmd package query-receivers --brief -a android.intent.action.BOOT_COMPLETED`
- 持有 POST_NOTIFICATIONS 且通知权限开启、持有 SYSTEM_ALERT_WINDOW
- 持有 enabled_notification_listeners（能读所有通知，重点核查）

### 信号 E：APK 标签（可选，重操作）
```
adb shell pm path <pkg>          # 取 APK 路径
adb pull <path> /tmp/…           # 拉到 Mac
aapt2 dump badging /tmp/….apk | grep application-label
```
仅在提案需要「人话名字」且信号不足时使用。

## 3. 分层与提案规则

- A+B/C 同时命中 → T1 候选（删除提案）
- 仅 B 或仅 C 命中 → T2/T3 候选（硬化或卸载提案，说明置信度）
- 零命中 + 名字不可读 → **不进提案**，报告标「未知，未处理」
- 所有提案表必须带：包名、信号证据、置信度、建议动作（卸载/禁用/硬化/不动）

## 4. 执行中的保守约束

- 未知厂商上禁用「批量」：每包单独执行、单独验证（HOME + systemui + dropbox）。
- 遇到任何 SecurityException 且 disable-user 也失败 → 记 skipped，立即停止该厂商的删除类操作，只保留通知/后台硬化。
- 用户确认环节必须显式列出「证据不足」的包，让用户逐个拍板。
