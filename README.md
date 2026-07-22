# 📝 Projet ToDoList – Frontend Angular

Ce dépôt contient le frontend de l’application **ToDoList**, une application web de gestion de tâches conçue avec Angular. Il permet à un utilisateur de créer, modifier, supprimer et filtrer des tâches selon leur statut.

---

## 🧱 Stack technique

- **Framework** : [Angular 15](https://angular.io/)
- **Langage** : TypeScript
- **Design system** : Angular Material
- **Gestion des formulaires** : Reactive Forms
- **Tests unitaires** : Jasmine + Karma
- **Client HTTP** : HttpClient Angular

---

## 🚀 Lancement du projet

### ✅ Prérequis

- Node.js (version recommandée : >= 16)
- Angular CLI (version 15 installée globalement)

```bash
npm install -g @angular/cli@15
```

### 📦 Installation des dépendances

```bash
npm install
```

### ▶️ Démarrage du serveur de développement

```bash
ng serve
```

L'application sera accessible à l'adresse : [http://localhost:4200](http://localhost:4200)

---

## 🧪 Tests unitaires

```bash
ng test
```

Cela lance les tests unitaires avec **Karma** et **Jasmine**. Les tests sont définis dans les fichiers `*.spec.ts` à côté de chaque composant, service ou module testé.

ℹ️ **Aucun test end-to-end (e2e)** n’est présent dans ce projet.

---

## 📁 Arborescence du projet

```
src/
├── app/
│   ├── components/
│   │   ├── task-form/         # Composant de formulaire de tâche (création/modification)
│   │   │   └── task-form.component.*
│   │   ├── task-list/         # Composant d'affichage des tâches avec filtres
│   │   │   └── task-list.component.*
│   ├── services/
│   │   └── task.service.ts    # Service de communication avec l'API backend
│   ├── models/
│   │   └── task.model.ts      # Interface représentant une tâche
│   ├── app.component.*        # Composant racine et layout général
│   ├── app-routing.module.ts  # Configuration des routes Angular
│   └── app.module.ts          # Module principal de l'application
├── assets/                    # Fichiers statiques
└── index.html                 # Page HTML principale
```

---

## 🧩 Modules et fonctionnalités

### 📌 `task-list.component`
- Affiche la liste des tâches récupérées depuis le backend.
- Permet un filtrage par statut (`À faire`, `En cours`, `Terminée`).
- Propose des boutons d'action pour modifier ou supprimer chaque tâche.
- Composant réactif, rafraîchi automatiquement après suppression.

### ✍️ `task-form.component`
- Gère le formulaire d’ajout ou de modification de tâche.
- Détermine automatiquement le mode (`création` ou `édition`) selon la route (`/ajouter` ou `/modifier/:id`).
- Envoie les données au backend via le `TaskService`.

### 🔁 `task.service.ts`
- Contient les appels HTTP au backend :
  - `getAllTasks()`
  - `getTask(id)`
  - `addTask(task)`
  - `updateTask(task)`
  - `deleteTask(id)`
- Utilise `HttpClient` et renvoie des observables typés.

### 🧭 `app-routing.module.ts`
- Déclare les routes suivantes :
  - `/` → composant `TaskListComponent`
  - `/ajouter` → composant `TaskFormComponent`
  - `/modifier/:id` → composant `TaskFormComponent` en mode édition

### 🧱 `task.model.ts`
- Interface `Task` définissant les propriétés :
  - `id?: number`
  - `nom: string`
  - `description: string`
  - `statut: string`

---

## 📚 Bonnes pratiques

- La logique métier est centralisée dans le `TaskService`.
- L’UI est construite avec Angular Material pour un rendu cohérent et accessible.
- Les composants sont découplés et respectent le principe SRP (Single Responsibility Principle).
- Les routes sont définies de manière claire et intuitive.
- Le code est tapé fortement grâce à TypeScript et aux interfaces.

---

## 🌐 URL du backend : pourquoi ce n'est plus `localhost` en dur

Avant, `task.service.ts` pointait directement vers `http://localhost:3000/api/tasks`. Ça fonctionne en local, mais c'est cassé dès que l'application est déployée sur Azure : le backend n'a plus cette adresse-là, et surtout **on ne connaît pas son URL Azure à l'avance** (elle dépend du nom de l'App Service qui sera créé).

La solution repose sur le mécanisme standard d'Angular : les fichiers d'environnement.

- [`src/environments/environment.ts`](src/environments/environment.ts) : utilisé en développement (`ng serve`, `ng test`). Contient `apiUrl: 'http://localhost:3000/api'`.
- [`src/environments/environment.prod.ts`](src/environments/environment.prod.ts) : utilisé uniquement pour le build de production (`ng build --configuration production`). Sa valeur `apiUrl` est **écrasée automatiquement par le pipeline CI/CD** juste avant le build (voir plus bas) — le contenu commité dans ce fichier n'est qu'un garde-fou.
- [`angular.json`](angular.json) déclare un `fileReplacements` dans la configuration `production` : au build, Angular remplace le contenu de `environment.ts` par celui de `environment.prod.ts`, de façon totalement transparente pour le code.
- [`task.service.ts`](src/app/services/task.service.ts) ne contient plus aucune URL en dur : il lit `environment.apiUrl`.

**Ce que ça veut dire concrètement pour vous :**
- En local, ne touchez à rien : `ng serve` utilise déjà `environment.ts` (localhost).
- Ne modifiez jamais `environment.prod.ts` à la main pour y mettre une vraie URL Azure : ce fichier est régénéré automatiquement à chaque exécution du pipeline. Si vous le modifiez en local, ça n'aura aucun effet sur le déploiement.

---

## 🔐 CI/CD : déploiement automatique sur Azure (GitHub Actions)

Le pipeline est défini dans [`.github/workflows/ci-cd.yml`](.github/workflows/ci-cd.yml). Il comporte deux jobs qui s'enchaînent :

1. **`test`** (à chaque push et pull request) : installe les dépendances puis lance `ng test --watch=false --browsers=ChromeHeadless`. Si un test échoue, le pipeline s'arrête ici — **rien n'est déployé**.
2. **`build-and-deploy`** (uniquement sur un push sur `master`, et uniquement si `test` a réussi) :
   1. Connexion à Azure avec un compte de service (service principal).
   2. Interrogation d'Azure pour récupérer l'URL réelle du backend déjà déployé (`az webapp show`).
   3. Régénération du fichier `environment.prod.ts` avec cette URL.
   4. Build de production (`ng build --configuration production`).
   5. Déploiement du frontend sur Azure *(étape encore en `TODO` dans le fichier : à compléter une fois que la conteneurisation du frontend et le choix du service Azure — Web App, Container Apps, etc. — seront décidés)*.

Pour que ce pipeline fonctionne, il faut lui donner accès à Azure. **Suivez ces étapes dans l'ordre, une seule fois par équipe/projet :**

### Étape 1 — Créer un compte de service Azure (service principal)

Sur votre machine, connectés à Azure CLI (`az login`) avec un compte ayant les droits sur l'abonnement du projet, exécutez :

```bash
az ad sp create-for-rbac \
  --name "sp-todolist-github-actions" \
  --role reader \
  --scopes /subscriptions/<VOTRE_SUBSCRIPTION_ID>/resourceGroups/<VOTRE_RESOURCE_GROUP> \
  --sdk-auth
```

- `<VOTRE_SUBSCRIPTION_ID>` : visible avec `az account show --query id -o tsv`.
- `<VOTRE_RESOURCE_GROUP>` : le groupe de ressources Azure qui contient (ou contiendra) le backend et le frontend.
- Le rôle `reader` suffit ici : le pipeline a seulement besoin de *lire* l'URL du backend (`az webapp show`), pas de modifier quoi que ce soit sur Azure via cette étape.

La commande affiche un bloc JSON qui ressemble à :

```json
{
  "clientId": "...",
  "clientSecret": "...",
  "subscriptionId": "...",
  "tenantId": "..."
}
```

⚠️ **Copiez tout ce bloc JSON immédiatement** (il ne sera plus jamais réaffiché) — vous en avez besoin à l'étape suivante.

### Étape 2 — Renseigner les secrets et variables dans GitHub

Dans le dépôt GitHub : **Settings → Secrets and variables → Actions**.

Onglet **Secrets** (valeurs sensibles, jamais affichées) → **New repository secret** :

| Nom du secret | Valeur |
|---|---|
| `AZURE_CREDENTIALS` | le bloc JSON complet copié à l'étape 1 |

Onglet **Variables** (non sensibles, visibles dans les logs) → **New repository variable** :

| Nom de la variable | Valeur | Comment la trouver |
|---|---|---|
| `AZURE_BACKEND_APP_NAME` | nom de l'App Service Azure du backend | `az webapp list --resource-group <VOTRE_RESOURCE_GROUP> --query "[].name" -o tsv`, ou dans le portail Azure : App Services → nom de la ressource backend |
| `AZURE_RESOURCE_GROUP` | nom du resource group Azure | Portail Azure → Resource groups, ou `az group list --query "[].name" -o tsv` |

Ces noms doivent correspondre **exactement** à ceux utilisés dans `.github/workflows/ci-cd.yml` (`vars.AZURE_BACKEND_APP_NAME`, `vars.AZURE_RESOURCE_GROUP`).

### Étape 3 — Vérifier que le backend est déjà déployé

Le pipeline frontend interroge un App Service **qui doit déjà exister sur Azure** au moment où il tourne (il ne le crée pas). Le backend doit donc être déployé avant, ou au moins avoir sa CI/CD qui tourne en amont de celle du frontend.

### Étape 4 — Compléter le déploiement du frontend

La toute dernière étape du job `build-and-deploy` est volontairement laissée en commentaire (`TODO`) dans `ci-cd.yml` : elle dépend de décisions pas encore prises dans ce projet (conteneurisation avec Docker, choix entre Azure Web App / Container Apps / Static Web App). Une fois ces choix faits, décommentez et adaptez cette section, en ajoutant les secrets/variables correspondants (ex. `AZURE_FRONTEND_APP_NAME`, `AZURE_FRONTEND_PUBLISH_PROFILE`).

### Vérifier que tout fonctionne

Après un push sur `master` : onglet **Actions** du dépôt GitHub → workflow **CI/CD Frontend** → vérifier que les deux jobs sont verts. En cas d'échec, ouvrez le job concerné et lisez le message d'erreur de l'étape en rouge — c'est presque toujours une des situations décrites ci-dessous.

### 🩺 Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Le job `test` échoue | Un test unitaire est cassé (voir le message Jasmine dans les logs) | Corriger le test ou le code avant tout, le pipeline ne déploiera pas tant que les tests sont rouges |
| `Azure login` échoue | `AZURE_CREDENTIALS` mal copié, expiré, ou absent | Regénérer le secret via la commande de l'étape 1 et le remplacer dans GitHub |
| `az webapp show` renvoie une erreur ou une valeur vide | `AZURE_BACKEND_APP_NAME` ou `AZURE_RESOURCE_GROUP` incorrect, ou le backend n'est pas encore déployé | Vérifier les noms exacts avec `az webapp list` / `az group list`, s'assurer que le backend est déployé |
| Le site déployé appelle encore `REPLACE_AT_BUILD_TIME` ou `localhost` | L'étape "Generate environment.prod.ts" n'a pas tourné (job en échec avant, ou déploiement fait manuellement en dehors de la CI) | Ne jamais faire de `ng build --configuration production` en dehors du pipeline pour un déploiement réel |

---

## 📎 Ressources utiles

- [Documentation Angular](https://angular.io/docs)
- [Angular CLI cheatsheet](https://angular.io/cli)
- [Angular Material](https://material.angular.io/components/categories)
