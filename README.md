# Forensic Analysis — Samsung SM-A175F (Galaxy A17)
### Silent Installation of Remote Access Tool via OMC Agent

**Analyst:** SyliSec  
**Devices analyzed:** 2 × Samsung SM-A175F (Android 16, patch 2026-03-05)  
**Analysis dates:** 2026-08-06 / 2026-08-25  
**Status:** Private review — pending responsible disclosure to Samsung

---

## Summary

On two brand-new Samsung Galaxy A17 devices, forensic analysis revealed that **OMC Agent** (`com.samsung.android.app.omcagent`) — a system app displayed as "Recommended apps" in the Android UI — silently installs **RSSupport AAS2** (`com.rsupport.rs.activity.rsupport.aas2`) during initial device setup, without any user notification or consent.

The installation is triggered when `Settings.Global.device_provisioned = 1`, which can occur **before the Setup Wizard is fully completed**.

RSSupport AAS2 (also known as "Smart Tutor") is a full remote-control application that receives 11 critical permissions automatically, including screen capture, touch injection, and call log access.

**No remote session was detected on either device. No malware was found.**

---

## Key Findings

| Finding | Detail |
|---|---|
| Trigger | `device_provisioned=1` set by OMC Agent |
| App installed | RSSupport AAS2 (`com.rsupport.rs.activity.rsupport.aas2`) |
| Installer | OMC Agent (`com.samsung.android.app.omcagent`) |
| UI name of OMC Agent | **"Recommended apps"** — misleading |
| Permissions granted | 11 critical (screen capture, touch control, call log...) |
| Devices confirmed | 2 × SM-A175F, builds 452 and 454 |
| Remote sessions | None detected |
| Prior publications | **None documenting this exact mechanism** |

---

## Repository Structure

```
├── rapport_forensic_samsung_A17.md   # Full technical report (French)
├── rapport_forensic_samsung_A17.pdf  # PDF version
└── captures/
    ├── 01_rsupport_absent_appareil2.txt       # ADB: RSSupport absent before provisioning
    ├── 02_setup_wizard_non_termine.txt        # ADB: user_setup_complete=null
    ├── 03_omcagent_setupwizard_listener.txt   # ADB: OMC Agent broadcast listeners
    ├── 04_omcagent_present.txt                # ADB: OMC Agent version confirmed
    ├── 05_screenshot_appareil2.png            # Device 2 home screen (Setup Wizard pending)
    ├── 06_apps_tierces_appareil2.txt          # 26 third-party apps, RSSupport absent
    └── 07_settings_apps_appareil2.png         # Settings > Apps (RSSupport absent)
```

---

## Causal Proof

| `device_provisioned` | `user_setup_complete` | RSSupport AAS2 |
|---|---|---|
| `null` | `null` | **Absent** |
| `1` | `null` or `1` | **Present, installed by OMC Agent** |

Two devices, two different builds (452 and 454), same mechanism → **reproducible causal evidence**.

---

## Critical Permissions Granted Automatically

| Permission | Capability |
|---|---|
| `CAPTURE_VIDEO_OUTPUT` | Real-time screen capture |
| `READ_FRAME_BUFFER` | Screen image capture |
| `INJECT_EVENTS` | Simulate touches / remote control |
| `READ_CALL_LOG` | Call history access |
| `DELETE_PACKAGES` | Uninstall any app |
| `SYSTEM_ALERT_WINDOW` | Overlay on all apps |
| `INTERACT_ACROSS_USERS_FULL` | Cross-user access |
| `READ_PRIVILEGED_PHONE_STATE` | Full phone state data |
| `DUMP` | Full system dump |
| `QUERY_ALL_PACKAGES` | List all installed apps |
| `INTERNET` | Network access |

---

## Activation Flow

```
Samsung Technician
    → Samsung Backend
        → OMC Agent (Intent with serveraddr + conncode)
            → RSSupport AAS2 connects to rc-samsung-global-win.rsup.io
                → Remote control session established
```

The app **cannot self-activate** — it requires a trigger from Samsung's backend infrastructure. However, Samsung can remotely activate it at any time without requiring physical user action at the moment of activation.

---

## Mitigation

```bash
# Remove RSSupport AAS2
adb shell pm uninstall --user 0 com.rsupport.rs.activity.rsupport.aas2
```

---

## Disclosure Status

- [x] Internal forensic analysis completed
- [x] Reproduced on second device
- [ ] Responsible disclosure sent to Samsung
- [ ] Samsung response received
- [ ] Public disclosure

---

## Related Work

| Topic | Source | Coverage |
|---|---|---|
| AppCloud/ironSource pre-installed on Samsung | Forbes, Nov. 2025 | Wide |
| Qihoo 360 pre-installed Samsung | Forbes, Jan. 2020 | Wide |
| LANDFALL spyware CVE-2025-21042 | TechCrunch, Nov. 2025 | Wide |
| **RSSupport AAS2 silent install mechanism** | **This report** | **Not previously documented** |

---

*Tools: ADB + jadx 1.5.6 + aapt | SyliSec 2026*
