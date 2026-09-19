---
name: romcleanse
description: >-
  经 SSH 隧道到 Mac、通过 adb 为国产安卓手机去广告/去臃肿/关推送/限后台（user 0 无 root）。
  覆盖：小米 HyperOS/MIUI、OPPO·一加·realme（ColorOS）、vivo/iQOO（OriginOS/Funtouch）、
  荣耀 MagicOS、华为 EMUI/HarmonyOS 2-4、魅族 Flyme、中兴/努比亚。
  触发：手机去广告、卸载预装软件、关闭系统广告、手机预装清理、debloat、adb 清理手机、
  HyperOS 广告、ColorOS 广告、关闭推送通知、限制后台自启、手机耗电发热排查、
  手机被监控检测、查流氓/间谍软件。HarmonyOS NEXT（纯血）无 adb，检出即中止并转人工。
---

# romcleanse — 国产安卓去广告与后台治理

## 0. 铁律（先于一切）

- **只做 `--user 0`**：禁 root/su/remount；禁碰 /system；禁 `pm uninstall` 不带 `--user 0`。
- **NEVER 清单**（任何情况下不得卸载/禁用，不得提案）：
  - 当前或唯一启用的输入法（先查 `settings get secure default_input_method` + `ime list -s`）
  - launcher（如 com.miui.home / com.android.launcher3 / 各厂商桌面）、`com.android.systemui`
  - 通知框架（如 com.miui.notification）、安全内核（如 com.miui.securitycore）、查找设备（finddevice）
  - `com.android.settings`、packageinstaller、telecom/telephony 栈、活跃 device-admin 持有者
  - 厂商账号/云服务（com.xiaomi.account、cloudservice 等）默认不动，用户点名才提案
- **全程只一个交互点**：§4 分级方案确认。其余不逐包请示；未知包默认只提案不执行。
- 多台设备同时在线时，**所有命令必须 `adb -s <serial>`**。
- 收尾必须提醒用户关闭 USB 调试（开着 = 任何能碰到 Mac 的人都能执行命令）。

## 1. 环境接入

- Agent → Mac：`sshpass -e ssh -p 2233 user@127.0.0.1`（隧道建立方式见 `lan-ssh-tunnel` skill）。
- adb 在 Mac：`~/platform-tools/adb`；不存在则 `curl -sSL -o ~/pt.zip https://dl.google.com/android/repository/platform-tools-latest-<darwin|linux>.zip && unzip -oq ~/pt.zip -d ~`。
- 先 `adb devices -l` 验在线。中止条件：`unauthorized`（让用户在手机屏幕点允许后再继续）/ `offline` / 无设备。

## 2. ROM 识别（改动前第一步）

一条 getprop 大网打签名：

```
adb shell 'getprop | grep -Ei "mi\.os|miui|oplus|oppo|realme|vivo|bbk|honor|hihonor|magic|emui|hw_sc|harmony|flyme|meizu|zte|nubia|product\.brand|product\.manufacturer"'
```

| ROM | 判定 prop | 状态 |
|---|---|---|
| HyperOS | `ro.mi.os.version.name` / `ro.mi.os.version.code` | ✅ 实测 |
| MIUI | `ro.miui.ui.version.name` | ✅ 实测 |
| ColorOS / 一加 / realme | `ro.build.version.oplusrom`（brand 区分三家） | ⚠️ 未实机验证 |
| OriginOS / Funtouch | `ro.vivo.os.name` / `ro.vivo.os.version` | ⚠️ |
| MagicOS | `ro.build.version.magic` 或存在 `ro.hihonor.*` | ⚠️ |
| EMUI | `ro.build.version.emui` | ⚠️ |
| HarmonyOS 2–4 | `hw_sc.build.os.version` / `hw_sc.build.platform.version`；adb 部分受限，逐条试 | ⚠️ |
| **HarmonyOS NEXT** | adb 无设备或 shell 全拒 → **立即中止**，告知「纯血鸿蒙无 adb，只能在手机设置里手动操作」 | 规则确定 |
| Flyme | `ro.build.display.id` 含 "Flyme" | ⚠️ |
| 兜底 | `ro.product.brand` + `ro.product.manufacturer` | — |

- 命中厂商 → 加载 `references/<vendor>.md` 取包分类表。
- **签名落空 ≠ 不支持**：全部落空时走 `references/unknown-rom-triage.md` 保守模式（删除类操作只提案不执行）。

## 3. 安全快照（改任何东西之前，必做）

```bash
TS=$(date +%Y%m%d_%H%M%S)
adb shell pm list packages | sort > ~/phone_pkg_backup_$TS.txt          # 恢复依据，必须落盘到 Mac
adb shell pm list packages -d  > ~/phone_pkg_disabled_$TS.txt           # 已禁用清单基线
adb shell settings get secure default_input_method                     # IME 保底依据
adb shell ime list -s                                                  # 启用中的输入法全集
adb shell dumpsys dropbox | grep -cE 'crash|anr'                       # 崩溃基线，收尾对比
```

审计在范围内时加存：`enabled_accessibility_services`、`enabled_notification_listeners`。

## 4. 包分类与分级提案（唯一交互点）

按厂商 reference 把**已装包**分成四层，输出一张表给用户确认：

- **T0 禁区**：绝不提案（NEVER 清单 + reference 内标注的厂商核心件）
- **T1 广告/追踪框架**：纯后台组件，无界面功能
- **T2 推广型预装应用**：自带 app 带广告位/推广推送
- **T3 第三方预装垃圾**：换机/商店塞进来的非厂商应用

不在任何表的包 → 按 `references/unknown-rom-triage.md` 信号打分，归入「待用户确认」组，不混入 T1–T3。
用户确认执行哪几层后才开始改。

## 5. 执行阶梯（逐包，遇错降级不升级）

```
主路径： pm uninstall -k --user 0 <pkg>        # APK 与数据保留，可恢复
  ↳ Failure [-1000] / 被拒 → pm disable-user --user 0 <pkg>   # 实测 -1000 全部能禁用成功
    ↳ 仍失败 → 记 skipped，不尝试更高危手段
```

- 每批 ≤10 包；每批后跑 alive 检查：
  - `cmd package query-activities -a android.intent.action.MAIN -c android.intent.category.HOME` 非空
  - `pidof com.android.systemui` 有 pid
- **中止并回滚触发条件**：HOME 查询为空 / systemui 无 pid / 设备中途 offline / `dumpsys dropbox` 出现新 crash → 停手，对本批包跑 `pm install-existing` 回滚，如实报告。

## 6. 硬化四件套（对保留但需噤声/限自启的包）

1. **通知**：`pm revoke <pkg> android.permission.POST_NOTIFICATIONS`
   - SecurityException（系统固定权限）→ `appops set <pkg> POST_NOTIFICATION deny`
   - 读回仍 allow（HyperOS 3 手机管家案例）→ 记「仅可手动关：设置→通知管理」，**不反复尝试**，不对核心安全件做更深操作
   - 包本就没申请该权限 → 正常跳过，不算失败
2. **后台启动**：`appops set <pkg> RUN_ANY_IN_BACKGROUND ignore`（这是有效层）
3. **待机桶**：`am set-standby-bucket <pkg> restricted`
   - 可能被厂商 powerkeeper 改回（实测 HyperOS 3 对 3/20 个回跳到 20/10）——**bucket 回跳不算失败，不重试**，AppOps ignore 才是挡自启动的实体
4. **Doze 白名单**：`dumpsys deviceidle whitelist -<pkg>`（广告框架常自加白，逐个踢出）

## 7. 验证套件（收尾必跑，逐项出证据）

```
cmd package query-activities -a android.intent.action.MAIN -c android.intent.category.HOME   # 非空
pidof com.android.systemui                                                                   # 有 pid
pm list packages -d | grep <被禁用的包>                                                        # 命中
dumpsys dropbox | grep -cE 'crash|anr'                                                        # 不高于基线
settings get secure default_input_method                                                      # 未变
```

报告附：改动计数（卸载 n / 禁用 n / 硬化 n）、快照文件路径、恢复命令、USB 调试关闭提醒。

## 8. 恢复手册（随报告输出）

```
pm install-existing <pkg>                                  # 撤销 uninstall -k --user 0
pm enable <pkg>                                            # 撤销 disable-user
appops set <pkg> POST_NOTIFICATION default
appops set <pkg> RUN_ANY_IN_BACKGROUND allow
am set-standby-bucket <pkg> active
dumpsys deviceidle whitelist +<pkg>
```

## 9. 隐私/监控审计（按需）

仅当用户提出「查监控/查流氓软件」时加载 `references/privacy-audit.md`，不默认执行。
