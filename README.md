# Analyse Forensic — Samsung SM-A175F (Galaxy A17)

### Installation Silencieuse d'un Outil d'Accès à Distance via OMC Agent

**Analyste :** SyliSec
**Appareils analysés :** 5 (3× SM-A175F, 1× SM-G991N, 1× SM-S921B)
**Dates d'analyse :** 2026-08-06 / 2026-08-25 / 2026-09-27
**Statut :** Responsible disclosure envoyé (2026-09-19) — aucune réponse Samsung — publication 2026-10-02

---

## Résumé

Sur trois Samsung Galaxy A17 (SM-A175F) neufs, l'analyse forensic a révélé que **OMC Agent** (`com.samsung.android.app.omcagent`) — une app système affichée sous le nom **"Recommended apps"** dans l'interface Android — installe silencieusement **RSSupport AAS2** (`com.rsupport.rs.activity.rsupport.aas2`) lors de la configuration initiale, sans notification ni consentement de l'utilisateur.

L'installation est déclenchée dès que `Settings.Global.device_provisioned = 1`, ce qui peut survenir **avant même la fin du Setup Wizard**.

RSSupport AAS2 (connu commercialement sous le nom "Smart Tutor") est un outil complet de contrôle à distance qui reçoit automatiquement **11 permissions critiques**, dont la capture d'écran, le contrôle tactile et l'accès au journal d'appels.

**Aucune session de contrôle à distance n'a été détectée. Aucun malware trouvé.**

---

## Résultats Clés

| Élément                 | Détail                                                                                                             |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Déclencheur              | `device_provisioned=1` positionné par OMC Agent                                                                  |
| Application installée    | RSSupport AAS2 (`com.rsupport.rs.activity.rsupport.aas2`)                                                         |
| Installateur              | OMC Agent (`com.samsung.android.app.omcagent`)                                                                    |
| Nom affiché d'OMC Agent  | **"Recommended apps"** — volontairement trompeur                                                             |
| Permissions accordées    | 11 critiques (capture écran, contrôle tactile, journal d'appels...)                                               |
| Appareils confirmés      | 3× SM-A175F (builds 452, 454, 454 — dont une installation live) + 2 appareils comparatifs (SM-G991N KT, SM-S921B) |
| Sessions à distance      | Aucune détectée                                                                                                   |
| Publications antérieures | **Mécanisme non documenté dans la littérature sécurité**                                                 |

---

## Structure du Dépôt

```
├── rapport_forensic_samsung_A17.md   # Rapport technique complet
├── rapport_forensic_samsung_A17.pdf  # Version PDF
└── captures/
    ├── 01_rsupport_absent_appareil2.txt       # ADB : RSSupport absent avant provisioning
    ├── 02_setup_wizard_non_termine.txt        # ADB : user_setup_complete=null
    ├── 03_omcagent_setupwizard_listener.txt   # ADB : broadcast listeners OMC Agent
    ├── 04_omcagent_present.txt                # ADB : version OMC Agent confirmée
    ├── 05_screenshot_appareil2.png            # Appareil 2 (Setup Wizard non terminé)
    ├── 06_apps_tierces_appareil2.txt          # 26 apps tierces, RSSupport absent
    ├── 07_settings_apps_appareil2.png         # Paramètres > Applications (RSSupport absent)
    ├── 08_s21_model.txt                       # ADB : SM-G991N (Corée KT) — RSSupport absent
    ├── 09_s24_analysis.txt                    # ADB : SM-S921B (S24) — RSSupport absent
    └── 10_a175fds_live_install.txt            # ADB : SM-A175F/DS — installation live capturée
```

---

## Preuve Causale

| `device_provisioned` | `user_setup_complete` | RSSupport AAS2                              |
| ---------------------- | ----------------------- | ------------------------------------------- |
| `null`               | `null`                | **Absent**                            |
| `1`                  | `null` ou `1`       | **Présent, installé par OMC Agent** |

Trois SM-A175F, trois captures indépendantes (builds 452, 454, 454 — dont une installation observée en temps réel sur appareil neuf), même mécanisme → **preuve causale reproductible**.

---

## Permissions Critiques Accordées Automatiquement

| Permission                      | Capacité                              |
| ------------------------------- | -------------------------------------- |
| `CAPTURE_VIDEO_OUTPUT`        | Capture d'écran vidéo en temps réel |
| `READ_FRAME_BUFFER`           | Capture d'écran image                 |
| `INJECT_EVENTS`               | Simule touches / contrôle à distance |
| `READ_CALL_LOG`               | Accès au journal d'appels             |
| `DELETE_PACKAGES`             | Désinstaller n'importe quelle app     |
| `SYSTEM_ALERT_WINDOW`         | Overlay sur toutes les apps            |
| `INTERACT_ACROSS_USERS_FULL`  | Accès multi-utilisateur               |
| `READ_PRIVILEGED_PHONE_STATE` | Données téléphoniques complètes    |
| `DUMP`                        | Dump système complet                  |
| `QUERY_ALL_PACKAGES`          | Liste toutes les apps installées      |
| `INTERNET`                    | Accès réseau                         |

---

## Flux d'Activation

```
Technicien Samsung
    → Infrastructure Samsung
        → OMC Agent (Intent avec serveraddr + conncode)
            → RSSupport AAS2 se connecte à rc-samsung-global-win.rsup.io
                → Session de contrôle à distance établie
```

L'application **ne peut pas s'activer seule** — elle nécessite un déclencheur depuis l'infrastructure Samsung. Cependant, Samsung peut l'activer à distance à tout moment, sans action physique de l'utilisateur au moment de la connexion.

---

## Portée

OMC Agent est présent sur **tous les appareils Samsung** (firmware de base). RSSupport n'est pas déployé sur tous les marchés — Samsung contrôle la liste des applications à installer côté serveur.

| Appareil       | Marché                      | RSSupport   |
| -------------- | ---------------------------- | ----------- |
| SM-A175F (×3) | International (suffixe`F`) | ✅ Présent |
| SM-G991N       | Corée (KT)                  | ❌ Absent   |
| SM-S921B       | International (suffixe`B`) | ❌ Absent   |

### Présence Multi-Fabricants

L'analyse du code décompilé de l'APK révèle que RSSupport commercialise son infrastructure à **plusieurs fabricants majeurs** :

| Package OEM | Fabricant |
|---|---|
| `com.rsupport.rs.activity.sec` | **Samsung** |
| `com.rsupport.rs.activity.lge` | **LG** |
| `com.rsupport.rs.activity.oneplus` | **OnePlus** |
| `com.rsupport.rs.activity.meizu` | **Meizu** |
| `com.rsupport.rs.activity.tcl` | **TCL** |
| `com.rsupport.rs.activity.kt` | **KT** (opérateur coréen) |
| `com.rsupport.rs.activity.qihoo360` | **Qihoo 360** |

Ce rapport se concentre sur Samsung. La présence de variants pour d'autres fabricants suggère que des mécanismes similaires pourraient exister sur d'autres appareils Android — à vérifier indépendamment.

---

## Mitigation

```bash
# Désinstaller RSSupport AAS2
adb shell pm uninstall --user 0 com.rsupport.rs.activity.rsupport.aas2
```

---

## Statut du Responsible Disclosure

- [X] Analyse forensic complète
- [X] Reproduit sur 3 appareils indépendants
- [X] Responsible disclosure envoyé à Samsung (2026-09-19 — aucune réponse)
- [ ] Réponse Samsung reçue
- [X] Publication publique

---

## Travaux Connexes

| Sujet                                                        | Source                | Couverture                              |
| ------------------------------------------------------------ | --------------------- | --------------------------------------- |
| AppCloud/ironSource pré-installé Samsung                   | Forbes, nov. 2025     | Large                                   |
| Qihoo 360 pré-installé Samsung                             | Forbes, jan. 2020     | Large                                   |
| Spyware LANDFALL CVE-2025-21042                              | TechCrunch, nov. 2025 | Large                                   |
| **Mécanisme installation silencieuse RSSupport AAS2** | **Ce rapport**  | **Non documenté précédemment** |

---

*Outils : ADB + jadx 1.5.6 + aapt | SyliSec 2026*
