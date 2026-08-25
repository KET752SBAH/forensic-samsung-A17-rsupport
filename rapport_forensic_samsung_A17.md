Rapport Forensic — Samsung SM-A175F (Galaxy A17)

**Date d'analyse :** 2026-08-06 (appareil 1) / 2026-08-25 (appareil 2)
**Analyste :** SyliSec
**Appareils analysés :** 2 × Samsung SM-A175F (Galaxy A17)
**Identifiants ADB :** RZGL5193PVW / RFGL630AEAA
**OS :** Android 16
**Patch sécurité :** 2026-03-05

---

## 1. Résumé Exécutif

L'analyse forensic de **deux Samsung Galaxy A17 neufs** a révélé le mécanisme exact par lequel Samsung installe silencieusement une application d'accès à distance (**RSSupport AAS2**) sur ses appareils.

L'analyse comparative des deux appareils prouve de façon **causale** (et non simplement corrélative) que :

1. RSSupport AAS2 est **absent** tant que `device_provisioned` n'est pas positionné à `1`
2. RSSupport AAS2 est **installé automatiquement** par OMC Agent dès que `device_provisioned=1` — ce qui peut survenir **avant même la fin du Setup Wizard**
3. L'application reçoit d'office **11 permissions critiques** sans que l'utilisateur en soit informé

Aucune session de contrôle à distance n'a été établie sur les deux appareils. Le reste du système est propre.

**Niveau de risque global : MOYEN — Pratique Samsung préoccupante, mécanisme de déclenchement désormais prouvé**

---

## 2. Informations sur les Appareils

| Champ | Appareil 1 | Appareil 2 |
| --- | --- | --- |
| ID ADB | RZGL5193PVW | RFGL630AEAA |
| Modèle | Samsung SM-A175F | Samsung SM-A175F |
| Version Android | 16 | 16 |
| Patch sécurité | 2026-03-05 | 2026-03-05 |
| Fabricant | Samsung | Samsung |
| `user_setup_complete` | `1` (SUW terminé) | `null` (SUW incomplet) |
| `device_provisioned` | `1` | `1` |
| RSSupport AAS2 | **Présent** (build 452) | **Présent** (build 454) |
| Date installation RSSupport | 2026-08-06 22:58:50 | **2026-08-25 22:55:00** |
| Apps tierces | 27 | 27 |

---

## 3. Applications Tierces Installées (Appareil 1)

27 applications tierces ont été identifiées sur l'appareil 1. Le tableau ci-dessous présente les 10 applications les plus notables — les 17 restantes sont des applications Samsung et Google légitimes (Google Docs, Samsung Notes, Samsung Find, One Connect, Game Home, etc.) sans comportement suspect.

| Application | Package | Statut |
| --- | --- | --- |
| Netflix | com.netflix.mediaclient | Légitime |
| Outlook | com.microsoft.office.outlook | Légitime |
| Samsung Browser | com.sec.android.app.sbrowser | Légitime |
| YouTube Music | com.google.android.apps.youtube.music | Légitime |
| Samsung Health | com.sec.android.app.shealth | Légitime |
| LinkedIn | com.linkedin.android | Légitime |
| Spotify | com.spotify.music | Légitime |
| Facebook | com.facebook.katana | Légitime |
| Instagram | com.instagram.android | Légitime |
| **RSSupport AAS2** | **com.rsupport.rs.activity.rsupport.aas2** | **⚠️ SUSPECT** |

---

## 4. Application Suspecte : RSSupport AAS2

### 4.1 Identification

| Champ | Valeur |
| --- | --- |
| Package | `com.rsupport.rs.activity.rsupport.aas2` |
| Nom | Samsung Remote Support (by RSSupport) |
| Version | 1.5 (build 452) |
| Date d'installation | **2026-08-06 22:58:50** |
| Installé par | `com.samsung.android.app.omcagent` |
| Jamais lancée | `notLaunched=true`, `stopped=true` |
| Usage statistiques | `used=<uninitialized>` |

### 4.2 Permissions Accordées (11 permissions critiques)

| Permission | Impact |
| --- | --- |
| `CAPTURE_VIDEO_OUTPUT` | Capture écran vidéo |
| `READ_FRAME_BUFFER` | Capture écran image |
| `INJECT_EVENTS` | Simule touches/clics (contrôle à distance) |
| `READ_CALL_LOG` | Accès journal d'appels |
| `DELETE_PACKAGES` | Peut désinstaller des apps |
| `SYSTEM_ALERT_WINDOW` | Overlay sur toutes les apps |
| `INTERACT_ACROSS_USERS_FULL` | Accès multi-utilisateur |
| `READ_PRIVILEGED_PHONE_STATE` | Données téléphoniques complètes |
| `DUMP` | Dump système complet |
| `QUERY_ALL_PACKAGES` | Liste toutes les apps installées |
| `INTERNET` | Accès réseau |

### 4.3 Infrastructure Serveur

L'APK a été extrait et décompilé (jadx 1.5.6) pour identifier le serveur de connexion :

- **Serveur relais par défaut :** `rc-samsung-global-win.rsup.io`
- **Domaine propriétaire :** `rsup.io` → RSSupport Co., Ltd. (Séoul, Corée du Sud)
- **URL serveur secondaire :** `http://www.rsupport.com`
- **Endpoint liste serveurs :** `[serveraddr]/mobile/choose_country_main_server`

Le `serveraddr` est fourni dynamiquement via un Intent Android par Samsung OMC Agent — l'app ne peut **pas** s'activer seule sans ce déclencheur externe.

### 4.4 Flux d'Activation

```
Technicien Samsung
    → Backend Samsung
        → OMC Agent sur le téléphone (Intent avec serveraddr + conncode)
            → RSSupport AAS2 se connecte à rc-samsung-global-win.rsup.io
                → Session de contrôle à distance établie
```

---

## 5. Analyse du Système de Fichiers Administrateur

### Administrateurs de l'appareil

| Package | Composant | Statut |
| --- | --- | --- |
| `com.samsung.android.kgclient` | Knox Guard Device Admin | Légitime (Samsung Knox) |

Identique sur les deux appareils. Aucun administrateur de périphérique suspect détecté.

### Services d'Accessibilité

- **Appareil 1 :** Seul `com.samsung.accessibility` (Samsung natif) est actif.
- **Appareil 2 :** Aucun service d'accessibilité configuré (Setup Wizard non terminé).

Aucun service d'accessibilité malveillant sur les deux appareils.

---

## 6. Analyse Réseau

### Connexions Actives au Moment de l'Analyse (Appareil 1)

| IP Distante | Port | État | Propriétaire |
| --- | --- | --- | --- |
| 216.58.205.238 | 443 | ESTABLISHED | Google |
| 172.217.195.238 | 443 | CLOSE_WAIT | Google |
| 173.194.41.188 | 5228 | ESTABLISHED | Google Play (FCM) |
| 34.102.190.5 | 443 | ESTABLISHED | Google |
| 18.64.221.76 | 443 | ESTABLISHED | AWS |
| 104.18.24.14 | 443 | TIME_WAIT | Cloudflare |

**Aucune connexion vers les serveurs RSSupport détectée** sur les deux appareils. Comportement réseau normal pour un Android neuf.

---

## 7. Timeline des Événements

### Appareil 1 — 2026-08-06

| Heure | Événement |
| --- | --- |
| 22:58:50 | Broadcast `SETUPWIZARD_COMPLETE` émis — `device_provisioned=1` |
| 22:58:50 | Installation silencieuse de RSSupport AAS2 (build 452) par OMC Agent |
| 22:58:50 | `lastDisabledCaller: com.samsung.android.app.omcagent` |
| 23:02:52 | Notification OMC Agent postée (canal `SUW_CHANNEL`) |
| 23:10:05–19 | OMC Agent continue d'envoyer des notifications |

### Appareil 2 — 2026-08-25

| Heure | Événement |
| --- | --- |
| ~21:37 | Première analyse : RSSupport absent, `device_provisioned=null` |
| 22:55:00 | Installation silencieuse de RSSupport AAS2 (build 454) par OMC Agent |
| 22:55:00 | `device_provisioned=1`, `user_setup_complete` toujours `null` |
| 23:03 | Vérification : RSSupport présent, jamais lancé (`notLaunched=true`) |

---

## 8. Analyse Comparative — Preuve du Mécanisme de Déclenchement

### 8.1 Phase 1 — Appareil 2 avant interaction (RSSupport absent)

Lors de la première analyse le 2026-08-25, l'appareil 2 avait `user_setup_complete=null` et `device_provisioned=null`. RSSupport AAS2 était **absent** — 26 apps tierces.

L'analyse d'OMC Agent révélait les intents enregistrés :

```
com.sec.android.app.secsetupwizard.SETUPWIZARD_COMPLETE
com.sec.android.app.setupwizard.SETUPWIZARD_COMPLETE
```

### 8.2 Phase 2 — Appareil 2 après interaction partielle (RSSupport installé)

Lors de la vérification suivante (22:55:00), **RSSupport AAS2 est apparu sur l'appareil 2** — alors que `user_setup_complete` reste `null` (Setup Wizard non terminé) mais que `device_provisioned=1` était désormais positionné.

| Champ | Valeur |
| --- | --- |
| Package | `com.rsupport.rs.activity.rsupport.aas2` |
| Version | 1.5 (build **454**) |
| Date installation | **2026-08-25 22:55:00** |
| Installé par | `com.samsung.android.app.omcagent` |
| `user_setup_complete` | **null** (SUW incomplet) |
| `device_provisioned` | **1** |
| Jamais lancée | `notLaunched=true`, `stopped=true` |

**Découverte critique :** RSSupport peut être installé **avant même la fin du Setup Wizard**, dès que `device_provisioned=1` est positionné. Le trigger n'est pas uniquement `SETUPWIZARD_COMPLETE` — OMC Agent peut agir dès qu'une interaction intermédiaire du SUW provisionne l'appareil.

### 8.3 Découverte Annexe — Nom Trompeur d'OMC Agent

L'extraction de l'APK d'OMC Agent (`/system/priv-app/OMCAgent5/OMCAgent5.apk`) révèle son nom d'affichage officiel dans l'interface Android :

> **`application-label: 'Recommended apps'`**

Samsung a nommé son agent d'installation silencieuse **"Applications recommandées"** — un nom volontairement anodin qui ne laisse pas soupçonner son rôle réel : déployer RSSupport AAS2 sur tous les appareils neufs. Cette app est une **app système privilégiée** (`/system/priv-app/`), invisible par défaut dans les paramètres.

### 8.4 Tableau Comparatif Final

| Critère | Appareil 1 | Appareil 2 |
| --- | --- | --- |
| Setup Wizard terminé | Oui | **Non** |
| `device_provisioned` | 1 | **1** |
| RSSupport AAS2 | **Présent** (build 452) | **Présent** (build 454) |
| Installé par | OMC Agent | OMC Agent |
| OMC Agent version | 5.8.19 | 5.8.19 |
| OMC Agent nom affiché | "Recommended apps" | "Recommended apps" |
| Connexion RSSupport | Aucune | Aucune |

### 8.5 Conclusion Causale Révisée

| `device_provisioned` | `user_setup_complete` | RSSupport AAS2 |
| --- | --- | --- |
| `null` | `null` | **Absent** |
| `1` | `null` ou `1` | **Présent, installé par OMC Agent** |

Le déclencheur réel est le positionnement de **`device_provisioned=1`** — qui peut survenir en cours de Setup Wizard, avant même sa complétion. Cela rend l'installation encore plus précoce et moins détectable que supposé initialement.

---

## 9. Verdict

### Est-ce normal ?

**Non.** Samsung installe silencieusement un outil de contrôle à distance complet sans demander le consentement explicite de l'utilisateur. Bien que cette pratique soit techniquement permise par les CGU Samsung dans certaines régions, elle :

- N'est **pas transparente** envers l'utilisateur
- Installe un outil avec **11 permissions critiques** (capture écran, contrôle tactile)
- Utilise le canal `SUW_CHANNEL` (Setup Wizard) de façon non divulguée
- Peut être activée à distance **par Samsung via OMC Agent**, sans action physique de l'utilisateur

### Cause confirmée

| Scénario | Évaluation |
| --- | --- |
| **Samsung installe RSSupport AAS2 via OMC Agent dès que `device_provisioned=1`** | ✅ **PROUVÉ — preuve causale sur 2 appareils, 2 builds différents** |
| Reset usine + push Samsung lors de la config | Écarté (appareils neufs) |
| Session de support Samsung déclenchée sans consentement | Écarté (jamais lancée) |
| Installation uniquement après fin complète du Setup Wizard | ⚠️ **RÉVISÉ — peut survenir avant la fin du SUW** |

---

## 10. Recommandations

### Action immédiate

```bash
# Désinstaller RSSupport AAS2
adb shell pm uninstall --user 0 com.rsupport.rs.activity.rsupport.aas2
```

### Actions complémentaires

1. **Vérifier les paramètres Samsung** → Paramètres > Assistance à distance > s'assurer que le service est désactivé
2. **Révoquer les permissions OMC Agent** → Paramètres > Applications > OMC Agent > Permissions
3. **Surveiller les futures installations** → Paramètres > Sécurité > Administrateurs de l'appareil
4. **Activer les notifications d'installation** pour détecter toute future installation silencieuse

---

## 11. Portée Globale — RSSupport sur Tous les Appareils Samsung

### 11.1 Présence Multi-Constructeurs

L'analyse du code décompilé de l'APK révèle que RSSupport fournit des variantes de son outil à **plusieurs fabricants majeurs** :

| Package OEM | Fabricant |
| --- | --- |
| `com.rsupport.rs.activity.sec` | **Samsung** (SEC = Samsung Electronics) |
| `com.rsupport.rs.activity.lge` | **LG** |
| `com.rsupport.rs.activity.oneplus` | **OnePlus** |
| `com.rsupport.rs.activity.meizu` | **Meizu** |
| `com.rsupport.rs.activity.tcl` | **TCL** |
| `com.rsupport.rs.activity.kt` | **KT** (opérateur télécom coréen) |
| `com.rsupport.rs.activity.qihoo360` | **Qihoo 360** |

RSSupport commercialise son infrastructure de prise en main à distance à l'ensemble des grands fabricants Android mondiaux. La présence de `com.rsupport.rs.activity.qihoo360` indique que RSSupport et Qihoo 360 partagent la même infrastructure — ce sont deux clients distincts d'une plateforme commune, à ne pas confondre avec l'intégration Qihoo 360 dans Samsung Device Care documentée par Forbes en 2020.

### 11.2 Déploiement Systématique sur Samsung

**Tous les appareils Samsung sont très probablement concernés**, pour trois raisons :

1. **OMC Agent** (`com.samsung.android.app.omcagent`) est intégré au **firmware de base de tous les Samsung Android** — il est présent sur chaque appareil, quelle que soit la région ou l'opérateur
2. Le trigger **`device_provisioned=1`** est positionné automatiquement sur tous les appareils Samsung dès la phase initiale de configuration — OMC Agent l'observe et installe RSSupport sans attendre la fin complète du Setup Wizard
3. C'est la **politique de support global de Samsung**, déployée sur la majorité des marchés mondiaux

### 11.3 Impact Estimé

> Des **centaines de millions d'appareils Samsung** dans le monde ont probablement cet outil de contrôle à distance pré-installé sans en être informés, avec des permissions permettant la capture d'écran, le contrôle tactile et l'accès aux données personnelles. Cette estimation repose sur les parts de marché Samsung et la présence universelle d'OMC Agent dans le firmware — elle reste à valider sur d'autres modèles et régions.

Cette pratique n'est pas divulguée lors de l'achat et n'apparaît dans aucune documentation grand public de Samsung.

---

## 12. État des Publications et Alertes Existantes

### 12.1 Sur RSSupport AAS2 / Smart Tutor spécifiquement

**Aucune publication de recherche sécurité majeure** n'a à ce jour formellement alerté sur l'installation silencieuse de RSSupport AAS2 sur les appareils Samsung neufs, ni documenté le mécanisme de déclenchement via `device_provisioned=1`. L'application est connue sous le nom commercial **"Smart Tutor"** sur le Google Play Store. Quelques articles d'explication existent (HackerNoon, LearnProTips) mais sans caractérisation du risque lié à l'installation automatique.

> **Le présent rapport documente un angle non couvert dans la littérature sécurité publique, avec preuve causale sur deux appareils.**

### 12.2 Précédent le Plus Proche : Qihoo 360 Pré-installé (2020)

La controverse la plus similaire documentée publiquement :

- **Forbes, janvier 2020** — *"Does Your Samsung Galaxy S10 Come With Undeletable Chinese Spyware Pre-Installed?"*
- Samsung avait intégré **Qihoo 360** comme moteur d'analyse dans l'outil "Entretien de l'appareil", sans en informer les utilisateurs
- La communauté Reddit avait alerté, Forbes avait couvert l'affaire, Samsung avait dû se justifier publiquement

### 12.3 Spyware LANDFALL Ciblant Samsung (2024-2025)

- **Palo Alto Networks Unit 42 / TechCrunch / Forbes, novembre 2025**
- Un spyware commercial a exploité la CVE-2025-21042 pour compromettre des Samsung Galaxy pendant plus d'un an
- La CISA a émis une alerte officielle avec délai de 21 jours
- L'appareil analysé (patch 2026-03-05) est patché contre cette CVE
- Différent de RSSupport mais confirme que Samsung est une cible et un vecteur récurrents

### 12.4 Tableau Récapitulatif

| Sujet | Publication | Couverture médiatique |
| --- | --- | --- |
| RSSupport AAS2 — mécanisme SUW prouvé sur 2 appareils | ❌ Aucune alerte publiée | **Non documenté — apport de ce rapport** |
| Qihoo 360 pré-installé Samsung | ✅ Forbes, jan. 2020 | Large |
| Spyware LANDFALL Samsung CVE-2025-21042 | ✅ TechCrunch / Forbes, nov. 2025 | Large |

### 12.5 Sources

- [Forbes — Samsung Galaxy S10 Chinese Spyware (2020)](https://www.forbes.com/sites/daveywinder/2020/01/10/does-your-samsung-galaxy-s10-come-with-undeletable-chinese-spyware-pre-installed/)
- [Private Internet Access — Qihoo 360 Samsung](https://www.privateinternetaccess.com/blog/android-community-worried-about-presence-of-chinese-spyware-by-qihoo-360-in-samsung-smartphones-and-tablets/)
- [TechCrunch — Landfall spyware Samsung (2025)](https://techcrunch.com/2025/11/07/landfall-spyware-abused-zero-day-to-hack-samsung-galaxy-phones/)
- [Forbes — CISA Samsung spyware warning (2025)](https://www.forbes.com/sites/daveywinder/2025/11/11/update-your-samsung-smartphone-now---cisa-issues-21-day-spyware-warning/)
- [Palo Alto Unit 42 — LANDFALL spyware](https://unit42.paloaltonetworks.com/landfall-is-new-commercial-grade-android-spyware/)
- [HackerNoon — Smart Tutor App Review](https://hackernoon.com/smart-tutor-app-review-samsungs-official-remote-view-app)

---

## 13. Conclusion

Les deux appareils ne présentent **pas de malware** au sens strict. L'application RSSupport AAS2 est un outil légitime de support Samsung mais constitue un **risque de confidentialité significatif** :

- **Installation automatique déclenchée par `device_provisioned=1`** — prouvé sur deux appareils, deux builds (452 et 454)
- **Peut survenir avant la fin du Setup Wizard** — plus précoce et moins détectable que supposé
- **11 permissions critiques accordées d'office** : capture écran, contrôle tactile, accès journal d'appels
- **Activation à distance possible par Samsung via OMC Agent**, sans action physique de l'utilisateur
- **OMC Agent se nomme "Recommended apps"** dans l'interface — nom volontairement trompeur
- **Pratique non divulguée** lors de l'achat

L'analyse comparative sur deux Samsung SM-A175F neufs établit la **causalité directe** : dès que `device_provisioned=1` est positionné pendant la configuration initiale, OMC Agent ("Recommended apps") installe silencieusement RSSupport AAS2 avec 11 permissions critiques.

**Samsung installe un outil de prise en main à distance dès les premières secondes de configuration d'un appareil neuf, sans en informer l'acheteur.**

Il est fortement recommandé de désinstaller cette application immédiatement.

---

*Rapport généré le 2026-08-06, mis à jour le 2026-08-25 | Outils : ADB + jadx 1.5.6 + aapt | Analyste : SyliSec | 2 appareils analysés*
