

# 🚀 AKM GitOps Big Data

Ce dépôt contient l'infrastructure GitOps de **GS2E** pour déployer et gérer un écosystème **Big Data** sur Kubernetes à l'aide de **Helm** et **ArgoCD**.

---

## 🗂️ Structure du projet

```
apps/
  airbyte/
  dremio/
  kafka/
  metabase/
  monitoring/
  nessie/
  postgresql/
  superset/

charts/
  airbyte/
  airflow/
  dremio/
  grafana/
  grafana-loki/
  kafka/
  kube-prometheus/
  metabase/
  nessie/
  postgresql/
  prometheus/
  prometheus-msteams/
  spark-operator/
  superset/
```

* `apps/` : Définitions GitOps des applications pour ArgoCD.
* `charts/` : Charts Helm personnalisés ou tiers pour chaque composant.

---

## ⚙️ Prérequis

* Cluster Kubernetes (Minikube, Kind, EKS, GKE, etc.)
* [Helm](https://helm.sh/) 3.x
* [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
* `kubectl` configuré sur le cluster

---

## 🚀 Déploiement pas à pas

### 1. Démarrer Minikube (si local)

```bash
minikube start
```

### 2. Installer ArgoCD

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Accéder à ArgoCD (en local)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Ouvrir dans le navigateur : [http://localhost:8080](http://localhost:8080)

> Identifiants par défaut :
> **Username**: `admin`
> **Password**:
>
> ```bash
> kubectl -n argocd get secret argocd-initial-admin-secret \
>   -o jsonpath="{.data.password}" | base64 -d && echo
> ```

---

### 4. Configurer le dépôt Git dans ArgoCD

```bash
argocd login localhost:8080 --insecure

argocd repo add https://github.com/AKMADOU/gitops-bigdata.git
```

---

### 5. Déployer une application (ex: Kafka)

Créer une ressource ArgoCD :

```yaml
# apps/kafka/kafka-app-argocd.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kafka
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/AKMADOU/gitops-bigdata.git
    path: kafka
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: kafka
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
```

Puis appliquer :

```bash
kubectl apply -f apps/kafka/kafka-app-argocd.yaml
```

---

### 6. Accéder aux services (ex: Metabase)

```bash
kubectl get svc -n metabase
```

Exemple de sortie :

```
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
metabase   NodePort   10.110.36.2    <none>        80:31700/TCP   1m
```

Récupérer l'IP de Minikube :

```bash
minikube ip
```

Puis accéder à l'URL :
➡️ `http://<minikube-ip>:31700`

> Ou plus simple :

```bash
minikube service metabase -n metabase
```

---

## 📦 Liste des composants disponibles

| Composant  | Rôle principal                       |
| ---------- | ------------------------------------ |
| Airbyte    | Intégration de données open source   |
| Dremio     | Plateforme de Lakehouse              |
| Kafka      | Streaming distribué                  |
| Metabase   | Outil de BI et de data visualisation |
| Superset   | BI Open Source                       |
| Nessie     | Data Catalog / Data Lake Git-like    |
| PostgreSQL | Base de données relationnelle        |
| Spark      | Traitement distribué de données      |
| Prometheus | Monitoring                           |
| Grafana    | Visualisation de métriques           |
| Loki       | Logging distribué                    |

---

## ✅ Bonnes pratiques

* Utiliser des **tags immuables** pour les images Docker.
* Versionner les `values.yaml` spécifiques par environnement.
* Toujours utiliser les dossiers `apps/` pour la gestion via ArgoCD.
---

## 📚 Ressources utiles

- **Documentation ArgoCD** :https://argo-cd.readthedocs.io/
- **Helm Charts** : https://helm.sh/docs/
- **Kubernetes Concepts** : https://kubernetes.io/docs/concepts/
- **Minikube** : https://minikube.sigs.k8s.io/

---



## 🗺️ 2. Diagramme d’architecture

Voici le **diagramme d’architecture** simplifié de mon stack.

### Schéma de base :

```
                     +--------------------+
                     |     ArgoCD         |
                     | (GitOps Control)   |
                     +---------+----------+
                               |
            +------------------+-------------------+
            |                                      |
    +-------v--------+                    +--------v--------+
    |   Helm Charts  |                    |   Git Repository |
    +----------------+                    +------------------+
            |
            |
   +--------v---------+       +---------v--------+     +--------v---------+
   |    Kubernetes     |<---->|     Services      |<--->|      Users       |
   |     Cluster       |       | (Kafka, Spark...)|     +------------------+
   +-------------------+
```

---


## 📋 4. Table des matières (automatique)

Ajoute une table des matières pour faciliter la navigation (surtout si tu pousses ça sur GitHub) :

```md
## 📚 Table des matières

- [Structure du projet](#structure-du-projet)
- [Prérequis](#prérequis)
- [Déploiement pas à pas](#déploiement-pas-à-pas)
- [Composants disponibles](#liste-des-composants-disponibles)
- [Bonnes pratiques](#bonnes-pratiques)
- [Ressources utiles](#ressources-utiles)
- [Licence](#licence)
```

---

