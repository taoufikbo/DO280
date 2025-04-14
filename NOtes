
# 🧾 Notes de formation DO280 – OpenShift

## 📚 Sommaire

1. [Types de nœuds dans OpenShift](#1-types-de-nœuds-dans-openshift)  
2. [Configuration DNS dans OpenShift](#2-configuration-dns-requise-dans-openshift)  
3. [Notion de Tenant et Multi-Tenancy](#3-🔹-notion-de-tenant-dans-openshift)  
4. [Architecture d’un opérateur OpenShift/Kubernetes avec CRD](#4-🔹-architecture-dun-opérateur-openshiftkubernetes-avec-crd)  
5. [Types d’utilisateurs dans Kubernetes](#5-🔹-types-dutilisateurs-dans-kubernetes)  
6. [Connexion à l’API OpenShift (diagramme de séquence)](#6-🔹-diagramme-de-séquence--connexion-à-lapi-openshift)  
7. [MachineConfig, MCP, MachineSet – OpenShift 4](#7-machineconfig-machineconfigpool-et-machineset)  
8. [Tableau des rôles des MachineConfigs](#8-tableau-des-rôles-des-machineconfigs)  
9. [Équivalent dans Kubernetes Vanilla](#9-équivalent-dans-kubernetes-vanilla)  
10. [Node Selector & Scheduling](#10-node-selector-dans-kubernetes--scheduler)

---

Objectif de la formation 




Dans OpenShift, il existe plusieurs types de **nœuds**, chacun ayant des rôles et des configurations DNS spécifiques. Voici un résumé des principaux types de nœuds et leurs configurations DNS associées.

---

### **1. Types de nœuds dans OpenShift**
#### **1.1. Nœuds de contrôle (Control Plane / Master Nodes)**
- Rôles :  
  - Gèrent l’état du cluster.
  - Hébergent les composants critiques comme etcd, le scheduler, le contrôleur API.
- Caractéristiques :
  - Ne s’occupent généralement pas de l’exécution des workloads (sauf dans des environnements compacts).
- Configurations DNS associées :
  - Chaque nœud master doit pouvoir résoudre l’adresse du cluster API (`api.<cluster_name>.<domain>`) et celle des autres masters.
  - Exemple d’entrées DNS :
    ```
    api.cluster.example.com → 192.168.1.10
    api-int.cluster.example.com → 192.168.1.11
    master-1.cluster.example.com → 192.168.1.12
    master-2.cluster.example.com → 192.168.1.13
    master-3.cluster.example.com → 192.168.1.14
    ```

---

#### **1.2. Nœuds de calcul (Worker Nodes)**
- Rôles :
  - Hébergent les applications et les workloads.
  - Peuvent être organisés en pools selon des contraintes (GPU, stockage, réseau spécifique...).
- Configurations DNS associées :
  - Ils doivent pouvoir résoudre l’API du cluster (`api.cluster.example.com`) et les autres services internes.
  - Les nœuds workers doivent aussi pouvoir résoudre le service de registre interne (`default-route-openshift-image-registry.apps.cluster.example.com`).
  - Exemple d’entrées DNS :
    ```
    worker-1.cluster.example.com → 192.168.1.21
    worker-2.cluster.example.com → 192.168.1.22
    ```

---

#### **1.3. Nœuds d’infrastructure (Infrastructure Nodes)**
- Rôles :
  - Hébergent les services essentiels au cluster (registre d’images, router, monitoring, logging…).
- Configurations DNS associées :
  - Doivent être accessibles par les autres nœuds pour exposer les services internes.
  - Ils ont souvent des noms spécifiques pour les distinguer des workers classiques :
    ```
    infra-1.cluster.example.com → 192.168.1.31
    infra-2.cluster.example.com → 192.168.1.32
    ```

---

### **2. Configuration DNS requise dans OpenShift**
#### **2.1. DNS Externe (Obligatoire pour l’installation)**
- **Entrées A et CNAME** pour les points d’accès API :
  ```
  api.cluster.example.com → IP du load balancer (ou master)
  api-int.cluster.example.com → IP du load balancer interne
  ```

#### **2.2. DNS Interne (géré par CoreDNS ou dnsmasq)**
- OpenShift utilise un **service DNS interne** basé sur CoreDNS pour la résolution des services internes.
- Exemple de noms de service internes :
  ```
  etcd-0.cluster.example.com → 192.168.1.12
  etcd-1.cluster.example.com → 192.168.1.13
  etcd-2.cluster.example.com → 192.168.1.14
  ```

- Les pods et services utilisent une résolution DNS basée sur `.cluster.local`, par exemple :
  ```
  my-app.my-namespace.svc.cluster.local
  ```
### 🔹 **Notion de "Tenant" dans OpenShift**  

Dans OpenShift, un **tenant** représente un **groupe isolé d’utilisateurs et de workloads** qui partagent des ressources tout en restant cloisonnés des autres groupes. OpenShift n’a pas un concept de "tenant" au sens strict, mais il permet de mettre en place une **multi-tenancy** grâce à plusieurs mécanismes.

---

### 🔹 **Les Mécanismes de Multi-Tenancy dans OpenShift**  

| **Mécanisme**       | **Rôle** |
|---------------------|---------|
| **Namespaces (Projects)** | Isolation des ressources et des workloads pour chaque tenant. |
| **RBAC (Role-Based Access Control)** | Gestion des droits et permissions des utilisateurs par projet. |
| **Network Policies** | Isolation réseau entre tenants/pods. |
| **Resource Quotas & LimitRanges** | Limitation des ressources (CPU, mémoire, etc.) par tenant. |
| **Security Contexts & SCC (Security Context Constraints)** | Contrôle de la sécurité des conteneurs et des permissions. |

---

### 🔹 **1️⃣ Isolation des Tenants avec des Namespaces**  

Un namespace (`project` dans OpenShift) est un espace de travail dédié aux workloads d’un tenant. Chaque tenant peut avoir son propre namespace où il exécute ses applications.

📌 **Exemple : Créer un namespace pour un tenant nommé "team1"**
```bash
oc new-project team1 --description="Tenant Team1" --display-name="Team 1"
```
➡ Ce namespace isole les ressources et évite toute interférence avec d’autres tenants.

---

### 🔹 **2️⃣ Gestion des Droits avec RBAC**  

OpenShift utilise **RBAC (Role-Based Access Control)** pour définir quels utilisateurs ou groupes peuvent accéder à quels namespaces.

📌 **Exemple : Donner accès à "team1" à un utilisateur spécifique**
```bash
oc adm policy add-role-to-user admin user1 -n team1
```
➡ L’utilisateur `user1` a un rôle d’**admin** sur le namespace `team1`, mais pas sur les autres namespaces.

---

### 🔹 **3️⃣ Isolation Réseau avec NetworkPolicies**  

Par défaut, dans OpenShift SDN, tous les pods peuvent communiquer entre eux. Pour isoler les tenants, on utilise **les NetworkPolicies**.

📌 **Exemple : Empêcher tout trafic entrant dans les pods du tenant "team1" sauf depuis son namespace**  
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-external
  namespace: team1
spec:
  podSelector: {}
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: team1
```
➡ Cette règle **bloque tout trafic provenant d’autres namespaces**, sauf `team1` lui-même.

---

### 🔹 **4️⃣ Gestion des Ressources avec Resource Quotas**  

On peut limiter les ressources disponibles pour chaque tenant pour éviter qu’un utilisateur monopolise le cluster.

📌 **Exemple : Limiter un namespace à 4 CPU et 8 Go de RAM**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-team1
  namespace: team1
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```
➡ **Garantit que le tenant "team1" ne dépasse pas ces ressources.**

---

### 🔹 **5️⃣ Sécurité des Conteneurs avec SCC**  

OpenShift utilise **Security Context Constraints (SCC)** pour limiter les actions possibles des conteneurs dans un namespace.

📌 **Exemple : Empêcher un tenant d’exécuter des conteneurs en root**
```bash
oc adm policy add-scc-to-group restricted system:serviceaccounts:team1
```
➡ **Applique une politique de sécurité stricte sur les conteneurs du tenant "team1".**

---

### 🔹 **📌 Conclusion : Comment bien gérer la multi-tenancy ?**  

✔ **Créer un namespace par tenant**  
✔ **Configurer des rôles avec RBAC** pour éviter que les tenants ne voient d’autres ressources  
✔ **Appliquer des NetworkPolicies** pour isoler les communications  
✔ **Limiter les ressources avec Resource Quotas**  
✔ **Restreindre les permissions des conteneurs avec SCC**  

Tu veux mettre en place une séparation des tenants dans un environnement spécifique ? 😊
---

### 🔹 **Architecture d’un opérateur OpenShift/Kubernetes avec CRD**  

Un **opérateur** dans OpenShift/Kubernetes est une application qui automatise la gestion d’un logiciel complexe sur un cluster. Il repose sur le **pattern de contrôle** et utilise des **Custom Resource Definitions (CRD)** pour gérer des objets spécifiques à une application.

---

## **🔹 1️⃣ Composants clés d’un opérateur avec CRD**  

| **Composant**  | **Rôle** |
|---------------|---------|
| **Custom Resource Definition (CRD)** | Définit un nouvel objet personnalisé dans l’API Kubernetes. |
| **Custom Resource (CR)** | Instance du CRD qui décrit la configuration d’un service. |
| **Operator Controller** | Boucle de contrôle qui surveille les CR et applique les changements nécessaires. |
| **Reconciler** | Logique métier qui s’assure que l’état actuel correspond à l’état désiré. |
| **RBAC (Role-Based Access Control)** | Permissions d’accès pour l’opérateur dans le cluster. |

---

## **🔹 2️⃣ Architecture globale**  

L’opérateur suit le pattern **Controller-Operator** et fonctionne selon les étapes suivantes :  

1️⃣ **Définition d’un CRD** → Ajoute un nouvel objet (`MyApp`) dans l’API Kubernetes.  
2️⃣ **Création d’un CR** → Un utilisateur crée une ressource personnalisée (`MyApp-instance`).  
3️⃣ **Surveillance par le contrôleur** → L’opérateur détecte les changements sur le CR.  
4️⃣ **Reconciliation Loop** → L’opérateur applique les changements en créant/modifiant des ressources sous-jacentes (Pods, Services, ConfigMaps, etc.).  
5️⃣ **Mise à jour et gestion continue** → L’opérateur surveille en permanence et corrige tout écart.

---

## **🔹 3️⃣ Exemple d’architecture technique**

```yaml
# 📌 1️⃣ Définition du CRD (Custom Resource Definition)
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.example.com
spec:
  group: example.com
  names:
    kind: MyApp
    plural: myapps
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          replicas:
            type: integer
```

---

```yaml
# 📌 2️⃣ Création d’un CR (Custom Resource)
apiVersion: example.com/v1
kind: MyApp
metadata:
  name: myapp-instance
spec:
  replicas: 3
```

---

```go
// 📌 3️⃣ Contrôleur Go pour gérer le CR
func (r *MyAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := r.Log.WithValues("myapp", req.NamespacedName)

    // Récupérer l'état actuel du CR
    var myApp examplev1.MyApp
    if err := r.Get(ctx, req.NamespacedName, &myApp); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // Vérifier et mettre à jour les ressources (ex: Deployment)
    desiredReplicas := myApp.Spec.Replicas
    err := r.updateDeployment(ctx, myApp, desiredReplicas)
    if err != nil {
        log.Error(err, "Erreur lors de la mise à jour")
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil
}
```

---

## **🔹 4️⃣ Déploiement et gestion de l’opérateur**  

### ✅ **1. Construire l’opérateur avec Operator SDK**  
```bash
operator-sdk init --domain=example.com --repo=github.com/example/my-operator
operator-sdk create api --group=example --version=v1 --kind=MyApp --resource --controller
```

### ✅ **2. Déployer l’opérateur**  
```bash
make deploy
kubectl apply -f config/samples/example_v1_myapp.yaml
```

---

## **📌 Conclusion**
✔ **Les CRD** permettent d’ajouter des API personnalisées dans OpenShift/Kubernetes.  
✔ **L’opérateur** surveille et applique l’état désiré en boucle.  
✔ **Le reconciler** exécute la logique métier pour créer/modifier les ressources Kubernetes.  

Tu veux un opérateur pour un besoin spécifique, comme OpenShift ou une solution cloud ? 😊

### 🔹 **Types d’utilisateurs dans Kubernetes**  

Dans Kubernetes, les utilisateurs ne sont **pas** des objets gérés directement par l’API (contrairement aux ressources comme les Pods ou les Services). Ils sont définis de manière externe (via des certificats, des identités IAM, etc.).  

Il existe **deux types principaux d’utilisateurs** :  

| **Type**            | **Description** |
|---------------------|----------------|
| **Regular users (utilisateurs classiques)** | Représentent des humains accédant au cluster (ex: DevOps, Admins). |
| **System users (utilisateurs système)** | Représentent des services ou des composants internes de Kubernetes. |

---

## **🔹 1️⃣ Regular Users (Utilisateurs classiques)**  

Les **utilisateurs réguliers** sont des **humains** qui interagissent avec le cluster via `kubectl`, `oc` (OpenShift), l’API Kubernetes, ou des interfaces graphiques (comme OpenShift Console ou Lens).  

📌 **Caractéristiques :**  
✔ Ne sont **pas stockés** dans Kubernetes (gérés par des systèmes externes).  
✔ Authentification via **certificats X.509, OIDC, LDAP, IAM (Cloud)**, etc.  
✔ Accès contrôlé par **RBAC (Role-Based Access Control)**.  

📌 **Exemples de Regular Users :**  
- **Admin du cluster** (`admin`)  
- **Développeur** (`developer`)  
- **Opérateur DevOps** (`devops`)  

📌 **Exemple d’authentification par certificat X.509**  
```bash
kubectl config set-credentials taoufik --client-certificate=taoufik.crt --client-key=taoufik.key
```
➡ Ici, `taoufik` est un utilisateur classique identifié par un certificat.

---

## **🔹 2️⃣ System Users (Utilisateurs système)**  

Les **utilisateurs système** sont des comptes utilisés par Kubernetes pour ses propres composants et services.  

📌 **Caractéristiques :**  
✔ Ont un **préfixe `system:`** dans leur nom.  
✔ Gérés automatiquement par Kubernetes.  
✔ Utilisés pour la communication interne entre les composants Kubernetes.  
✔ Assignés à des **ServiceAccounts** pour l’authentification des Pods.  

📌 **Exemples de System Users :**  
| **Utilisateur**                | **Rôle** |
|---------------------------------|---------|
| `system:admin`                  | Administrateur du cluster (OpenShift). |
| `system:kube-controller-manager` | Gère les objets comme les Deployments, les Jobs… |
| `system:kube-scheduler`         | Planifie l’exécution des Pods sur les nœuds. |
| `system:kubelet`                | Exécute les Pods sur un nœud spécifique. |
| `system:serviceaccount:<namespace>:<name>` | Compte attribué aux Pods. |

📌 **Exemple de ServiceAccount attribué à un Pod :**  
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: my-namespace
```
➡ Ce ServiceAccount `my-app-sa` peut être utilisé par un Pod pour interagir avec l’API Kubernetes.

---

## **🔹 3️⃣ Différences entre Regular Users et System Users**  

| **Critère**        | **Regular Users** | **System Users** |
|--------------------|-----------------|----------------|
| **Type d’utilisateur** | Humains | Services/Kubernetes |
| **Stocké dans Kubernetes ?** | ❌ Non (externe) | ✅ Oui (géré en interne) |
| **Authentification** | Certificat X.509, OIDC, IAM | ServiceAccount, RBAC |
| **Utilisation principale** | Interaction avec le cluster (`kubectl`, API…) | Automatisation des composants Kubernetes |
| **Exemple d’utilisateur** | `taoufik`, `devops` | `system:kube-scheduler`, `system:serviceaccount:default:my-app` |

---

## **🔹 📌 Conclusion**  
✔ **Regular Users** 👉 Pour les humains interagissant avec Kubernetes (admins, devs, ops).  
✔ **System Users** 👉 Pour les composants internes de Kubernetes et les Pods.  
✔ L’accès est géré via **RBAC** et l’authentification dépend du contexte (certificats, OIDC, ServiceAccounts).  

Tu veux configurer un accès utilisateur spécifique sur un cluster OpenShift/K8s ? 😊

Voici un **diagramme de séquence** simplifié pour illustrer le processus de connexion à l'API OpenShift avec deux types de comptes différents : un **compte local** et un **compte externe via Keycloak**.

---

### **🔹 Diagramme de séquence : Connexion à l'API OpenShift**

---

#### **1️⃣ Connexion avec un compte local :**

1. **Utilisateur** envoie une demande de connexion à l'API OpenShift.
2. **API OpenShift** redirige la demande vers l'authentification locale (par exemple via `htpasswd`, certificat, ou autres mécanismes).
3. **Service d’authentification OpenShift** vérifie les **identifiants locaux**.
4. Si **authentification réussie**, **OpenShift** génère un **token d’accès** (Bearer Token).
5. L’**API OpenShift** répond avec le **token d’accès**.
6. **Utilisateur** peut maintenant faire des requêtes API avec le **token d’accès** dans l’en-tête.

---

#### **2️⃣ Connexion avec un compte externe via Keycloak :**

1. **Utilisateur** envoie une demande de connexion à l'API OpenShift.
2. **API OpenShift** redirige la demande vers **Keycloak** (serveur d'identité externe).
3. **Keycloak** demande à l’utilisateur de s’authentifier via **OIDC** (par exemple : via un formulaire de login).
4. **Utilisateur** saisit ses identifiants dans **Keycloak**.
5. **Keycloak** vérifie les identifiants de l’utilisateur contre sa base de données ou un fournisseur d'identité (LDAP, etc.).
6. Si **authentification réussie**, **Keycloak** génère un **token ID** (ID Token) et un **token d’accès** (Access Token).
7. **Keycloak** redirige l’utilisateur vers l'**API OpenShift** avec ces tokens.
8. **API OpenShift** valide les tokens via le **provider OIDC** (Keycloak).
9. Si la validation réussie, **OpenShift** génère un **token d’accès** valide pour l’utilisateur et le renvoie.
10. **Utilisateur** utilise le **token d’accès** pour faire des requêtes à l’API OpenShift.

---

### **🔹 Diagramme de séquence visuel :**

Voici une illustration pour mieux comprendre :

```plaintext
                    +-------------------+           +-------------------+           +-----------------+
                    |   Utilisateur     |           |   API OpenShift   |           |    Keycloak     |
                    +-------------------+           +-------------------+           +-----------------+
                             |                            |                               |
                             |     1. Demande de connexion                               |
                             | ------------------------> |                               |
                             |                            | 2. Authentification locale / OIDC|
                             |                            | <-------------------------+     |
                             |                            |                               |
                             |                            |                               |   
                             |                            | 3. Rediriger vers Keycloak (OIDC) |
                             |                            | ---------------------------->     |
                             |                            |                               |
                             |                            |                               |
                             |                            |                               |
                             |                            |    4. Demande d’identifiants     |
                             |                            | <----------------------------     |
                             |                            |                               |
                             |                            | 5. Vérification des identifiants |
                             |                            | ---------------------------->     |
                             |                            |                               |
                             |                            | 6. Token d'accès + ID Token     |
                             |                            | <----------------------------     |
                             |                            |                               |
                             |                            | 7. Rediriger vers OpenShift     |
                             |                            | ---------------------------->     |
                             |                            |                               |
                             |                            | 8. Validation des tokens       |
                             |                            | ---------------------------->     |
                             |                            |                               |
                             |                            | 9. Retourner le token d’accès  |
                             |                            | <----------------------------     |
                             |                            |                               |
                             |                            | 10. Requête API avec token d’accès|
                             | ------------------------> |                               |
```

---

### **🔹 Conclusion**

- **Compte local** : Authentification interne avec OpenShift, pas besoin de service externe.
- **Compte Keycloak** : Utilisation de l’OIDC pour l'authentification, avec un fournisseur externe (Keycloak), permettant une gestion centralisée des identités.

Cela te semble-t-il clair ou tu veux explorer un cas d’utilisation particulier ? 😊


















Dans OpenShift 4, les MachineConfigs et les MachineConfigPools jouent un rôle clé dans la gestion des nœuds du cluster. Voici un détail sur leur fonctionnement et leur configuration.

1. MachineConfig (MC)
Un MachineConfig est une ressource OpenShift qui définit la configuration d'un nœud du cluster, notamment :

Le système d'exploitation (RHCOS)
Les fichiers systèmes (exemple : SSH keys, sysctl, kubelet config)
Les registres de conteneurs
Les modules du kernel
Les paramètres du réseau
Les mises à jour du système
Exemple de MachineConfig
Ce fichier YAML ajoute une clé SSH à l'utilisateur core sur tous les nœuds workers :

yaml
Copier
Modifier
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: worker-ssh-key
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - path: /home/core/.ssh/authorized_keys
        mode: 0600
        overwrite: true
        contents:
          source: data:,ssh-rsa%20AAAAB3...your-key-here
Dans cet exemple :

La config s'applique aux nœuds worker (machineconfiguration.openshift.io/role: worker).
Le fichier /home/core/.ssh/authorized_keys est mis à jour avec une clé SSH.
2. MachineConfigPool (MCP)
Un MachineConfigPool est un groupe de nœuds qui partagent les mêmes configurations. Il est responsable de :

Appliquer les MachineConfig aux nœuds concernés.
Assurer un redémarrage progressif en cas de mise à jour de la configuration.
Exemple de MachineConfigPool
yaml
Copier
Modifier
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  name: worker
spec:
  machineConfigSelector:
    matchExpressions:
      - key: machineconfiguration.openshift.io/role
        operator: In
        values:
          - worker
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/worker: ""
Dans cet exemple :

Le MCP worker applique les configurations aux nœuds ayant le label node-role.kubernetes.io/worker.
3. MachineSet et Machines
OpenShift 4 utilise Machine API pour gérer automatiquement les nœuds. Chaque MachineSet est un groupe de machines géré par un Cluster Autoscaler.

Exemple de MachineSet
yaml
Copier
Modifier
apiVersion: machine.openshift.io/v1beta1
kind: MachineSet
metadata:
  name: worker-us-east-1a
  namespace: openshift-machine-api
spec:
  replicas: 3
  selector:
    matchLabels:
      machine.openshift.io/cluster-api-machineset: worker-us-east-1a
  template:
    metadata:
      labels:
        machine.openshift.io/cluster-api-machineset: worker-us-east-1a
    spec:
      providerSpec:
        value:
          instanceType: m5.large
          region: us-east-1
          ami:
            id: ami-xxxxx
Dans cet exemple :

Un MachineSet gère 3 instances m5.large dans AWS (us-east-1).
Il fonctionne avec la Machine API Operator, qui surveille les machines et remplace celles qui échouent.
4. Cycle de Vie des MachineConfigs
Application : Une nouvelle MachineConfig est ajoutée au cluster.
Drainage des nœuds : Les nœuds affectés sont mis en mode "cordon" (drainés).
Reboot des nœuds : OpenShift applique les changements (nécessite un redémarrage).
Validation : OpenShift s'assure que les nœuds redémarrés fonctionnent bien.
Mise à jour progressive : Le processus continue jusqu'à ce que tous les nœuds soient mis à jour.
5. Commandes Utiles
Lister les MachineConfigs
bash
Copier
Modifier
oc get machineconfig
Lister les MachineConfigPools
bash
Copier
Modifier
oc get machineconfigpool
Appliquer une nouvelle configuration
bash
Copier
Modifier
oc apply -f my-machineconfig.yaml
Voir l'état des mises à jour
bash
Copier
Modifier
oc get machineconfigpool -o wide
Conclusion
MachineConfig permet d'ajouter des configurations aux nœuds.
MachineConfigPool applique les configurations par groupe de nœuds.
MachineSet gère l'autoscaling et la création des machines.
Tu veux approfondir un point particulier ? 😊







Vous avez dit :
detail dans un tableau le role de chauqe mC
ChatGPT a dit :
Voici un tableau détaillant les principaux MachineConfigs (MC) dans OpenShift 4 et leur rôle :

Nom du MachineConfig	Rôle	Description
99-master-ssh	Ajout de clés SSH pour les masters	Configure les clés SSH autorisées pour les nœuds master
99-worker-ssh	Ajout de clés SSH pour les workers	Configure les clés SSH autorisées pour les nœuds worker
99-master-kubelet	Configuration du kubelet sur les masters	Modifie les paramètres du kubelet (ex: logs, cgroup, limites de ressources)
99-worker-kubelet	Configuration du kubelet sur les workers	Modifie les paramètres du kubelet sur les nœuds workers
99-master-container-runtime	Configuration du runtime de conteneurs des masters	Définit les paramètres de crio ou containerd (ex: logs, sandbox, proxy)
99-worker-container-runtime	Configuration du runtime de conteneurs des workers	Configure crio sur les nœuds workers
99-master-registries	Configuration des registres de conteneurs sur les masters	Ajoute des registres privés et configure les proxys de conteneurs
99-worker-registries	Configuration des registres de conteneurs sur les workers	Idem pour les workers
99-master-network	Configuration du réseau sur les masters	Active ou modifie les paramètres réseau (ex: MTU, sysctl, firewall)
99-worker-network	Configuration du réseau sur les workers	Applique les mêmes paramètres réseau sur les workers
99-master-sysctl	Configuration avancée du kernel sur les masters	Applique des réglages système (sysctl) sur les masters
99-worker-sysctl	Configuration avancée du kernel sur les workers	Applique des réglages système (sysctl) sur les workers
99-master-chronyd	Configuration NTP sur les masters	Configure le service NTP pour synchroniser l'heure
99-worker-chronyd	Configuration NTP sur les workers	Configure NTP sur les workers
99-master-audit	Configuration des logs d'audit sur les masters	Active et configure l'audit Kubernetes et Linux
99-worker-audit	Configuration des logs d'audit sur les workers	Idem pour les workers
99-master-extensions	Activation de modules kernel sur les masters	Charge des modules kernel (ex: br_netfilter, ip_vs)
99-worker-extensions	Activation de modules kernel sur les workers	Idem pour les workers
99-master-etcd-encryption	Activation du chiffrement ETCD sur les masters	Active le chiffrement des données stockées dans ETCD
99-worker-cis	Sécurisation des workers selon CIS Benchmarks	Applique des règles de sécurité pour conformité CIS
👉 Remarque :

Tous ces MachineConfigs sont gérés par OpenShift et stockés dans la namespace openshift-machine-config-operator.
Toute modification de l’un d’eux entraîne un redémarrage des nœuds concernés.
Tu peux créer tes propres MachineConfigs personnalisés si besoin.
Tu veux des détails sur un MC spécifique ? 😊







Vous avez dit :
equivalent dans kubertenes Vanilla 
ChatGPT a dit :
Dans un Kubernetes Vanilla (c’est-à-dire sans OpenShift), il n’y a pas de MachineConfig ni de MachineConfigPool, car ces concepts sont propres à OpenShift et basés sur Red Hat Enterprise Linux CoreOS (RHCOS). Cependant, on peut obtenir un comportement similaire en combinant plusieurs outils et pratiques.

Équivalents des MachineConfigs dans Kubernetes Vanilla
MachineConfig OpenShift	Équivalent Kubernetes Vanilla	Description
99-master-ssh / 99-worker-ssh	ConfigMaps + DaemonSet + Ansible	Ajout de clés SSH via un DaemonSet qui met à jour /root/.ssh/authorized_keys
99-master-kubelet / 99-worker-kubelet	kubeletConfiguration dans kubeadm ou ConfigMap	Modification des paramètres du kubelet via kubeadm (/var/lib/kubelet/config.yaml)
99-master-container-runtime / 99-worker-container-runtime	Helm chart ou Ansible pour containerd ou CRI-O	Configuration des paramètres du runtime de conteneurs (/etc/crio/crio.conf ou /etc/containerd/config.toml)
99-master-registries / 99-worker-registries	/etc/containers/registries.conf via ConfigMap ou Ansible	Ajout de registres privés pour CRI-O ou containerd
99-master-network / 99-worker-network	ConfigMap + DaemonSet (sysctl)	Application de paramètres réseau via sysctl (net.core.*, net.ipv4.*, etc.)
99-master-sysctl / 99-worker-sysctl	ConfigMap + DaemonSet (sysctl -p)	Ajout de paramètres kernel via sysctl.conf
99-master-chronyd / 99-worker-chronyd	Ansible ou DaemonSet pour configurer NTP	Configuration de chronyd ou systemd-timesyncd pour la synchronisation NTP
99-master-audit / 99-worker-audit	Audit Policy dans kube-apiserver (--audit-policy-file)	Activation des logs d’audit Kubernetes
99-master-extensions / 99-worker-extensions	DaemonSet pour charger les modules kernel (modprobe)	Activation de modules spécifiques (br_netfilter, ip_vs)
99-master-etcd-encryption	--encryption-provider-config dans kube-apiserver	Activation du chiffrement des données ETCD
99-worker-cis	Scripts Ansible (kube-bench de l'outil CIS Kubernetes Benchmark)	Sécurisation des nœuds selon les standards CIS
Comment appliquer ces configurations ?
1. Via un ConfigMap et un DaemonSet
Exemple pour modifier des paramètres sysctl sur tous les nœuds :

yaml
Copier
Modifier
apiVersion: v1
kind: ConfigMap
metadata:
  name: sysctl-config
  namespace: kube-system
data:
  sysctl.conf: |
    net.ipv4.ip_forward=1
    net.bridge.bridge-nf-call-iptables=1
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: sysctl-daemon
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: sysctl-daemon
  template:
    metadata:
      labels:
        name: sysctl-daemon
    spec:
      hostNetwork: true
      containers:
      - name: sysctl
        image: busybox
        command: ["/bin/sh", "-c", "sysctl -p /host/sysctl.conf"]
        volumeMounts:
        - name: sysctl-config
          mountPath: /host/sysctl.conf
          subPath: sysctl.conf
      volumes:
      - name: sysctl-config
        configMap:
          name: sysctl-config
🔹 Ce DaemonSet applique des paramètres sysctl sur tous les nœuds.

2. Via Ansible pour des modifications système
Exemple de playbook pour ajouter une clé SSH sur tous les nœuds :

yaml
Copier
Modifier
- hosts: all
  become: true
  tasks:
    - name: Ajout de la clé SSH
      authorized_key:
        user: root
        key: "ssh-rsa AAAAB3...your-key-here"
        state: present
🔹 Ce playbook applique la clé SSH sur tous les nœuds.

3. Via kubeadm pour configurer kubelet
yaml
Copier
Modifier
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
eventRecordQPS: 5
maxPods: 110
🔹 Ce fichier de configuration modifie le comportement du kubelet.

Conclusion
OpenShift gère automatiquement ces configurations via MachineConfig, alors que Kubernetes Vanilla nécessite des outils externes comme Ansible, Helm, DaemonSets ou kubeadm.
Si tu veux une gestion similaire dans Kubernetes Vanilla, tu peux utiliser Cluster API pour automatiser la configuration des nœuds, mais cela demande plus d’efforts qu’OpenShift.


Node Selector dans Kubernetes
Le Node Selector est un mécanisme utilisé pour contraindre l'exécution d'un pod sur des nœuds spécifiques en fonction de leurs labels. C'est une solution simple pour influencer le placement des pods sur un cluster.

1. Principe de fonctionnement
Chaque nœud dans Kubernetes possède des labels, qui sont des paires clé-valeur.
Exemple de labels sur un nœud :

shell
Copier
Modifier
kubectl label nodes worker-node1 disktype=ssd
Ce label disktype=ssd pourra être utilisé par un Node Selector.

2. Exemple de Node Selector
Forcer un pod à s’exécuter sur un nœud avec un SSD
yaml
Copier
Modifier
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd  # Le pod ne s'exécutera que sur les nœuds ayant ce label
  containers:
  - name: my-container
    image: nginx
🔹 Si aucun nœud ne correspond, le pod restera en attente (Pending).

3. Limitations du Node Selector
Pas de logique avancée : On ne peut pas exprimer des conditions complexes (AND, OR, NOT).
Pas de fallback : Si aucun nœud ne correspond, Kubernetes ne tente pas d'exécuter le pod ailleurs.
4. Alternatives plus avancées
Méthode	Description
Node Affinity	Permet d’exprimer des règles plus complexes (In, NotIn, Exists, DoesNotExist)
Taints & Tolerations	Évite que des pods non désirés s'exécutent sur un nœud
Pod Affinity/Anti-Affinity	Définit des relations entre les pods (ex: éviter d’avoir deux pods critiques sur le même nœud)

### **Rôle du Scheduler dans Kubernetes et OpenShift**

Le **Scheduler** dans Kubernetes (et OpenShift) est un composant essentiel qui détermine sur quel **nœud** un **pod** doit être déployé. Il prend des décisions en fonction des **ressources disponibles** (CPU, mémoire, stockage) et des **affinités** (comme les labels de nœuds ou les préférences de placement).

Le rôle principal du Scheduler est donc de garantir que les pods soient affectés à des nœuds spécifiques en tenant compte des **conditions d’équilibre** entre les ressources et des **règles de placement** définies par l'utilisateur.

---

## **1. Fonctionnement du Scheduler**

Lorsqu’un pod est créé, il passe par différentes étapes avant d'être exécuté :

1. **Filtrage** : Le Scheduler commence par filtrer les nœuds qui ne satisfont pas aux exigences du pod, comme des ressources insuffisantes (mémoire, CPU) ou des contraintes de **taints** et **tolerations**.
  
2. **Scoring** : Après le filtrage, il évalue les nœuds restants en fonction de **critères de score** tels que :
   - Disponibilité des ressources
   - Affinité de nœud
   - **Pod Affinity/Anti-Affinity** (relations entre les pods)
   - **Node Affinity** (affinité avec des nœuds spécifiques)
   - Proximité des ressources (ex: NUMA, GPU)

3. **Placement** : Après avoir effectué le **scoring**, le Scheduler choisit le nœud avec le meilleur score et y place le pod.

---

## **2. Tableau des Fonctions du Scheduler**

| **Fonction**                       | **Description**                                                                                                                                                          | **Exemple** |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| **Filtrage**                       | Élimine les nœuds qui ne peuvent pas exécuter le pod (en raison de ressources insuffisantes, de taints non tolérés, etc.).                                               | **CPU/Mémoire insuffisants**, **Taints non tolérés** |
| **Scoring des nœuds**              | Évalue les nœuds en fonction de critères tels que la disponibilité des ressources, l'affinité des pods, la topologie NUMA, etc.                                         | **Meilleur nœud pour la topologie NUMA** |
| **Affinité et Anti-affinité**      | Permet de spécifier des règles concernant la relation entre les pods et les nœuds (ex: placer deux pods similaires sur des nœuds différents, ou les mêmes nœuds).       | **Pod Affinity** : Placer un pod avec un autre pod |
| **Node Affinity**                  | Spécifie les règles de placement d’un pod sur un nœud particulier en fonction des labels des nœuds.                                                                     | **Placer sur nœud ayant un label spécifique** |
| **Taints et Tolerations**          | Le Scheduler vérifie si un pod peut tolérer les taints appliqués sur un nœud. Les taints repoussent certains pods d’un nœud, mais les **tolerations** les autorisent à y être. | **Tolerer un taint** pour un placement particulier |
| **Resource Requests et Limits**    | Le Scheduler prend en compte les **ressources demandées** et **limitées** par les pods pour s'assurer que les nœuds aient les ressources nécessaires pour le pod.         | **Demander 2 CPU, 4Gi de RAM** |
| **Proximité des ressources**       | Le Scheduler peut optimiser le placement en fonction de la **proximité des ressources** comme les GPU, les disques ou les réseaux.                                        | **Placer un pod proche d’un GPU ou d’un périphérique spécifique** |
| **Affinité de Topologie (NUMA)**   | Dans des systèmes multi-sockets, il optimise l'affectation des pods pour éviter les cross-NUMA et améliorer les performances.                                              | **Placer un pod sur un nœud NUMA spécifique** |
| **Pod Priority**                   | Le Scheduler peut attribuer des priorités aux pods pour déterminer l’ordre dans lequel ils sont placés sur les nœuds lorsque les ressources sont insuffisantes.           | **Priorité haute pour les pods critiques** |

---

## **3. Configuration du Scheduler dans OpenShift**

OpenShift permet de personnaliser le Scheduler par **MachineConfig** ou via la création de **SchedulerProfiles**. Par exemple :

### **A. Créer un SchedulerProfile personnalisé**
```yaml
apiVersion: operator.openshift.io/v1
kind: Scheduler
metadata:
  name: custom-scheduler
spec:
  policy:
    apiVersion: v1
    kind: Policy
    bindings:
    - name: custom-scheduler-policy
      weight: 100
      rules:
        - host: "worker-node1"
          schedulerName: "custom-scheduler"
```

### **B. Application d’une Affinité de Nœud**
Dans OpenShift, vous pouvez appliquer un **Node Affinity** dans les spécifications des pods :
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  nodeSelector:
    disktype: ssd  # Le pod sera affecté uniquement sur les nœuds avec ce label
  containers:
  - name: nginx
    image: nginx
```

---

## **4. Conclusion**

Le **Scheduler** de Kubernetes/OpenShift joue un rôle crucial dans l’affectation optimale des ressources d'un cluster. Il prend en compte une série de règles, critères de filtrage, et logiques de scoring afin de garantir que les pods sont correctement positionnés pour maximiser l'efficacité et la performance des applications.

Le tableau résume bien toutes les fonctions disponibles, que tu souhaites utiliser dans OpenShift pour affiner l’allocation des ressources, comme **Node Affinity**, **Taints & Tolerations**, et **Pod Affinity**. 
