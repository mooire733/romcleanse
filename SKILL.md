---
name: romcleanse
description: >-
  通过 adb 为国产安卓手机去广告/去臃肿/关推送/限后台（user 0 无 root）。
  覆盖：小米 HyperOS/MIUI、OPPO·一加·realme（ColorOS）、vivo/iQOO（OriginOS/Funtouch）、
  荣耀 MagicOS、华为 EMUI/HarmonyOS 2-4、魅族 Flyme、中兴/努比亚。
  触发：手机去广告、卸载预装软件、关闭系统广告、手机预装清理、debloat、adb 清理手机、
  HyperOS 广告、ColorOS 广告、关闭推送通知、限制后台自启、手机耗电发热排查、
  手机被监控检测、查流氓/间谍软件。HarmonyOS NEXT（纯血）无 adb，检出即中止并转人工。
---

# romcleanse — 国产安卓去广告与后台治理

## 0. 铁律（先于一切）

- **只做 `--user 0`**：禁 root/su/remount；禁碰 /system；禁 `pm uninstall` 不带 `-k`。
- **命令白名单制**：只允许 §5/§6 列出的命令。**显式禁止**：`pm clear`（清用户数据，不可逆）、`pm uninstall` 不带 `-k`、`pm disable`（只用 disable-user）、`settings put`（global/secure/system 任何写入）、`cmd package suspend/hide`、`am force-stop` 之外的任何 am 写操作、`input`/`keyevent` 模拟点击。白名单/阶梯之外的命令一律记 skipped，**禁止即兴发挥**。
- **NEVER 清单**（任何情况下不得卸载/禁用，不得提案）：
  - **动态 NEVER**：§3 解析出的**当前默认桌面包名**、`ime list -s` 输出的**全部启用输入法** + `default_input_method` 指向的包——无论它们是否出现在任何厂商表中
  - launcher（如 com.miui.home / com.android.launcher3 / 各厂商桌面）、`com.android.systemui`
  - 通知框架（如 com.miui.notification）、安全内核（如 com.miui.securitycore）、查找设备（finddevice）
  - `com.android.settings`、packageinstaller、telecom/telephony 栈、活跃 device-admin 持有者
  - 厂商账号/云服务（com.xiaomi.account、cloudservice 等）默认不动，用户点名才提案
- **全程只一个交互点**：§4 分级方案确认（含行内「用户选择」条目的逐项确认）。其余不逐包请示；未知包默认只提案不执行。
- 多台设备在线时，所有命令必须 `adb -s <serial>`；下文代码块中的 `adb` 是 `adb -s $SERIAL`（形态 B 时在 SSH 内层执行）的简写。
- 收尾必须提醒用户关闭 USB 调试（开着 = 任何能碰到这台电脑的人都能执行命令）。

## 1. 环境接入（两种部署形态，先探测用哪种）

**形态 A：直连（最常见）** — Agent/harness 就跑在手机 USB 所连的那台电脑上：
- adb 在本机：macOS `brew install --cask android-platform-tools`；Linux/Windows 下载官方 platform-tools；Windows 下命令为 `adb.exe`
- 直接 `adb devices -l` 验证

**形态 B：远程隧道** — Agent 在远程服务器，手机接在用户的 Mac 上：
- Agent → Mac：`sshpass -e ssh -p 2233 user@127.0.0.1 -o StrictHostKeyChecking=accept-new`（隧道建立方式见 `lan-ssh-tunnel` skill）
- **密码纪律**：密码只能经 `SSHPASS` 环境变量传入（用户预先 export）；**禁止** `sshpass -p <明文>`（会进进程列表）、禁止把密码写进任何文件或命令行；会话结束 `unset SSHPASS`
- adb 在 Mac：`~/platform-tools/adb`；不存在则 `curl --proto '=https' -sSfL -o ~/pt.zip https://dl.google.com/android/repository/platform-tools-latest-darwin.zip && unzip -oq ~/pt.zip -d ~ && rm ~/pt.zip`（优先 `brew install --cask android-platform-tools`，自带校验）
- 每条 adb 命令都要包一层 SSH 执行

**通用规则**：
- 用 `adb devices -l` 探测：本机能看到设备 → 形态 A；看不到但存在 SSH 隧道 → 形态 B。检测到哪种就全程固定用哪种，**不混用**
- 中止条件：`unauthorized`（让用户在手机屏幕点允许）/ `offline` / 无设备
- 多台设备在线时，所有命令必须 `adb -s <serial>`（形态 B 中在 SSH 内层加）

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
# 包清单（去掉 package: 前缀与 \r，恢复脚本直接可用）
adb shell pm list packages | tr -d '\r' | sed 's/^package://' | sort > ~/phone_pkg_backup_$TS.txt
adb shell pm list packages -d > ~/phone_pkg_disabled_$TS.txt
# 输入法全集（全部进动态 NEVER）
adb shell settings get secure default_input_method
adb shell ime list -s
# 当前默认桌面（进动态 NEVER + 作为 alive 基线）
adb shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME > ~/home_baseline_$TS.txt
# doze 白名单基线（恢复时只加回原本存在的条目）
adb shell dumpsys deviceidle whitelist > ~/doze_baseline_$TS.txt
# 崩溃基线：记计数 + 最新条目时间戳行（dropbox 会自动清理旧条目，计数可能不升反降，时间戳才可靠）
adb shell dumpsys dropbox | grep -cE 'crash|anr'
adb shell dumpsys dropbox | grep -m1 -E '^[0-9]{4}-'
# 多用户记录（--user 0 不触达分身/工作资料，报告需注明）
adb shell pm list users
```

审计在范围内时加存：`enabled_accessibility_services`、`enabled_notification_listeners`。

## 4. 包分类与分级提案（唯一交互点）

按厂商 reference 把**已装包**分成四层，输出一张表给用户确认：

- **T0 禁区**：绝不提案（NEVER 清单 + reference 内标注的厂商核心件）
- **T1 广告/追踪框架**：纯后台组件，无界面功能
- **T2 推广型预装应用**：自带 app 带广告位/推广推送
- **T3 第三方预装垃圾**：换机/商店塞进来的非厂商应用

**提案表每行必须标注恢复方式**——先跑 `adb shell pm path <pkg>`：
- 路径以 `/system`、`/product`、`/vendor` 开头 → 标「✅ 可一键恢复（install-existing）」
- 路径以 `/data/app` 开头 → 标「⚠️ 卸载后需重装，不可自动恢复」，且**该行必须逐项点名确认，不随层级确认批量放行**

不在任何表的包 → 按 `references/unknown-rom-triage.md` 信号打分，归入「待用户确认」组，不混入 T1–T3。
**层级确认不覆盖行内标注**：厂商表中标「用户选择」「卸载或硬化」的条目必须逐项点名，用户逐个拍板。

## 5. 执行阶梯（逐包，遇错降级不升级）

```
主路径： pm uninstall -k --user 0 <pkg>        # 系统分区包：APK 保留，可恢复
  ↳ 任何失败（含 Failure [-1000]）→ pm disable-user --user 0 <pkg>   # 同等可逆
    ↳ 仍失败 → 记 skipped，终点。禁止尝试任何阶梯外手段（见 §0 白名单制）
```

- **改动台账**：每改一个包，追加一行到 `~/phone_pkg_changes_$TS.log`：`<pkg> <action> <逆操作命令>`——完整回滚靠它，不靠记忆。
- 每批 ≤10 包；每批后跑 alive 检查：
  - `cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME` 结果与 `~/home_baseline_$TS.txt` **逐字符一致**（非空不够，防第三方桌面假阴性）
  - `pidof com.android.systemui` 有 pid
  - 等 30 秒后查 dropbox：**存在时间戳晚于基线的 crash/anr 条目** → 异常（计数不可靠，dropbox 会清理旧条目）
- **中止并回滚触发条件**：HOME 解析结果变化 / systemui 无 pid / 设备中途 offline / dropbox 出现新于基线的 crash → 停手，回滚**自上次全部检查通过以来的所有批次**（按 `phone_pkg_changes_$TS.log` 逆序执行逆操作），如实报告。

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
cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME   # 与基线一致
pidof com.android.systemui                                                                           # 有 pid
pm list packages -d | grep <被禁用的包>                                                               # 命中
dumpsys dropbox | grep -E 'crash|anr'                                                                # 无新于基线时间戳的条目
settings get secure default_input_method                                                             # 未变
```

报告附：改动计数（卸载 n / 禁用 n / 硬化 n）、每项恢复方式、多用户说明（如有分身/工作资料：本次仅治理 user 0）、快照与台账文件路径、恢复命令、USB 调试关闭提醒。

## 8. 恢复手册（随报告输出）

```bash
# 一键恢复所有被卸载的系统分区包（与当前列表求差）：
comm -23 ~/phone_pkg_backup_$TS.txt \
  <(adb shell pm list packages | tr -d '\r' | sed 's/^package://' | sort) \
  | while read p; do adb shell pm install-existing "$p"; done

adb shell pm enable <pkg>                              # 撤销 disable-user（恢复为出厂版本，商店更新会丢，需重装更新）
adb shell appops set <pkg> POST_NOTIFICATION default   # 恢复通知
adb shell appops set <pkg> RUN_ANY_IN_BACKGROUND allow # 恢复后台启动
# 待机桶恢复基线值（基线非 active 就回基线值，不要一律 active）
adb shell dumpsys deviceidle whitelist +<pkg>          # 仅加回 doze_baseline 中原本存在的条目
```

注意：/data 分区的第三方预装（提案表中标 ⚠️ 的行）卸载后无 APK 可恢复，只能从应用商店重装。

## 9. 隐私/监控审计（按需）

仅当用户提出「查监控/查流氓软件」时加载 `references/privacy-audit.md`，不默认执行。
