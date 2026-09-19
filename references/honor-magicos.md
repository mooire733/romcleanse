# MagicOS（荣耀）包分类表

> ⚠️ 状态：未实机验证。执行前按 unknown-rom-triage 流程二次确认。
> 检出签名：`ro.build.version.magic` 或存在 `ro.hihonor.*`。

## T1 广告/追踪框架候选

| 包名 | 作用 |
|---|---|
| com.hihonor.systemserver 内嵌统计不可拆，跳过 | — |
| com.hihonor.datareport | 数据上报 |
| com.hihonor.logupload | 日志上传 |
| com.hihonor.android.launcher.ai.dci? | 以实机枚举为准，任何不确定的 launcher 相关包一律 T0 |

## T2 推广型预装候选

| 包名 | 应用 |
|---|---|
| com.hihonor.appstore | 荣耀应用市场（替代渠道可用时） |
| com.hihonor.browser | 荣耀浏览器 |
| com.hihonor.magazineunlock | 杂志锁屏（锁屏广告，对应小米画报） |
| com.hihonor.thememall | 主题商店 |
| com.hihonor.gamecenter | 游戏中心 |
| com.hihonor.video / com.hihonor.music | 视频/音乐 |
| com.hihonor.phoneservice | 服务/会员中心（推广） |
| com.hihonor.my honorspace | 荣耀亲选/商城类 |

## NEVER（本厂商禁区）

com.hihonor.android.launcher（桌面）、com.hihonor.systemmanager（手机管家核心，只硬化不卸）、com.hihonor.push（推送）、com.hihonor.cloudservice（云）、com.hihonor.id / 账号组件、com.hihonor.findmyphone（查找手机）、使用中的输入法

## 特殊注意

- 荣耀与华为同源但已分家：包名前缀 com.hihonor（荣耀）/ com.huawei（华为底层件可能仍存在，按华为表处理并同样保守）。
- systemmanager 通知权限若被系统固定，走「仅可手动关」记录，不硬碰。
