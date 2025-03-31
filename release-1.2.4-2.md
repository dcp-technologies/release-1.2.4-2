# PROCEDURE POUR APPLIQUER LE PATCH 1.2.4-2 A PARTIR DU PATCH 1.2.3-3

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
    Vérfier les valeurs de ces paramètres
    ```sh
    SCHEDULER_REQUESTCPU=4
    SCHEDULER_LIMITCPU=4
    SCHEDULER_REQUESTMEMORY=10Gi
    SCHEDULER_LIMITMEMORY=10Gi
    ```
    Ajouter ces paramètre, pour l'activation du mécanisme GC pour le scheduler
    ```sh
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
./upgradedatase-1.2.4.sh
```
Vérifier les log du job "upgrade-postgresql-job-124"

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

## 🔹 Mise à jour du scheduler
Avant de redémarrer le cluster, vérifier qu’aucun traitement n’est en cours: Spark ou autre service.
Ensuite, exécuter la commande suivante :
```sh
./05-scheduler.sh
```

Vérifier dans l'IHM de DCP, dans le menu ***Scheduler*** que les queues ont été chargées dans le scheduler, si ce n'est pas le cas, utiliser le bouton ***Synchronize*** pour recharger les queues.

---

# Configuration Globale
1. Se rendre dans le menu Admin.
2. Aller dans Kubernetes. et seléctionner ***local*** dans la liste

## 🔹 Scheduler
Aller dans l'onglet "Scheduler
Activer l'option "Queue # from tenant"

## 🔹 Storage
Aller dans l'onglet "Global Configuration" puis l'onglet "Storage".
- modifier le champ 'objectorageprotocol' avec cette valeur:
```
s3a,s3n,s3,gs
```

- modifier le champ 'objectstorageconf' avec cette valeur:
```
{%- if bucketprovider != 'GCS' %}
spark.hadoop.fs.s3a.bucket.{{bucketname}}.aws.credentials.provider=org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider
spark.hadoop.fs.s3a.bucket.{{bucketname}}.impl=org.apache.hadoop.fs.s3a.S3AFileSystem
spark.hadoop.fs.s3a.bucket.{{bucketname}}.change.detection.version.require=false
spark.hadoop.fs.s3a.bucket.{{bucketname}}.access.key={{accesskey}}
spark.hadoop.fs.s3a.bucket.{{bucketname}}.secret.key={{secretkey}}
spark.hadoop.fs.s3a.bucket.{{bucketname}}.endpoint={{endpoint}}
spark.hadoop.fs.s3a.bucket.{{bucketname}}.path.style.access={{styleaccess}}
spark.hadoop.fs.s3a.bucket.{{bucketname}}.signing-algorithm={{signertype}}
spark.hadoop.fs.s3a.bucket.{{bucketname}}.connection.ssl.enabled={{sslmode}}
spark.hadoop.fs.s3a.connection.timeout=1200000
spark.hadoop.fs.s3a.connection.maximum=200
spark.hadoop.fs.s3a.fast.upload=true
spark.hadoop.fs.s3a.readahead.range=256K
spark.hadoop.fs.s3a.input.fadvise=random
spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem
{%- else %}
spark.hadoop.fs.gs.impl=com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem
spark.hadoop.google.cloud.auth.service.account.enable=true
{%- if sparkjobgcstokenmountpath != '' %}
spark.hadoop.google.cloud.auth.service.account.json.keyfile={{sparkjobgcstokenmountpath}}
{%- endif %}
{%- endif %}
```

## 🔹 SQL CATALOG
Aller dans l'onglet "Sql" puis l'onglet "Catalogs".
- modifier le champ 'Pattern to control sql catalog names' avec cette valeur:
```
[^a-zA-Z0-9-_]
```

## 🔹 Meatstore
Aller dans l'onglet "Sql" puis l'onglet "Metastore" puis l'onglet "Conf By Bucket".
remplacer le contenu avec:
```
{%- if bucketprovider != 'GCS' %}
fs.s3a.bucket.{{bucketname}}.access.key={{accesskey}}
fs.s3a.bucket.{{bucketname}}.secret.key={{secretkey}}
fs.s3a.bucket.{{bucketname}}.endpoint={{endpoint}}
fs.s3a.bucket.{{bucketname}}.path.style.access={{styleaccess}}
fs.s3a.bucket.{{bucketname}}.signing-algorithm={{signertype}}
fs.s3a.bucket.{{bucketname}}.connection.ssl.enabled={{sslmode}}
fs.s3a.bucket.{{bucketname}}.region={{region}}
{%- else %}
fs.gs.auth.type="SERVICE_ACCOUNT_JSON_KEYFILE"
{%- if gcsprojectid is not none and gcsprojectid != "" %}
fs.gs.project.id="{{gcsprojectid}}"
{%- endif %}
fs.gs.impl="com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem"
fs.AbstractFileSystem.gs.impl="com.google.cloud.hadoop.fs.gcs.GoogleHadoopFS"
fs.gs.auth.service.account.json.keyfile="/opt/dcp/dcpgcstoken/{{randomString}}/gcstoken.json"
fs.gs.reported.permissions="777"
{%- endif %}
```

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


## 🔹 Sauvegarder


# PARAMS

## 🔹 Globals
1. Se rendre dans le menu ***Admin***.
2. Aller dans ***Parameter***
3. Aller dans l'onglet ***Global***.

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
    checkserviceauthbydata
    ```

## 🔹 Repos
Aller dans l'onglet ***Repos***
    ```
    mlflowrepo: mlflow
    mlprojectflaskagent: mlflow
    daskrepo: dask
    filemanagerrepo: dcpfm
    ```

## 🔹 Sauvegarder


# PATCH
1. Se rendre dans le menu ***Admin***.
2. Se rendre dans le sous-menu ***Patch Service***

Clisquer dans l'ordre sur les boutons suivants
Vérifier que chaque étape termine correctement avant de passer à l'étape suivante.
```
Patch namespaces
Patch volumes
Patch Spark Jobs
```

3. Avant d'appliquer le patch des service "Sql policy", s'assurer que tous ces services sont démarrées, sinon le patch ne s'appliquera pas correctement
```
Patch Sql Policy
```

# Hive Metastore
1. Se rendre dans le menu ***Sql***.
2. Se rendre dans le menu ***Metastore***
Le service ***Metastore*** ne supporte plus de prendre en charge plusieurs buckets. 
Sélectionner chaque metastore et sélectionner la bucket à lui associer.

# Sql Policy
Redémarrer tous les services ***Sql Policy*** pour utiliser la dernière version qui apporte des améliorations
