[README.md](https://github.com/user-attachments/files/30011410/README.md)
# Teleport-Access-Management
Gestion centralisée des accès dans une infrastructure distribuée avec Teleport

## Présentation du projet

Ce dépôt documente l'implémentation d'une solution de **gestion centralisée des accès** basée sur **Teleport** dans une infrastructure distribuée, réalisée dans le cadre du mémoire de fin de cycle en Sécurité Informatique à l'IFRI, Université d'Abomey-Calavi, Bénin.

L'objectif : remplacer les accès SSH directs et non contrôlés par une solution moderne alignée sur les principes du **moindre privilège**, de la **traçabilité complète** et de l'**authentification forte**.

---

## Architecture de l'infrastructure

![Architecture](screenshots/architecture.png)

| Composant | Détail |
|---|---|
| Hyperviseur | VirtualBox |
| Serveur Teleport | VM Linux (Proxy + Auth Server) |
| SSH Node 1 | Serveur SSH — accès admin et standard |
| SSH Node 2 | Serveur SSH — serveur de production (accès JIT uniquement) |
| Serveur Web | Nginx |
| Base de données | MySQL |
| MFA | OTP via Google Authenticator |

---

## Fonctionnalités implémentées

- **RBAC** — Contrôle d'accès basé sur les rôles : chaque utilisateur accède
  uniquement aux ressources autorisées par son rôle
- **Just-In-Time Access (JIT)** — Accès temporaire de 30 minutes au serveur de
  production, accordé après soumission et approbation d'une requête explicite
- **MFA** — Authentification forte via OTP (Google Authenticator)
- **Audit & Traçabilité** — Journalisation complète de toutes les actions
- **Session Recording & Replay** — Enregistrement et relecture des sessions SSH
- **Accès différencié** — 3 types de ressources contrôlées (SSH, Web, DB)
  avec des niveaux d'accès distincts selon les rôles

---

## Rôles RBAC configurés

| Rôle | Ressource accessible | Détail |
|---|---|---|
| `admin` | SSH-Sandbox | Privilèges élevés pour utilisateur administrateur |
| `role-standard` | SSH-Sandbox | Privilèges minimaux pour utilisateur standard |
| `marketing` | Nginx (serveur web) | Accès dédié au serveur web uniquement |
| `jit-request` | — | Permet de soumettre une demande d'accès JIT |
| `jit-access` | File-server (production) | Accès temporaire 30 min avec privilèges élevés, accordé après approbation |

*Détail des rôles disponibles dans `config/roles/`*

---

## Flux d'accès Just-In-Time (JIT)

```
Utilisateur (rôle jit-request)
        │
        ▼
  Soumet une Access Request via Teleport
        │
        ▼
  Approbation par un administrateur
        │
        ▼
  Rôle jit-access accordé temporairement (30 min)
        │
        ▼
  Accès File-server (production) avec privilèges élevés
        │
        ▼
  Expiration automatique, accès révoqué
```

---

## Structure du dépôt

```
Teleport-Access-Management/
├── docs/
│   └── memoire.pdf                # Document de mémoire complet
├── config/
│   ├── teleport.png               # Configuration principale de Teleport
    ├── ssh-sandbox.png            # Configuration du noeud ssh 1
    ├── file-server.png            # Configuration du noeud ssh 2
    ├── app_web.png                # Configuration du serveur web
    ├── db-service.png             # Configuration de MySQL
│   └── roles/                     # Définition des rôles RBAC
│       ├── admin.png
│       ├── role-standard.png
│       ├── marketing.png
│       ├── jit-request.png
│       └── jit-access.png
└── screenshots/
    ├── architecture.png             # Schéma d'architecture
    ├── dashboard.png                # Interface Teleport
    ├── users.png                    # Utilisateurs Teleport
    ├── mfa.png                      # Authentification forte
    ├── request-create.png           # Requête d'accès JIT
    ├── approbation.png              # Approbation de la requête JIT
    ├── approved.png                 # Approbation JIT reçue
    ├── jit-expired.png              # Expiration du JIT
    ├── session-recording.png        # Enregistrement de session
    ├── replay.png                   # Relecture de session
    └── logs.png                     # Journaux d'audit
```

---

## Difficultés rencontrées et solutions

**Intégration Keycloak échouée**
Tentative de délégation de la gestion des identités à Keycloak (SSO).
Teleport a rejeté les certificats auto-signés de Keycloak lors de la configuration OIDC.
→ Résolution partielle : la fonctionnalité a été abandonnée pour ce projet.

**Session JIT ne se fermait pas après 30 minutes**
L'accès temporaire restait actif au-delà de la durée prévue.
→ Solution : activation du paramètre `max_session_ttl` dans la définition
du rôle `jit-access`, paramètre non activé par défaut.

**PostgreSQL non enrôlé**
Tentative d'enrôlement d'une base PostgreSQL comme ressource Teleport.
Échec dû à un problème de certificats mTLS (mutual TLS) entre Teleport et PostgreSQL.
→ Alternative : MySQL utilisé à la place, dont l'enrôlement n'a pas posé de difficulté.

---

## Contexte académique

Ce projet constitue le travail de fin de cycle pour l'obtention du diplôme de licence en **Sécurité Informatique** à l'**IFRI — Institut de Formation et de Recherche en Informatique**, Université d'Abomey-Calavi, Bénin.

---

## Auteure
**Safirath**
Diplômée en Sécurité Informatique — IFRI, Université d'Abomey-Calavi, Bénin
Certification Google IT Support
En recherche de stage en Réseaux et Sécurité à Cotonou
https://www.linkedin.com/in/safirath-bakary-89844a208
