# PROCEDURE POUR APPLIQUER LE PATCH 1.2.4-2 A PARTIR DU PATCH 1.2.4-1

# Nouvelles Images du patch 1.2.4

## 🔹 API
- `docker.io/dcptechnologies/api:1.2.4-2`
- `docker.io/dcptechnologies/logwatcher:1.2.4-2`
- `docker.io/dcptechnologies/podwatcher:1.2.4-2`
- `docker.io/dcptechnologies/health:1.2.4-2`

## 🔹 PROXY
- `docker.io/dcptechnologies/proxy:1.0.0-v3`

## 🔹 SPARK
- `docker.io/dcptechnologies/spark:sp-350-124-v1`
- `docker.io/dcptechnologies/notebook:nb-350-124-v1`

## 🔹 SQL
- `docker.io/dcptechnologies/sql:hbase`
- `docker.io/dcptechnologies/sql:470`
- `docker.io/dcptechnologies/sql:hms-3.1.2-v3`
- `docker.io/dcptechnologies/sql:policy-1.2.4-2`

## 🔹 DASK
- `docker.io/dcptechnologies/dask:2024.9.1-py3.11`
- `docker.io/dcptechnologies/dask:24.10-cuda12.5-py3.12`
- `docker.io/dcptechnologies/dask:nb-sp350-dask2024.8.0`
- `docker.io/dcptechnologies/dask:nb-gpu-sp350-dask2024.8.0`

## 🔹 Dbt
- `dcptechnologies/dbt:proxy-1.9.2-1`
- `docker.io/dcptechnologies/dbt:storageinitializer-1.2.4`

## 🔹 File manager
- `docker.io/dcptechnologies/filemanager:1.2.4`

## 🔹 Démo Spark GPU
- `docker.io/dcptechnologies/demo-spark-gpu:1.2.4`

## 🔹 Airflow DAG Editor
- `docker.io/dcptechnologies/airflow:editor`

---

# Vérification de la Sauvegarde de la Base de Données  
Avant toute mise à jour, **s'assurer que la base de données a bien été sauvegardée**.

---


# Installation du Package
1. **Télécharger** le package d'installation.
2. **Copier** les répertoires `env` et `certs` de l'ancien package d'installation.
3. **Modifier** le fichier `env.sh` :  
    ```sh
    API_VERSION=1.2.4-2
    ```
4. Modifier le fichier env-api.sh
    ```sh
    MASTER_API_VERSION=$API_VERSION
    ```
5. **Modifier** le fichier `env-podwatcher.sh` :
    ```sh
    PODWATCHER_CLEANCONFIGMAP=true
    PODWATCHER_CLEANSERVICE=true
    PODWATCHER_CLEANVOLUME=true
    ```
6. **Modifier** le fichier `env-scheduler.sh` :
    ```sh
    SCHEDULER_REQUESTCPU=4
    SCHEDULER_LIMITCPU=4
    SCHEDULER_REQUESTMEMORY=10Gi
    SCHEDULER_LIMITMEMORY=10Gi
    GOGC=200
    GOMEMORYLIMIT=8GiB
    ```
6. **Modifier** le fichier `env-proxy.sh`:
    Activer le SSL et renseigner la nouvelle version du proxy: chercher et modifier les  paramètre:
    ```sh
    PROXY_VERSION=1.0.0-v3
    PROXY_SSL_ENABLED=$SSL_ENABLED
    ```
    Ajouter nouveaux paramètres
    ```sh
    DCPRANDOMUUID="xWFlmUb8Jxbz"
    DCPRANDOMUUIDLABEL="dcprandomuuid"
    MASTER_CLUSTER_ROLE=$CLUSTERROLE
    MASTER_NAMESPACES=$ROLE_NAMESPACES
    ```
7. Copier les certificats de l'ancien package d'installation dans :
    ```
    repos/master-api/files
    ```

---

# Arrêter les Services DCP et le PROXY DCP
Avant d’arrêter les services DCP, vérifier qu’aucun traitement Spark n’est en cours.
Ensuite, exécuter la commande suivante :

```sh
helm -n dcp uninstall dcpapi dcphealth dcplog dcppodwatcher-primary dcppodwatcher-backup dcppodwatcher-secondary masterproxy
```

---

# Upgrade de la base de données
```sh
./upgradedatabase-1.2.4-2.sh
```
Vérifier les log du job "upgrade-postgresql-job-124-2"

---

# Installation et Mise à Jour des Composants

## 🔹 Installer l’API

```sh
./09-api.sh
```

## 🔹 Mettre à jour les modèles

```sh
./10-models.sh
```

## 🔹 Mettre à jour les repositories
```sh
./11-repos.sh
```

## 🔹 Installer le Log Watcher
```sh
./12-logwatcher.sh
```

## 🔹 Installer le Pod Watcher

```sh
./13-podwatcher.sh primary
./13-podwatcher.sh secondary
./13-podwatcher.sh backup
```

# Installer le health-checker
```sh
./14-health.sh
```

# Installer le proxy
```sh
./06-proxy.sh
```
Consulter les logs du du pod "proxy", afin de vérifier que le service est démarré en SSL:
```
Démarrage du serveur HTTPS sur le port 8080...
```

---

# Configuration Globale
1. Se rendre dans le menu Admin.
2. Aller dans Kubernetes. et seléctionner ***local*** dans la liste

## 🔹 Scheduler
Aller dans l'onglet "Scheduler
Activer l'option "Queue # from tenant"

## 🔹 Activer SSL pour Dask
Aller dans l'onglet "Dask"
- Activer SSL pour Dask en cochant l'option "Enable SSL"


## 🔹 Activer SSL pour Dbt
Aller dans l'onglet "Dbt"
- Activer SSL pour Dbt en cochant l'option "Enable SSL"

## 🔹 Spark
Aller dans l'onglet ***Spark***
Aller dans l'onglet ***Spark History***
 - Activer SSL pour Spark History en cochant l'option "Enable TLS"
Aller dans l'onglet ***Spark Monitoring***
 - Activer SSL pour Spark Monitoring en cochant l'option "Enable TLS"
Aller dans l'onglet ***Spark Notebook***
 - Activer SSL pour Spark Notebook en cochant l'option "Enable TLS" 

## 🔹 Airflow
Aller dans l'onglet ***Airflow***
 - Activer SSL pour Airflow en cochant l'option "Enable TLS"
 - Dans l'onglet ***Dag Editor*** saisir la valeur 4 dans les champs:
    ```
    Minimum Core
    Minimum Memory
    Maximum Core
    Maximum Memory
    ```
 - Dans l'onglet ***Metrics Storage*** saisir la valeur 4 dans les champs:
    ```
    Request CPU = 2
    Request Memory = 2
    Limit CPU = 2
    limit Memory = 2
    ```
 - Dans l'onglet ***Sync Config***, saisir les valeurs suivantes dasn chaque champ
    ```
    Request CPU = 1
    Request Memory = 1
    Limit CPU = 2
    limit Memory = 2
    ```
 - Dans l'onglet ***Dashboard Config***: 
    1. Activer SSL pour le monitoring Airflow en cochant l'option "Enable SSL"
    2. saisir les valeurs suivantes dasn chaque champ
    ```
    Request CPU = 2
    Request Memory = 2
    Limit CPU = 2
    limit Memory = 2
    ```

## 🔹 GPU
Aller dans l'onglet ***GPU*** et utiliser le bouton "Add provide/label" pour ajouter les labels mig pour gpu.

# PARAMS
## 🔹 GPU
Aller dans l'onglet ***Global***
Vérifier que le champ "bucketprovider" contient la valeur Scality
```
bucketprovider='Scality'
```

Vérifier que le champ "Uid" contient la valeur xWFlmUb8Jxbz
```
Uid='xWFlmUb8Jxbz'
```


Vérifier que les options suivantes sont activées:
```
onlyadmincanmodifysc
showsource
checkserviceauthbydata
```

## 🔹 Repos
Aller dans l'onglet ***Global***
```
mlflowrepo: mlflow
mlprojectflaskagent: mlflow
daskrepo: dask
filemanagerrepo: dcpfm
```


# Hive Metastore
1. Se rendre dans le menu ***Sql***.
2. Se rendre dans le menu ***Metastore***
Le service ***Metastore*** ne supporte plus de prendre en charge plusieurs buckets. 
Sélectionner chaque metastore et sélectionner la bucket à lui associer.

# PATCH
```
Avant d'appliquer le patch des service "Sql policy", s'assurer que tous ces services sont démarrées, sinon le patch ne s'appliquera pas correctement
```
Patch Sql Policy
```

# Sql Policy
Redémarrer tous les services ***Sql Policy*** pour utiliser la dernière version qui apporte des améliorations
