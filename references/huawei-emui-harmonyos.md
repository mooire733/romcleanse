# EMUI / HarmonyOS 2–4（华为）+ HarmonyOS NEXT 中止路径

> ⚠️ 状态：未实机验证。华为系对 adb 限制最严，本文件以「先探测能力，再谈清理」为纲。

## 0. 能力探测（先于一切）

```
adb shell getprop hw_sc.build.os.version        # HarmonyOS 2-4 / EMUI 均可能有
adb shell getprop ro.build.version.emui         # EMUI 签名
adb shell echo ok                               # shell 是否可用
adb shell pm list packages | head -3            # pm 是否可用
```

- **HarmonyOS NEXT（纯血鸿蒙）**：`adb devices` 无设备或任何 shell 命令被拒/无输出 → **立即中止**。输出给用户：「纯血鸿蒙不开放 adb，本次只能指导你在手机设置里手动操作」，并列出设置路径（设置→应用管理、通知管理、广告与隐私）。
- shell 可用但 pm 受限（返回空/权限错误）→ 降级为「指导手动操作」模式，不硬试。

## T1 广告/追踪框架候选（shell 可用时）

| 包名 | 作用 |
|---|---|
| com.huawei.systemmanager 内嵌上报不可拆，跳过 | — |
| com.huawei.hwcloudmgr 的统计子组件 | 以实机为准 |
| com.huawei.lbs | 位置上报（谨慎，影响地图定位） |

## T2 推广型预装候选

| 包名 | 应用 |
|---|---|
| com.huawei.appmarket | 华为应用市场 |
| com.huawei.browser | 华为浏览器 |
| com.huawei.magazine | 杂志锁屏（锁屏广告） |
| com.huawei.android.thememanager | 主题 |
| com.huawei.gameassistant | 游戏助手/游戏中心 |
| com.huawei.video / com.huawei.music | 视频/音乐 |
| com.huawei.health 的推广部分 | 保留（运动健康是功能件） |

## NEVER（本厂商禁区）

com.huawei.android.launcher、com.huawei.systemmanager（手机管家）、com.huawei.hms.*（HMS 框架——删了地图/推送/账号全灭）、com.huawei.hwid（华为账号）、com.huawei.cloud / com.huawei.hicloud（云）、com.huawei.findmyphone*（查找手机）、使用中的输入法

## 特殊注意

- 华为系「纯净模式」和通知管理大部分锁在系统设置里，adb 改不动——预期管理：能卸的很少，输出以「手动操作指引」为主。
- 任何 `pm` 报 SecurityException/非 0 且无说明 → 记录并停止该包，不降级硬试。
