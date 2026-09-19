# 小米 HyperOS / MIUI 包分类表

> ✅ 状态：实机验证（Redmi HyperOS 1.0 / Android 14 与 HyperOS 3.0 / Android 16，2026-09-19 两台实测）
> 适用：小米全系列、Redmi 全系列。验证差异记录在文末。

## T1 广告/追踪框架（建议全卸）

| 包名 | 作用 |
|---|---|
| com.miui.systemAdSolution | MSA 系统广告框架（投放核心，最优先） |
| com.miui.analytics | 数据分析 |
| com.xiaomi.metoknlp | 用户画像/数据挖掘 |
| com.xiaomi.security.onetrack | 用户行为追踪 |
| com.xiaomi.ugd | 用户数据收集 |
| com.xiaomi.ab | A/B 实验投放 |
| com.xiaomi.joyose | 云控（远程下发推广策略） |
| com.xiaomi.mtb | 埋点上报 |
| com.miui.contentcatcher | 负一屏信息流抓取 |
| com.xiaomi.barrage | 推流/弹幕组件 |
| com.miui.uireporter | UI 行为上报 |
| com.miui.dmregservice | 运营商设备注册上报 |
| com.miui.daemon | 数据采集守护进程 |
| com.miui.otaprovision | OTA 预装下发（部分机型缺失，属正常） |

## T2 推广型预装应用（按用户选择：卸载 / 仅硬化 / 保留）

| 包名 | 应用 | 处置建议 |
|---|---|---|
| com.miui.hybrid | 快应用框架（广告重灾区） | 卸载 |
| com.miui.personalassistant | 负一屏（信息流广告） | 卸载或硬化 |
| com.miui.phrase | 神键手（剪贴板推广） | 卸载 |
| com.miui.yellowpage | 生活黄页 | 卸载 |
| com.mfashiongallery.emag | 锁屏画报（HyperOS 常返 -1000） | 禁用 |
| com.miui.cleanmaster | 垃圾清理（常返 -1000） | 禁用 |
| com.miui.newhome | 小米新闻（常返 -1000） | 禁用 |
| com.xiaomi.market | 应用商店（推荐流在应用内设置，无法 adb 关） | 保留+硬化 |
| com.xiaomi.gamecenter.sdk.service | 游戏中心 SDK | 保留+硬化（通知+doze 白名单都要处理） |
| com.miui.securitycenter | 手机管家 | 保留；通知 HyperOS 3 为双固定权限，AppOps 压不住，只能手动关 |
| com.miui.securityadd | 安全组件 | 不动（它不发自己的通知，跟随手机管家） |
| com.miui.contentextension | 传送门 | 默认保留（功能件），用户点名才卸 |
| com.miui.voiceassist | 小爱同学 | 默认保留（语音唤醒依赖其通知） |
| com.miui.themestore | 主题商店 | 用户选择 |
| com.miui.huanji | 换机助手 | 用户选择（迁移完可删） |

## T3 第三方预装（实测出现过的）

com.jiaohua_browser（垃圾浏览器）、cn.wps.moffice_eng.xiaomi.lite（WPS 小米版，开屏广告）、com.iflytek.inputmethod.miui（讯飞输入法）、com.baidu.input_mi（百度输入法小米版）、com.mi.liveassistant（小米直播）、com.xiaomi.shop（小米商城）

## NEVER（本厂商禁区）

com.miui.home（桌面）、com.miui.system、com.miui.notification、com.miui.securitycore、com.xiaomi.finddevice、com.xiaomi.account、com.miui.cloudservice、com.miui.micloudsync、com.miui.guardprovider（扫描服务）、com.lbe.security.miui（权限管理）、com.xiaomi.xmsf（厂商推送，删了部分自带应用通知全灭）、使用中的输入法（常为 com.sohu.inputmethod.sogou.xiaomi 或上述 T3 输入法之一）

## 特殊记录（HyperOS 1 vs 3）

- `pm uninstall -k --user 0` 对 emag/cleanmaster/newhome 均 -1000 → `pm disable-user` 全部成功。
- com.miui.securitycenter 通知：HyperOS 1 用 `appops POST_NOTIFICATION deny` 可压住；HyperOS 3 AppOps 也压不住（读回 allow），属双固定权限，唯一途径是用户手动关。
- HyperOS 3 广告框架会自加 doze 白名单（systemAdSolution/metoknlp/analytics/market/gamecenter SDK），硬化时逐个 `dumpsys deviceidle whitelist -<pkg>` 踢出。
- `settings secure msa_splash_config` 等残留配置在 MSA 卸载后为死数据，无需清理。
- 卸载前已在运行的进程（如 daemon）会残留到下次重启，无害，说明即可。
