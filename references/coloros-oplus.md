# ColorOS / OxygenOS / realme UI（OPPO · 一加 · realme）包分类表

> ⚠️ 状态：未实机验证。清单来自通用知识 + 社区 debloat 经验，执行前必须按 unknown-rom-triage 流程对每个包二次确认（先硬化后卸载，卸载前列入提案等用户确认）。
> 检出签名：`ro.build.version.oplusrom`；brand 用 `ro.product.brand` 区分 OPPO / OnePlus / realme。
> 旧系统（ColorOS 11 及更早）命名空间为 com.coloros.* / com.oppo.* / com.nearme.*。

## T1 广告/追踪框架候选

| 包名 | 作用 | 备注 |
|---|---|---|
| com.oplus.midas | 广告/内容推荐服务 | 高优先 |
| com.nearme.statistics.rom | 统计上报（旧机型） | |
| com.coloros.statistics.rom | 统计上报 | |
| com.oplus.statistics.rom | 统计上报（新命名） | |
| com.oplus.appplatform | 商店/推广服务组件 | |
| com.oplus.ocs | OPPO 云服务组件 | 谨慎，先硬化观察 |

## T2 推广型预装候选

| 包名 | 应用 |
|---|---|
| com.heytap.market | HeyTap 软件商店 |
| com.heytap.browser | HeyTap 浏览器 |
| com.heytap.themestore | 主题商店 |
| com.heytap.pictorial | 乐划锁屏（锁屏广告/画报，对应小米画报） |
| com.heytap.music / com.heytap.musicx | 音乐 |
| com.heytap.cloud | 云服务（用户用 O 云则保留） |
| com.coloros.weather.service | 天气服务（功能件，默认只硬化） |
| com.nearme.gamecenter | 游戏中心（旧机型） |

## T3 第三方预装候选

按 `pm list packages -3 -i` 的 installer 直方图找：installer 为 com.heytap.market / com.nearme.market / 换机类（com.coloros.backuprestore 等）装进来的非厂商应用。

## NEVER（本厂商禁区）

桌面（com.android.launcher3 / com.oppo.launcher / com.coloros.pictorial 除外）、com.android.systemui、com.coloros.safecenter / com.oplus.safecenter（安全中心核心）、com.coloros.phonemanager（管家核心）、com.oplus.push / com.heytap.mcs（厂商推送，删了自带应用通知全灭）、使用中的输入法（com.iflytek.inputmethod / com.sohu.inputmethod 常见）、账号（com.coloros.account / com.oppo.account / com.heytap.usercenter）

## 特殊注意

- ColorOS 对核心组件的 -1000 更常见，一律走 disable-user 降级。
- 「软件更新」组件（com.oplus.sau / com.coloros.ota）建议保留——OTA 安全更新依赖，仅硬化通知。
