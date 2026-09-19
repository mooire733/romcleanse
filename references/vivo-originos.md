# OriginOS / Funtouch OS（vivo · iQOO）包分类表

> ⚠️ 状态：未实机验证。执行前按 unknown-rom-triage 流程二次确认。
> 检出签名：`ro.vivo.os.name`（OriginOS）/ `ro.vivo.os.version`；iQOO 亦走 vivo 命名空间。

## T1 广告/追踪框架候选

| 包名 | 作用 |
|---|---|
| com.vivo.daemonService | 数据采集守护 |
| com.vivoagent | 统计/上报 |
| com.bbk.logservice | 日志上报 |
| com.vivo.agent? / com.iqoo.agent | 行为分析（以实机枚举为准） |

## T2 推广型预装候选

| 包名 | 应用 |
|---|---|
| com.vivo.appstore | vivo 应用商店 |
| com.vivo.browser | vivo 浏览器 |
| com.vivo.game / com.vivo.gamecube | 游戏中心 |
| com.bbk.theme | 主题商店 |
| com.vivo.video | 影视（信息流） |
| com.vivo.music | 音乐 |
| com.vivo.wallet | vivo 钱包 |
| com.vivo.email | 自带邮箱 |
| com.bbk.theme? com.vivo.themestore | 主题（以实机为准） |

## NEVER（本厂商禁区）

com.android.launcher（vivo 桌面）、com.iqoo.secure（i管家核心，只硬化不卸）、com.vivo.push（厂商推送，删了自带应用通知全灭）、com.vivo.doubleinstance、账号/云（com.vivo.account / com.bbk.cloud）、com.vivo.fingerprint、使用中的输入法

## 特殊注意

- vivo 的安全组件 com.iqoo.secure 兼管自启动白名单，禁用会破坏系统设置页——禁碰。
- com.vivo.push 影响面大：若用户依赖 vivo 推送的自带应用通知（短信验证类不受影响），卸载前列入「可禁但需明确告知后果」。
