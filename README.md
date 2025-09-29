# init-sre-beginner


# 📝 TD – Introduction au SRE, SLI/SLO et mise en pratique avec Grafana sur Kubernetes

---

## 🎯 Objectifs

* Comprendre ce qu’est le **Site Reliability Engineering (SRE)**.
* Assimiler les notions de **SLI (Service Level Indicator)**, **SLO (Service Level Objective)** et **SLA (Service Level Agreement)**.
* Mettre en place un **exemple pratique d’exposition de métriques** et les afficher dans **Grafana hébergé sur Kubernetes**.
* Faire le lien entre **observabilité**, **résilience** et **fiabilité des services**.

---

## 📚 Partie 1 – Cours théorique : SRE et SLI/SLO

### 1.1 Qu’est-ce que le SRE ?

* **Définition** : Le SRE est une discipline née chez Google, qui applique l’ingénierie logicielle à la gestion des opérations pour garantir la **fiabilité, la disponibilité et la performance des systèmes**.
* **Objectif** : équilibrer l’innovation rapide (dev) et la fiabilité (ops).
* **Principe clé** : *"L’ingénierie au service de la disponibilité"*.

Exemple concret :

* Une équipe SRE va automatiser la gestion des incidents, mettre en place des dashboards, définir des alertes, au lieu de gérer les problèmes uniquement à la main.

---

### 1.2 Les 3 notions fondamentales : SLI, SLO et SLA

* **SLI (Service Level Indicator)** : un indicateur mesurable de la qualité de service.
  → Exemple : le **taux de requêtes HTTP 200**, la **latence moyenne des requêtes**, le **taux d’erreurs**.

* **SLO (Service Level Objective)** : un objectif mesurable fixé sur un SLI.
  → Exemple : *95 % des requêtes doivent répondre en moins de 300ms sur une période de 30 jours*.

* **SLA (Service Level Agreement)** : contrat externe (souvent juridique) entre le fournisseur et le client.
  → Exemple : *Disponibilité garantie de 99,9 % sur un mois, sinon pénalité financière*.

⚖️ **Différence clé** :

* SLI = métrique brute.
* SLO = objectif technique interne.
* SLA = engagement contractuel externe.

---

### 1.3 Notions associées

* **Error budget** : tolérance maximale d’erreurs avant de violer un SLO.
  → Exemple : si SLO = 99,9% de disponibilité sur 30 jours → error budget = 0,1 % d’indisponibilité ≈ 43 min par mois.
* **Burn rate** : vitesse à laquelle on consomme l’error budget.

---

## 💻 Partie 2 – Mise en pratique avec Kubernetes et Grafana

### 2.1 Pré-requis

* Cluster Kubernetes (local avec **kind** ou **minikube**, ou distant).
* **kubectl** configuré.
* Helm installé.
* Docker pour build une petite app.

---

### 2.2 Déploiement d’une application simple avec métriques

On déploie une petite application Spring Boot (ou Node.js/Express) qui expose un endpoint `/metrics` compatible Prometheus.

👉 Exemple (Spring Boot Actuator) :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: my-dockerhub/demo-app:latest
        ports:
        - containerPort: 8080
```

👉 Service :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-app
spec:
  selector:
    app: demo-app
  ports:
    - port: 80
      targetPort: 8080
```

Cette app expose par exemple :

* `/hello` → retourne un "Hello World".
* `/metrics` → retourne des métriques Prometheus (latence, requêtes HTTP, erreurs).

---

### 2.3 Installation de Prometheus + Grafana avec Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Installation de Prometheus + Grafana
helm install monitoring prometheus-community/kube-prometheus-stack
```

Cela installe :

* Prometheus (scraping des métriques)
* Grafana (visualisation)
* Node-exporter, kube-state-metrics, etc.

---

### 2.4 Configuration de Grafana

* Accéder à Grafana (port-forward ou ingress) :

```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```

* URL : `http://localhost:3000`
* Login : `admin / prom-operator` (par défaut).

👉 Ajouter un **dashboard** qui affiche les SLIs :

* **SLI 1 : Disponibilité**

  ```promql
  sum(rate(http_server_requests_seconds_count{status=~"2.."}[5m]))
  /
  sum(rate(http_server_requests_seconds_count[5m]))
  ```
* **SLI 2 : Latence**

  ```promql
  histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket[5m])) by (le))
  ```

---

### 2.5 Exercices pratiques

1. **Exercice 1** : Définir un SLI sur la disponibilité de `demo-app`.

   * Créez un dashboard Grafana affichant le **% de requêtes 200**.
   * Définir un SLO : *99 % de requêtes OK sur 7 jours*.

2. **Exercice 2** : Définir un SLI sur la latence (p95).

   * Dashboard Grafana → latence 95th percentile.
   * Définir un SLO : *95 % des requêtes < 300ms*.

3. **Exercice 3** : Simuler des erreurs.

   * Injectez des erreurs (retourner 500 sur 10 % des requêtes).
   * Observez l’impact sur les dashboards et calculez si le SLO est violé.

4. **Exercice 4 (avancé)** : Définir un **alerting** dans Prometheus :

   * Si disponibilité < 95 % sur 10 min → alerte.
   * Si latence p95 > 500ms sur 5 min → alerte.

---

## 🏁 Partie 3 – Conclusion & ouverture

* **Le SRE apporte une méthodologie** pour allier innovation et fiabilité.
* **Les SLI/SLO sont le langage commun** entre dev, ops et métier.
* **Grafana + Prometheus + K8s** = stack idéale pour mesurer et suivre ces indicateurs.
* Ouverture : intégration avec **Alertmanager** et **Error Budget Policies**.


Veux-tu que je te prépare aussi **les supports de slides** résumant la partie cours (SRE, SLI, SLO, SLA, error budget, burn rate) pour que tu les projettes en début de TD ?
