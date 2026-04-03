# Project Talend ETL — SIO vers SID

Projet SAE SID : pipeline ETL complet réalisé avec **Talend Open Studio**, migrant les données d'un **Système d'Information Opérationnel (SIO)** vers un **Système d'Information Décisionnel (SID)** sous PostgreSQL.

---

## Contexte

Dans le cadre de la SAE (Situation d'Apprentissage et d'Évaluation) du module **Systèmes d'Information Décisionnels**, ce projet consiste à concevoir et implémenter un pipeline ETL (Extract, Transform, Load) permettant d'alimenter un entrepôt de données (data warehouse) à partir d'une base de données opérationnelle.

---

## Architecture

```
SIO (Sio_DB - PostgreSQL)
   ├── _client        (codecli, societe, fonction, ville, pays)
   ├── _produit       (refprod, nomprod, nofour, codecateg)
   └── _categorie     (codecateg, nomcateg, description)
            │
            │  ETL Talend Open Studio
            ▼
SID (PostgreSQL - Data Warehouse)
   ├── dim_client
   ├── dim_produit
   └── (tables de faits)
```

---

## Stack technique

| Composant | Technologie |
|-----------|------------|
| ETL | Talend Open Studio 8.x |
| Base source (SIO) | PostgreSQL 17 |
| Base cible (SID) | PostgreSQL 17 |
| Langage généré | Java |

---

## Structure du projet

```
.
├── LOCAL_PROJECT/                        # Projet Talend principal
│   ├── process/
│   │   └── SIO_vers_SIO_0.1.item        # Job : nettoyage intra-SIO
│   ├── metadata/connections/
│   │   ├── Conexion_SIO_0.1.item        # Connexion BDD source
│   │   └── connexion_SID_0.1.item       # Connexion BDD cible
│   └── code/routines/system/            # Routines Talend système
│
├── SIO_SID/                             # Projet intermédiaire
│   └── process/
│       └── Sio_vers_Sid_0.1.item        # Job ETL v1
│
├── SIO_VERS_SID/                        # Projet final (job principal)
│   └── process/
│       └── job_Sio_vers_Sid_0.1.item    # Job ETL final (SIO → SID)
│
└── README.md
```

---

## Jobs Talend

### 1. `SIO_vers_SIO` — Nettoyage intra-opérationnel
- Lecture de la table `_client` depuis la base SIO
- Transformation et réécriture dans la même base
- Composants : `tPostgresqlInput` → `tPostgresqlOutput`

### 2. `Sio_vers_Sid` — ETL v1
- Extraction de `_client` depuis SIO
- Chargement dans la base SID
- Première itération du pipeline ETL

### 3. `job_Sio_vers_Sid` — ETL final (job principal)
**Extraction (tPostgresqlInput x3) :**
```sql
-- Table client
SELECT codecli, societe, fonction, ville, pays FROM public._client

-- Table produit + categorie (jointure)
SELECT p.refprod, p.nomprod, p.nofour, p.codecateg,
       c.nomcateg, c.description
FROM public._produit p
JOIN public._categorie c ON p.codecateg = c.codecateg
```
**Chargement :** vers les dimensions du SID via `tPostgresqlOutput` x2

---

## Connexions base de données

| Paramètre | SIO | SID |
|-----------|-----|-----|
| Hôte | localhost | localhost |
| Port | 5432 | 5432 |
| Driver | PostgreSQL JDBC | PostgreSQL JDBC |
| Base | `Sio_DB` | SID |
| Utilisateur | postgres | postgres |

> **Securité** : Les mots de passe sont chiffrés dans les fichiers `.item` via le système de chiffrement Talend (system key v1). Ne jamais versionner de credentials en clair.

---

## Prise en main

### Prérequis
- Talend Open Studio 8.x
- PostgreSQL 14+
- Java 11+

### Import du projet
1. Ouvrir Talend Open Studio
2. `File > Import > Existing Projects into Workspace`
3. Sélectionner le dossier `SIO_VERS_SID/` (projet final)
4. Configurer les connexions dans `Metadata > Db Connections` :
   - Mettre à jour host/port/password pour `Conexion_SIO` et `connexion_SID`
5. Ouvrir le job `job_Sio_vers_Sid` et l'exécuter

### Ordre d'exécution recommandé
```
1. SIO_vers_SIO       → nettoyage des données sources
2. Sio_vers_Sid       → test ETL simplifié
3. job_Sio_vers_Sid   → ETL complet SIO → SID
```

---

## Auteur

**Mouhamed Moustapha Ndiaye**  
Étudiant en BUT Science des Données  
[GitHub](https://github.com/moustxf05)
