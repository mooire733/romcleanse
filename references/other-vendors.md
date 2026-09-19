# 其他厂商（魅族 Flyme · 中兴/努比亚 · 联想等）薄表

> ⚠️ 状态：未实机验证。清单极薄，以 unknown-rom-triage 证据流程为主，本表仅作起点。

## 魅族 Flyme

- 检出：`ro.build.display.id` 含 "Flyme"
- T2 候选：com.meizu.mstore（应用商店）、com.meizu.media.video、com.meizu.media.music、com.meizu.gamecenter、com.flyme.trade（以实机为准）
- NEVER：com.android.launcher（Flyme 桌面）、com.meizu.flyme.service（核心服务）、com.meizu.net.pushservice（推送）、com.meizu.account（账号）、使用中的输入法

## 中兴 / 努比亚

- 检出：`ro.product.brand` 含 ZTE / nubia；可能存在 com.zte.* / com.nubia.* 命名空间
- T2 候选：com.zte.appmarket / com.nubia.appstore（商店）、com.zte.gamebox、nubia 自带视频音乐（以实机枚举为准）
- NEVER：桌面、com.zte.systemmanager 类管家、推送与账号组件、使用中的输入法

## 通用规则

1. 这些厂商的包清单不完整，**一律走「枚举 → 打分 → 提案 → 确认」**，不假设任何包的作用。
2. 打分信号按 unknown-rom-triage.md 执行（包名关键词 / doze 自加白 / installer 直方图）。
3. 任何「名字看不出来、信号也不足」的包 → 不进提案表，报告里标「未知，未处理」。
