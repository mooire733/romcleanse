# 隐私/监控审计（查监控 · 查流氓软件）

> 仅在用户提出时执行。全程只读。每项都给出「发现什么算异常」的判读标准。

## 1. 设备管理员 / MDM（企业监控）

```
adb shell dumpsys device_policy | grep -E 'Active admin|Device Owner|Profile Owner'
```
判读：Enabled Device Admins 列表非空、Device Owner 存在 → 异常（用户没主动开企业管理就是问题）。

## 2. 无障碍服务（spyware 最爱通道）

```
adb shell settings get secure enabled_accessibility_services
```
判读：任何第三方包启用无障碍服务且用户不记得 → 高危。官方输入法/厂商服务之外都值得追问。

## 3. 通知监听（读所有通知）

```
adb shell settings get secure enabled_notification_listeners
```
判读：出现陌生包 → 高危。厂商健康/手环类同步属正常。

## 4. VPN 流量劫持

```
adb shell settings get global always_on_vpn_app
adb shell settings get global always_on_vpn_lockdown
adb shell dumpsys connectivity | grep -iE 'Active VPN|NOT_VPN' | head -5
```
判读：always_on_vpn 非用户自己装的工具 → 异常。用户自装 v2ray/openvpn 未激活属正常。

## 5. 悬浮窗（能看屏幕内容）

```
adb shell appops query-op SYSTEM_ALERT_WINDOW allow
```
判读：陌生第三方包 → 中危（可做屏幕嗅探）。

## 6. Stalkerware 特征扫描

```
adb shell pm list packages | grep -iE 'spy|mspy|flexi|hover|coco|xnspy|mobix|spyic|truth|kidlog|monitor|keylog|phonetracker|webwatcher|mobicop|mobistealth|highster|ttspy|clevguard|kidsguard|fami.*safe|catch.*cheat|celltracker'
```
判读：零命中为佳；命中厂商官方组件（如 com.miui.audiomonitor 通话录音）属误报，需人工辨认。

## 7. 安装来源直方图（侧载排查）

```
adb shell pm list packages -3 -i | grep -oE 'installer=[^ ]*' | sort | uniq -c | sort -rn
adb shell pm list packages -3 -i | grep 'installer=null'    # 逐个列出，多为出厂预置
```
判读：installer=null 的非厂商包、或来源不明的包 → 列出追问用户。

## 8. 用户 CA 证书（中间人监控）

```
adb shell ls /data/misc/user/0/cacerts-added/    # 无 root 会 Permission denied
```
判读：adb 读不到时引导用户手动查：设置→安全→加密与凭据→用户凭据（正常为空）。

## 9. 诚实边界（报告必须包含）

- Google 账号级监控（位置共享/Find My/家庭组）adb 查不到，引导用户自查账号设置。
- USB 调试本身是攻击面：审计完提醒关闭；任何能碰到电脑的人都可以通过 adb 执行命令。
- 无 root 时部分证据（CA store、/data 日志）不可达，报告中如实标注「未验证项」。
