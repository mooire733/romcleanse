# romcleanse

[English](README.md) | [简体中文](README.zh-CN.md)

> De-advertise, de-bloat and tune background behavior on Chinese-vendor Android phones — an [OpenCode](https://github.com/sst/opencode) skill driven by adb, no root required.

## What is this

`romcleanse` is an [OpenCode skill](https://opencode.ai) that lets an AI agent clean system-level ads from your Chinese-vendor Android phone over adb: uninstall ad/tracking frameworks, mute preloaded promo apps, block background auto-start, kick ad frameworks out of the doze whitelist — plus an optional privacy/spyware audit.

**In one line**: cut the "ad suite" at all five layers (delivery, tracking, auto-start, wake privileges, notifications) — system-partition changes are reversible in one command, and /data third-party preloads are annotated per-row with their recovery path.

## Supported devices

| Vendor / ROM | Status |
|---|---|
| Xiaomi HyperOS / MIUI | ✅ Verified on real devices (HyperOS 1.0 & 3.0) |
| OPPO / OnePlus / realme (ColorOS) | ⚠️ Unverified — evidence-driven triage |
| vivo / iQOO (OriginOS / Funtouch) | ⚠️ Unverified |
| HONOR (MagicOS) | ⚠️ Unverified |
| Huawei (EMUI / HarmonyOS 2–4) | ⚠️ (degrades to manual guidance when adb is restricted) |
| Meizu Flyme / ZTE / nubia | ⚠️ (thin tables + triage flow) |
| **HarmonyOS NEXT** | 🚫 No adb — detected and aborted with manual guidance |

If the ROM signature doesn't match any vendor, the skill falls back to the conservative evidence-driven triage mode: removal operations are proposed only, never executed without per-item user confirmation.

## How it works

romcleanse is deployment-agnostic — two environment shapes are supported:

**Shape A: direct** (most common) — the phone is plugged into the computer running the agent/harness:

```
AI Agent / Harness (local machine)
   │  adb
   ▼
Phone (USB debugging authorized)
```

**Shape B: remote tunnel** — the agent runs on a remote server, the phone is attached to the user's Mac:

```
AI Agent (remote server)
   │  sshpass ssh (tunnel, e.g. JetBrains Gateway reverse forward :2233)
   ▼
Your Mac (adb platform-tools home)
   │  adb -s <serial>
   ▼
Phone (USB debugging authorized)
```

All operations are **user 0 level** (`pm uninstall -k --user 0`): the APK and its data stay on the system partition, so system-partition changes are reversible with a single command — no root, no /system modification. Third-party preloads living on /data cannot be auto-restored after uninstall (reinstall from the store instead); the proposal table annotates each row.

## Core techniques

| Goal | Method | Fallback |
|---|---|---|
| Uninstall ad/tracking frameworks & preloads | `pm uninstall -k --user 0 <pkg>` | any failure → `pm disable-user --user 0` |
| Kill notifications | `pm revoke android.permission.POST_NOTIFICATIONS` | system-fixed permission → `appops POST_NOTIFICATION deny` → if that won't stick, marked "manual only" |
| Block auto-start | `appops RUN_ANY_IN_BACKGROUND ignore` (the effective layer) | + `am set-standby-bucket restricted` (may be overridden by vendor power keeper — not treated as failure) |
| Kick doze whitelist | `dumpsys deviceidle whitelist -<pkg>` | ad frameworks often self-whitelist; kicked one by one |

## Safety design

- **Snapshot before any change**: full package list + disabled list saved to disk — restore reference comes with the report
- **IME protection**: default and all enabled IMEs are auto-detected and never touched (no lock-screen lockout)
- **Per-batch liveness checks**: default-launcher resolution compared byte-for-byte against baseline, SystemUI alive, dropbox crash timestamps vs baseline — any anomaly halts and rolls back
- **NEVER list**: launcher, SystemUI, notification framework, security core, Find Device, account/cloud services, active device-admin holders… never proposed
- **Change ledger**: every action recorded with its inverse command; full rollback is a scripted loop, not memory
- **Single interaction point**: tiered proposal (T1 ad frameworks / T2 promo apps / T3 third-party preloads) confirmed by you, then executed; cautious rows require per-item confirmation
- **Explicit command whitelist**: destructive shortcuts like `pm clear` (wipes user data) are forbidden by the skill's own rules

## Usage

Clone into your OpenCode skills directory:

```bash
git clone https://github.com/mooire733/romcleanse ~/.config/opencode/skills/romcleanse
```

Restart OpenCode, then tell your agent:

- "Clean the ads off my phone / remove preinstalled bloat"
- "Turn off push notifications and auto-start for these apps"
- "Check whether my phone is being monitored"

The agent will: detect the ROM → take a safety snapshot → propose tiers → (you confirm) → execute & harden → deliver a verification report with restore commands.

## Repository layout

```
romcleanse/
├── SKILL.md                      # Main flow: detect → snapshot → tier → execute → verify → restore
├── README.md                     # This file (English)
├── README.zh-CN.md               # Chinese readme
└── references/
    ├── xiaomi-hyperos.md         # ✅ Xiaomi/Redmi full package taxonomy (verified)
    ├── coloros-oplus.md          # OPPO/OnePlus/realme
    ├── vivo-originos.md          # vivo/iQOO
    ├── honor-magicos.md          # HONOR
    ├── huawei-emui-harmonyos.md  # Huawei + HarmonyOS NEXT abort path
    ├── other-vendors.md          # Meizu/ZTE/nubia
    ├── unknown-rom-triage.md     # Evidence-driven classification for unknown ROMs/packages
    └── privacy-audit.md          # Surveillance/stalkerware audit (loaded on demand)
```

## Restore cheat sheet

```bash
adb shell pm install-existing <pkg>                       # undo uninstall (system-partition packages)
adb shell pm enable <pkg>                                 # undo disable-user (restores factory version)
adb shell appops set <pkg> POST_NOTIFICATION default      # restore notifications
adb shell appops set <pkg> RUN_ANY_IN_BACKGROUND allow    # restore background starts
adb shell am set-standby-bucket <pkg> active              # restore standby bucket
adb shell dumpsys deviceidle whitelist +<pkg>             # re-add to doze whitelist
```

## Boundaries & disclaimer

- Non-root has hard limits: system-fixed permissions (some vendor managers' notification switches) and in-app ad settings (e.g. store recommendation feeds) can only be turned off manually — the skill says so honestly
- The audit cannot see Google-account-level sharing/location; USB debugging itself is an attack surface — turn it off when done
- This cleans ads and bloat, it is not jailbreaking; when the purpose of any package is uncertain, the skill chooses "don't touch" over guessing

## License

[MIT](LICENSE)
