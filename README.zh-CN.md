# romcleanse

[English](README.md) | [简体中文](README.zh-CN.md)

> 🇨🇳 国产安卓去广告与后台治理 · 一个给 AI Agent 用的 skill
> An [OpenCode](https://github.com/sst/opencode) skill for de-advertising, de-bloating and background tuning on Chinese-vendor Android phones — driven by adb, no root required.

## 这是什么

`romcleanse` 是一个 [OpenCode skill](https://opencode.ai)：让 AI Agent 通过 adb，对你的国产安卓手机做系统级广告清理——卸载广告追踪框架、关预装推广、封锁后台自启、踢出后台唤醒白名单、顺带做一次隐私/监控审计。

**一句话**：把「广告全家桶」从投放、追踪、自启、唤醒特权、通知五个层面全部切断——系统分区改动全程一键可逆，/data 第三方预装逐行标注恢复方式。

## 支持的机型

| 厂商 / 系统 | 状态 |
|---|---|
| 小米 HyperOS / MIUI | ✅ 实机验证（HyperOS 1.0 与 3.0） |
| OPPO / 一加 / realme（ColorOS） | ⚠️ 未实机验证，走证据流程 |
| vivo / iQOO（OriginOS / Funtouch） | ⚠️ |
| 荣耀（MagicOS） | ⚠️ |
| 华为（EMUI / HarmonyOS 2–4） | ⚠️（adb 受限时降级为手动指引） |
| 魅族 Flyme / 中兴 / 努比亚 | ⚠️（薄表 + 证据流程） |
| **HarmonyOS NEXT（纯血）** | 🚫 无 adb，检出即中止 |

未命中厂商签名时自动进入 `unknown-rom-triage` 保守模式：先枚举证据再提案，删除类操作必须用户逐个确认。

## 工作原理

romcleanse 不绑定部署形态，两种环境都能跑：

**形态 A：直连**（最常见）——手机 USB 直接连着跑 Agent/Harness 的电脑：

```
AI Agent / Harness（本机）
   │  adb
   ▼
手机（USB 调试已授权）
```

**形态 B：远程隧道**——Agent 在远程服务器，手机接在用户的 Mac 上：

```
AI Agent（远程服务器）
   │  sshpass ssh（隧道，如 IDEA Gateway 反向转发 :2233）
   ▼
你的 Mac（adb platform-tools 所在地）
   │  adb -s <serial>
   ▼
手机（USB 调试已授权）
```

所有操作均为 **user 0 级**（`pm uninstall -k --user 0`）：APK 与数据保留在系统分区，系统分区预装的改动一条命令即可完整恢复，不 root、不动 /system（/data 分区的第三方预装卸载后需重装，提案时逐行标注）。

## 核心手法

| 目标 | 手段 | 降级路径 |
|---|---|---|
| 卸载广告/追踪框架、预装 | `pm uninstall -k --user 0 <pkg>` | 任何失败 → `pm disable-user --user 0` |
| 关通知 | `pm revoke android.permission.POST_NOTIFICATIONS` | 系统固定权限 → `appops POST_NOTIFICATION deny` → 仍压不住则标「仅可手动关」 |
| 禁自启动 | `appops RUN_ANY_IN_BACKGROUND ignore`（有效层） | + `am set-standby-bucket restricted`（可能被厂商电源守护改回，不算失败） |
| 踢后台唤醒白名单 | `dumpsys deviceidle whitelist -<pkg>` | 广告框架常自加白，逐个踢出 |

## 安全设计

- **改前必快照**：全量包列表 + 已禁用清单落盘，恢复依据随报告交付
- **输入法保底**：自动识别默认输入法与全部启用输入法，永不触碰（防锁屏打不了字）
- **每批验活**：默认桌面解析与基线逐字符比对、SystemUI 存活、dropbox 崩溃时间戳对比基线，异常立即停手回滚
- **NEVER 清单**：桌面、SystemUI、通知框架、安全内核、查找设备、账号云服务、活跃 device-admin……绝不提案
- **改动台账**：每个动作记录逆操作命令，完整回滚是脚本化 loop，不靠记忆
- **唯一交互点**：分级提案（T1 广告框架 / T2 推广应用 / T3 第三方预装）确认后执行；带限定语的行逐项确认
- **命令白名单**：`pm clear`（清用户数据）等破坏性捷径被 skill 自身规则禁止

## 使用

把本目录放进 OpenCode skills 目录：

```bash
git clone https://github.com/mooire733/romcleanse ~/.config/opencode/skills/romcleanse
```

重启 OpenCode 后对 Agent 说：

- 「帮我给手机去广告 / 卸载预装」
- 「关掉这些应用的推送和自启动」
- 「查一下我手机有没有被监控」

Agent 会自动：识别 ROM → 安全快照 → 分级提案 →（你确认）→ 执行与硬化 → 出验证报告和恢复命令。

## 目录结构

```
romcleanse/
├── SKILL.md                      # 主流程：识别 → 快照 → 分级 → 执行 → 验证 → 恢复
├── README.md                     # 英文说明
├── README.zh-CN.md               # 本文件（简体中文）
└── references/
    ├── xiaomi-hyperos.md         # ✅ 小米/红米 完整包分类表（实机验证）
    ├── coloros-oplus.md          # OPPO/一加/realme
    ├── vivo-originos.md          # vivo/iQOO
    ├── honor-magicos.md          # 荣耀
    ├── huawei-emui-harmonyos.md  # 华为 + 纯血鸿蒙中止路径
    ├── other-vendors.md          # 魅族/中兴/努比亚
    ├── unknown-rom-triage.md     # 未知厂商/未知包的证据驱动分类法
    └── privacy-audit.md          # 监控/间谍软件审计（按需加载）
```

## 恢复速查

```bash
adb shell pm install-existing <pkg>                       # 撤销卸载
adb shell pm enable <pkg>                                 # 撤销禁用（恢复为出厂版本）
adb shell appops set <pkg> POST_NOTIFICATION default      # 恢复通知
adb shell appops set <pkg> RUN_ANY_IN_BACKGROUND allow    # 恢复后台启动
adb shell am set-standby-bucket <pkg> active              # 恢复待机档
adb shell dumpsys deviceidle whitelist +<pkg>             # 加回唤醒白名单
```

## 边界与免责

- 非 root 方案的天然边界：系统固定权限（部分管家的通知开关）、应用内广告设置（如商店推荐流）只能手动关，skill 会如实标注
- 审计无法覆盖 Google 账号层的共享/定位；USB 调试本身是攻击面，用完请关闭
- 清理的是广告与预装，不是越狱；对任何包的作用拿不准时，skill 会选择「不执行」而不是赌一把

## 开源协议

[MIT](LICENSE)
