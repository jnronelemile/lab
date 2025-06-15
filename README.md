# Le Marché Frais - Application de Vente de Fruits

Ce projet est une application web full stack conçue pour être une base de travail propre, moderne et conteneurisée. Elle simule une plateforme de vente de fruits frais et est prête à être confiée à une équipe DevOps pour son intégration, son déploiement et son automatisation.

## Stack Technique

- **Frontend**: Vue.js 3 (avec Vite.js) servi par Nginx
- **Backend**: Node.js avec Express.js
- **Base de données**: PostgreSQL
- **Orchestration**: Docker & Docker Compose

## Structure du Projet

Le projet est organisé en trois services principaux, chacun dans son propre répertoire :

- `frontend/`: Contient l'application Vue.js et son Dockerfile pour le build et le service via Nginx
- `backend/`: Contient l'API RESTful Node.js/Express, ses modèles, contrôleurs et son propre Dockerfile
- `db/`: Contient le script d'initialisation init.sql pour la base de données PostgreSQL
# Structure du dossier 
``` markdown
leaky-market/
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   │   ├── authController.js
│   │   │   ├── productController.js
│   │   │   └── orderController.js
│   │   ├── middleware/
│   │   │   └── authMiddleware.js
│   │   ├── models/
│   │   │   ├── index.js
│   │   │   ├── user.js
│   │   │   ├── product.js
│   │   │   └── order.js
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── products.js
│   │   │   └── orders.js
│   │   └── server.js
│   ├── package.json
│   └── Dockerfile
│
├── frontend/
│   ├── public/
│   │   └── favicon.ico
│   ├── src/
│   │   ├── assets/
│   │   │   └── main.css
│   │   ├── components/
│   │   │   ├── ProductList.vue
│   │   │   └── Navbar.vue
│   │   ├── router/
│   │   │   └── index.js
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── views/
│   │   │   ├── HomeView.vue
│   │   │   ├── ProductsView.vue
│   │   │   └── LoginView.vue
│   │   ├── App.vue
│   │   └── main.js
│   ├── package.json
│   ├── Dockerfile
│   └── nginx.conf
│
├── db/
│   └── init.sql
│
├── docker-compose.yml
└── README.md


```

Le fichier `docker-compose.yml` à la racine orchestre le lancement et la mise en réseau de ces trois services.

## Instructions d'Exécution Locale

### Prérequis

- Docker
- Docker Compose

### Lancement de l'application

1. Clonez ce dépôt sur votre machine locale
2. Ouvrez un terminal et naviguez jusqu'à la racine du projet (`leaky-market/`)
3. Exécutez la commande suivante pour construire les images et démarrer les conteneurs :
```bash
docker-compose up --build
```
L'option `--build` force la reconstruction des images si des changements ont été faits dans les Dockerfile ou le code source.

4. Une fois les services lancés :
   - L'application web (Frontend) est accessible à l'adresse : http://localhost:8080
   - L'API (Backend) est accessible à l'adresse : http://localhost:3000

## Informations pour l'Équipe DevOps

### Variables d'Environnement

Le backend requiert les variables d'environnement suivantes. Elles sont actuellement définies dans `docker-compose.yml` pour le développement local. Pour la production, elles devront être gérées via un système de secrets (ex: HashiCorp Vault, AWS Secrets Manager, variables d'environnement CI/CD).

| Variable    | Description |
|-------------|-------------|
| `PORT` | Le port sur lequel le serveur Express écoute |
| `DB_HOST` | Nom d'hôte du service de base de données (`db` dans le réseau Docker) |
| `DB_USER` | Utilisateur de la base de données |
| `DB_PASS` | Mot de passe de l'utilisateur |
| `DB_NAME` | Nom de la base de données |
| `JWT_SECRET` | Clé secrète pour signer les jetons d'authentification JWT. **DOIT ÊTRE CHANGÉE EN PRODUCTION** |

### Description des Services

#### Service `db`
- **Image**: `postgres:14-alpine`
- **Rôle**: Service de base de données
- **Persistance**: Les données sont stockées dans un volume Docker nommé `postgres_data` pour survivre aux redémarrages des conteneurs
- **Initialisation**: Le script `db/init.sql` est exécuté au premier démarrage pour créer le schéma et insérer des données de test

#### Service `backend`
- **Build**: Construit à partir du `backend/Dockerfile`
- **Rôle**: Fournit l'API RESTful pour gérer les utilisateurs, produits et commandes
- **Dépendances**: Dépend du service `db`. Ne démarrera pas tant que la base de données n'est pas prête
- **Exposition**: Le port 3000 du conteneur est mappé sur le port 3000 de l'hôte

#### Service `frontend`
- **Build**: Utilise un Dockerfile multi-étapes. Le premier étage construit les assets statiques avec Node.js, et le second les sert avec un serveur Nginx ultra-léger
- **Rôle**: Sert l'interface utilisateur (Single Page Application Vue.js)
- **Configuration Nginx**: Le fichier `frontend/nginx.conf` est configuré pour servir l'application et gérer correctement le routing côté client (en redirigeant toutes les routes non-statiques vers index.html)
- **Exposition**: Le port 80 du conteneur est mappé sur le port 8080 de l'hôte

### Points d'Entrée de l'API

Les principaux points d'entrée à connaître pour les tests d'intégration et les health checks sont :

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/` | GET | Route de base de l'API |
| `/api/auth/register` | POST | Inscription d'un nouvel utilisateur |
| `/api/auth/login` | POST | Connexion d'un utilisateur |
| `/api/products` | GET | Récupération de la liste des produits |
