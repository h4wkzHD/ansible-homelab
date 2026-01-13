# Ansible Homelab

# Pourquoi ce projet existe

À l’origine, ce repo n’avait pas vocation à “refaire” tout mon homelab ou à automatiser chaque détail.

L’idée était beaucoup plus pragmatique :
avoir un vrai plan de secours le jour où mon serveur me lâche complètement.

Je voulais :

des sauvegardes régulières et fiables, sans y penser,

stockées hors du serveur (via Restic + AWS),

et surtout pouvoir restaurer rapidement sans tout refaire à la main.

Ansible est venu naturellement pour orchestrer tout ça :

déployer les services (Docker, BookStack , ect..),

gérer la configuration,

et me permettre de reconstruire un environnement fonctionnel en peu de temps.

Ce repo représente donc un PRA personnel (Plan de Reprise d’Activité) pour mon homelab :
Ce n’est pas un repo “clé en main” universel, c’est mon homelab, structuré proprement, mais avec mes choix.

---

## Stack technique

* **Ansible** : orchestration et idempotence
* **Docker / Docker Compose** : services
* **Restic** : sauvegarde et restauration
* **Ansible Vault** : gestion des secrets
* **GitHub** : versioning

Services principaux :

* BookStack
* MariaDB
* Restic (backups distants)

---

## Structure du dépôt

```text
├── ansible.cfg
├── group_vars
│   └── homelab.yml
├── inventory
│   └── hosts.yml
├── playbooks
│   ├── restore.yml
│   ├── setup.yml
│   └── site.yml
└── roles
    ├── bookstack
    │   ├── handlers
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── docker-compose.yml.j2
    ├── common
    │   ├── handlers
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    ├── cron
    │   ├── handlers
    │   ├── tasks
    │   └── templates
    ├── docker
    │   ├── handlers
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    └── restic
        ├── handlers
        ├── tasks
        └── templates
            ├── main.yml
            └── restic.env.j2
```

Chaque rôle fait **une chose précise**, sans logique cachée.

---

## Gestion des secrets (important)

Aucun secret n’est stocké en clair dans le repo.

* Tous les mots de passe, clés API, credentials cloud sont dans :

  ```text
  group_vars/homelab/vault.yml
  ```
* Ce fichier est chiffré avec **Ansible Vault**
* Le repo peut être public **sans fuite de secrets**

Commandes utiles :

```bash
ansible-vault edit group_vars/homelab/vault.yml
ansible-vault view group_vars/homelab/vault.yml
```

---

## Playbooks

### `site.yml`

Point d’entrée principal. Il appelle les autres playbooks et rôles.

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass
```

---

### `setup.yml`

Prépare la machine :

* Docker
* dépendances système
* structure des dossiers
* services nécessaires au homelab

Idempotent : peut être relancé sans risque.

---

### `restore.yml` – restauration sécurisée

C’est volontairement le playbook **le plus strict** du repo.

Principes :

* aucune restauration automatique
* interaction obligatoire
* impossible de restaurer `/`

Au lancement, Ansible demande explicitement **quel chemin restaurer** :

```bash
ansible-playbook playbooks/restore.yml --ask-vault-pass
```

Exemples valides :

* `/home/dylan/docker`
* `/var/lib/docker/volumes`
* `/etc`

Pourquoi autant de garde-fous ?
Parce qu’un `restic restore /` mal maîtrisé peut littéralement casser un système.

Ici, le but est d’avoir un **outil de reprise**, pas une arme.

---

## Sauvegardes

Les sauvegardes sont gérées par Restic :

* repository externe
* variables d’environnement chargées via `/etc/restic/.env`
* credentials jamais stockés en clair

Le restore utilise exactement la même source, garantissant la cohérence.

---

## Git & sécurité

* Le repo est versionné proprement
* Aucun secret n’est commit
* Authentification GitHub via **SSH** ou **token** (pas de mot de passe)

Ce repo est pensé pour être partagé **sans honte ni risque**.

---

## Philosophie

Ce projet n’a pas pour but d’être parfait.

Il a pour but d’être :

* lisible
* explicable
* maintenable dans 6 mois

Si je ne comprends pas ce que fait un rôle en le relisant plus tard, c’est qu’il est mal écrit.

---

Si tu lis ce README et que tu te dis :

> « ok, je vois pourquoi chaque choix a été fait »

alors le repo remplit son rôle.


Enjoy ^^
