# Formation Symfony 7 — Jour 1 : Les Fondamentaux

**Niveau :** PHP intermédiaire | **Durée totale :** 7h | **Framework :** Symfony 7.x

---

## 🎯 Objectifs du Jour 1

À la fin de cette journée, vous saurez :
- Installer et configurer un projet Symfony 7
- Créer des routes et des controllers
- Utiliser Twig pour créer des templates dynamiques
- Manipuler une base de données avec Doctrine ORM
- Créer et traiter des formulaires avec validation

---

## 📋 Programme de la Journée

| Horaire | Activité | Durée |
|---------|----------|-------|
| **9h00 - 9h45** | Module 1 — Introduction à Symfony 7 | 45 min |
| **9h45 - 10h45** | Module 2 — Routing & Controllers | 1h |
| **10h45 - 11h00** | ☕ Pause | 15 min |
| **11h00 - 11h45** | Module 3 — Twig : le moteur de templates | 45 min |
| **11h45 - 12h30** | Module 4 — Doctrine ORM : les bases | 45 min |
| **12h30 - 13h30** | 🍽️ Pause déjeuner | 1h |
| **13h30 - 14h00** | Récapitulatif & Q&A | 30 min |
| **14h00 - 17h00** | 📊 Évaluation fil-rouge (voir `evaluation_jour1.md`) | 3h |

---

# ☀️ MATIN : THÉORIE + PRATIQUE (9h00 - 12h30)

---

## 09h00 - 09h45 | Module 1 — Introduction à Symfony 7

### 1.1 — Qu'est-ce que Symfony ?

Symfony est un **framework PHP open-source** créé par SensioLabs en 2005. Il est aujourd'hui l'un des frameworks PHP les plus utilisés au monde, notamment dans les grandes entreprises.

**Points forts de Symfony :**
- Architecture modulaire basée sur des **composants réutilisables**
- Respect des standards PHP (PSR) et bonnes pratiques
- Communauté très active, documentation excellente
- Utilisé par des projets majeurs (Drupal, Magento, Laravel s'en inspire)

**Symfony 7 (2023) — Nouveautés clés :**
- PHP 8.2+ obligatoire
- Attributs PHP natifs (fini les annotations Doctrine)
- AssetMapper (remplace Webpack Encore pour les projets simples)
- Améliorations du composant Security

### 1.2 — Architecture MVC dans Symfony

Symfony suit le pattern **MVC (Modèle - Vue - Contrôleur)** :

```
Requête HTTP
     ↓
  Routing  →  Controller  →  Service / Repository
                  ↓                    ↓
               Twig (Vue)          Doctrine (Modèle)
                  ↓
           Réponse HTTP
```

- **Modèle** : Entités Doctrine + Repositories (accès données)
- **Vue** : Templates Twig (HTML dynamique)
- **Contrôleur** : Classes PHP qui orchestrent la logique

### 1.3 — Prérequis et Installation

#### Prérequis

```bash
# Vérifier les versions
php -v        # PHP 8.2 minimum
composer -v   # Composer 2.x
symfony -v    # Symfony CLI (optionnel mais recommandé)
```

#### Installer Symfony CLI (recommandé)

```bash
# Linux/Mac
curl -sS https://get.symfony.com/cli/installer | bash

# Windows
scoop install symfony-cli
```

#### Créer un projet Symfony 7

```bash
# Application web complète (recommandé pour ce cours)
symfony new symfoconnect --version="7.*" --webapp

# Ou sans Symfony CLI (avec Composer)
composer create-project symfony/skeleton:"7.*" symfoconnect
cd symfoconnect
composer require webapp
```

### 1.4 — Structure du Projet

```
symfoconnect/
├── bin/
│   └── console              ← Outil en ligne de commande
├── config/
│   ├── packages/            ← Configuration des bundles
│   ├── routes/              ← Définition des routes
│   └── services.yaml        ← Configuration des services
├── migrations/              ← Migrations Doctrine (SQL)
├── public/
│   └── index.php            ← Point d'entrée HTTP (front controller)
├── src/
│   ├── Controller/          ← Vos controllers
│   ├── Entity/              ← Vos entités Doctrine
│   ├── Form/                ← Vos types de formulaires
│   ├── Repository/          ← Vos repositories
│   └── Service/             ← Vos services métier
├── templates/               ← Vos templates Twig
├── tests/                   ← Vos tests PHPUnit
├── var/
│   ├── cache/               ← Cache applicatif
│   └── log/                 ← Logs
├── vendor/                  ← Dépendances Composer
├── .env                     ← Variables d'environnement (versionné)
├── .env.local               ← Surcharge locale (NON versionné)
└── composer.json
```

### 1.5 — Commandes `bin/console` Essentielles

```bash
# Lister toutes les commandes disponibles
php bin/console list

# Démarrer le serveur de développement
symfony serve          # ou : php -S localhost:8000 -t public/

# Générer un controller
php bin/console make:controller NomController

# Générer une entité
php bin/console make:entity NomEntite

# Générer un formulaire
php bin/console make:form NomForm

# Créer et exécuter les migrations
php bin/console make:migration
php bin/console doctrine:migrations:migrate

# Vider le cache
php bin/console cache:clear

# Afficher toutes les routes
php bin/console debug:router

# Afficher tous les services
php bin/console debug:container
```

> **Astuce :** Vous pouvez abréger les commandes : `php bin/console d:m:m` au lieu de `php bin/console doctrine:migrations:migrate`. Symfony auto-complète les commandes non ambiguës.

### 1.6 — Variables d'Environnement

```bash
# .env (commité dans git — valeurs par défaut)
APP_ENV=dev
APP_SECRET=votre_secret_tres_long_ici
DATABASE_URL="mysql://root:password@127.0.0.1:3306/symfoconnect?serverVersion=8.0&charset=utf8mb4"

# .env.local (ignoré par git — surcharge locale)
DATABASE_URL="mysql://root:monpassword@127.0.0.1:3306/symfoconnect_local?serverVersion=8.0"
```

> **Attention :** Ne jamais committer `.env.local` ! Il contient vos secrets locaux. Il est déjà dans `.gitignore` par défaut.

### 1.7 — À Vous de Jouer ! — Créer le projet SymfoConnect

```bash
# 1. Créer le projet
symfony new symfoconnect --version="7.*" --webapp
cd symfoconnect

# 2. Configurer la BDD dans .env.local
echo 'DATABASE_URL="mysql://root:password@127.0.0.1:3306/symfoconnect?serverVersion=8.0&charset=utf8mb4"' > .env.local

# 3. Créer la base de données
php bin/console doctrine:database:create

# 4. Démarrer le serveur
symfony serve -d

# 5. Visiter http://localhost:8000 → Page de bienvenue Symfony
```

---

## 09h45 - 10h45 | Module 2 — Routing & Controllers

### 2.1 — Créer un Controller

```bash
php bin/console make:controller HomeController
```

Cela génère automatiquement :
- `src/Controller/HomeController.php`
- `templates/home/index.html.twig`

```php
<?php
// src/Controller/HomeController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class HomeController extends AbstractController
{
    #[Route('/', name: 'app_home')]
    public function index(): Response
    {
        return $this->render('home/index.html.twig', [
            'message' => 'Bienvenue sur SymfoConnect !',
        ]);
    }
}
```

### 2.2 — Attributs de Route PHP 8

Les routes se définissent avec l'attribut `#[Route]` directement sur les méthodes :

```php
use Symfony\Component\Routing\Attribute\Route;

// Route simple
#[Route('/accueil', name: 'app_accueil')]
public function accueil(): Response { ... }

// Avec un paramètre
#[Route('/utilisateur/{id}', name: 'app_user_show')]
public function show(int $id): Response { ... }

// Avec contrainte (regex)
#[Route('/post/{slug}', name: 'app_post_show', requirements: ['slug' => '[a-z0-9-]+'])]
public function showPost(string $slug): Response { ... }

// Limiter les méthodes HTTP
#[Route('/post/create', name: 'app_post_create', methods: ['GET', 'POST'])]
public function createPost(): Response { ... }

// Valeur par défaut
#[Route('/page/{numero}', name: 'app_page', defaults: ['numero' => 1])]
public function page(int $numero): Response { ... }
```

### 2.3 — Paramètres de Route et Type Hinting

Symfony convertit automatiquement les paramètres grâce au **ParamConverter** :

```php
// Symfony convertit automatiquement {id} en entité Post
#[Route('/post/{id}', name: 'app_post_show')]
public function show(Post $post): Response
{
    // $post est automatiquement chargé depuis la BDD grâce à l'id
    return $this->render('post/show.html.twig', ['post' => $post]);
}
```

> **Astuce :** Si l'entité n'est pas trouvée, Symfony renvoie automatiquement une erreur 404.

### 2.4 — Request et Response

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\RedirectResponse;

#[Route('/search', name: 'app_search')]
public function search(Request $request): Response
{
    // Récupérer les paramètres GET
    $query = $request->query->get('q', '');         // ?q=symfony
    $page  = $request->query->getInt('page', 1);    // ?page=2

    // Paramètres POST
    $username = $request->request->get('username');

    // Tout le contenu POST en JSON
    $data = json_decode($request->getContent(), true);

    // Headers
    $accept = $request->headers->get('Accept');

    // Méthode HTTP
    if ($request->isMethod('POST')) { ... }

    // Retourner une vue
    return $this->render('search/results.html.twig', [
        'results' => [],
        'query'   => $query,
    ]);
}
```

### 2.5 — Types de Réponses

```php
// Réponse HTML classique (via Twig)
return $this->render('template.html.twig', ['data' => $data]);

// Redirection vers une route nommée
return $this->redirectToRoute('app_home');

// Redirection vers une URL externe
return $this->redirect('https://symfony.com');

// Réponse JSON
return $this->json(['success' => true, 'message' => 'OK']);

// Réponse avec code HTTP personnalisé
return new Response('Contenu', Response::HTTP_NOT_FOUND);

// Lancer une exception HTTP
throw $this->createNotFoundException('Post introuvable');
throw $this->createAccessDeniedException('Accès refusé');
```

### 2.6 — Génération d'URL

```php
// Dans un controller
$url = $this->generateUrl('app_post_show', ['id' => 42]);
$absoluteUrl = $this->generateUrl('app_post_show', ['id' => 42], UrlGeneratorInterface::ABSOLUTE_URL);
```

```twig
{# Dans un template Twig #}
<a href="{{ path('app_post_show', {id: post.id}) }}">Voir le post</a>
<a href="{{ url('app_home') }}">Accueil (URL absolue)</a>
```

### 2.7 — AbstractController : Méthodes Utiles

```php
class PostController extends AbstractController
{
    public function exemple(): Response
    {
        // Utilisateur connecté
        $user = $this->getUser();

        // Vérifier les droits
        $this->denyAccessUnlessGranted('ROLE_USER');

        // Message flash
        $this->addFlash('success', 'Post créé avec succès !');
        $this->addFlash('error', 'Une erreur est survenue.');

        // Vérifier un droit spécifique
        if (!$this->isGranted('ROLE_ADMIN')) {
            return $this->redirectToRoute('app_home');
        }

        return $this->render('post/index.html.twig');
    }
}
```

### 2.8 — Afficher les Routes Disponibles

```bash
# Lister toutes les routes
php bin/console debug:router

# Détails d'une route spécifique
php bin/console debug:router app_post_show
```

### 2.9 — À Vous de Jouer ! — Routes SymfoConnect

Créez les controllers et routes suivants :

```bash
# Générer les controllers
php bin/console make:controller HomeController
php bin/console make:controller ProfileController
php bin/console make:controller PostController
```

Routes à créer :

| Route | URL | Nom |
|-------|-----|-----|
| Page d'accueil | `/` | `app_home` |
| Profil utilisateur | `/profil/{username}` | `app_profile` |
| Créer un post | `/post/nouveau` | `app_post_new` |
| Voir un post | `/post/{id}` | `app_post_show` |

---

## 11h00 - 11h45 | Module 3 — Twig : le Moteur de Templates

### 3.1 — Syntaxe de Base

Twig utilise trois types de balises :

```twig
{# Commentaire — non rendu dans le HTML #}

{{ variable }}                  {# Affiche une variable #}
{{ post.title }}                {# Accès à une propriété #}
{{ post.getTitle() }}           {# Appel de méthode #}
{{ user.email|upper }}          {# Filtre #}

{% if condition %}...{% endif %} {# Bloc de code / logique #}
{% for item in items %}...{% endfor %}
```

### 3.2 — Variables et Filtres Courants

```twig
{# Affichage basique #}
{{ username }}
{{ post.createdAt|date('d/m/Y H:i') }}
{{ post.content|truncate(200) }}
{{ post.content|nl2br }}

{# Filtres essentiels #}
{{ "symfony"|upper }}           {# SYMFONY #}
{{ "SYMFONY"|lower }}           {# symfony #}
{{ "  texte  "|trim }}          {# texte #}
{{ posts|length }}              {# nombre d'éléments #}
{{ 19.99|number_format(2, ',', ' ') }}  {# 19,99 #}
{{ "<p>HTML</p>"|raw }}         {# Affiche le HTML brut (attention XSS !) #}
{{ post.content|escape }}       {# Échappe les caractères HTML (par défaut) #}

{# Valeur par défaut #}
{{ username|default('Anonyme') }}

{# Test de valeur #}
{% if variable is defined %}...{% endif %}
{% if variable is null %}...{% endif %}
{% if posts is empty %}Aucun post{% endif %}
```

### 3.3 — Structures de Contrôle

```twig
{# Condition if/elseif/else #}
{% if user.isAdmin %}
    <span class="badge admin">Admin</span>
{% elseif user.isModerator %}
    <span class="badge mod">Modérateur</span>
{% else %}
    <span class="badge user">Utilisateur</span>
{% endif %}

{# Boucle for #}
{% for post in posts %}
    <article>
        <h2>{{ post.title }}</h2>
        <p>{{ post.content }}</p>
    </article>
{% else %}
    <p>Aucun post à afficher.</p>
{% endfor %}

{# Accès à l'index de boucle #}
{% for post in posts %}
    {{ loop.index }}    {# 1, 2, 3... #}
    {{ loop.index0 }}   {# 0, 1, 2... #}
    {{ loop.first }}    {# true si premier #}
    {{ loop.last }}     {# true si dernier #}
    {{ loop.length }}   {# total d'éléments #}
{% endfor %}

{# Set (définir une variable) #}
{% set fullName = user.firstName ~ ' ' ~ user.lastName %}
{{ fullName }}
```

### 3.4 — Héritage de Templates

Le principe d'héritage permet de définir un **layout de base** et de surcharger des blocs :

```twig
{# templates/base.html.twig — Template parent #}
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}SymfoConnect{% endblock %}</title>
    {% block stylesheets %}
        <link rel="stylesheet" href="{{ asset('styles/app.css') }}">
    {% endblock %}
</head>
<body>
    <header>
        <nav>
            <a href="{{ path('app_home') }}">🏠 SymfoConnect</a>
            {% if app.user %}
                <a href="{{ path('app_profile', {username: app.user.username}) }}">Mon Profil</a>
                <a href="{{ path('app_logout') }}">Déconnexion</a>
            {% else %}
                <a href="{{ path('app_login') }}">Connexion</a>
                <a href="{{ path('app_register') }}">Inscription</a>
            {% endif %}
        </nav>
    </header>

    <main>
        {# Affichage des messages flash #}
        {% for type, messages in app.flashes %}
            {% for message in messages %}
                <div class="alert alert-{{ type }}">{{ message }}</div>
            {% endfor %}
        {% endfor %}

        {% block body %}{% endblock %}
    </main>

    <footer>
        <p>&copy; {{ "now"|date("Y") }} SymfoConnect</p>
    </footer>

    {% block javascripts %}
        {% block importmap %}{{ importmap('app') }}{% endblock %}
    {% endblock %}
</body>
</html>
```

```twig
{# templates/home/index.html.twig — Template enfant #}
{% extends 'base.html.twig' %}

{% block title %}Accueil — SymfoConnect{% endblock %}

{% block body %}
    <h1>Fil d'actualité</h1>

    {% for post in posts %}
        <article class="post-card">
            <div class="post-header">
                <strong>{{ post.author.username }}</strong>
                <span>{{ post.createdAt|date('d/m/Y à H:i') }}</span>
            </div>
            <p>{{ post.content }}</p>
            <div class="post-actions">
                <a href="{{ path('app_post_show', {id: post.id}) }}">Voir</a>
            </div>
        </article>
    {% else %}
        <p>Aucun post pour l'instant. Soyez le premier !</p>
    {% endfor %}
{% endblock %}
```

### 3.5 — Fonctions et Variables Globales Twig

```twig
{# Variables globales Symfony dans Twig #}
{{ app.user }}         {# Utilisateur connecté (ou null) #}
{{ app.request }}      {# Objet Request courant #}
{{ app.session }}      {# Session #}
{{ app.environment }}  {# dev / prod / test #}
{{ app.debug }}        {# true en mode debug #}

{# Fonctions Symfony #}
{{ path('nom_route', {param: valeur}) }}   {# URL relative #}
{{ url('nom_route', {param: valeur}) }}    {# URL absolue #}
{{ asset('images/logo.png') }}             {# URL d'un asset public #}
{{ csrf_token('intention') }}              {# Token CSRF #}
{{ is_granted('ROLE_ADMIN') }}             {# Vérifier un rôle #}
```

### 3.6 — Include et Macros

```twig
{# Inclure un sous-template #}
{% include 'components/_post_card.html.twig' with {post: post} %}

{# Macro : composant réutilisable #}
{% macro postCard(post) %}
    <article class="post-card">
        <h3>{{ post.title }}</h3>
        <p>Par {{ post.author.username }}</p>
    </article>
{% endmacro %}

{# Importer et utiliser une macro #}
{% import _self as macros %}
{{ macros.postCard(post) }}
```

### 3.7 — À Vous de Jouer ! — Layout SymfoConnect

Créez le layout de base avec :
1. Une barre de navigation avec les liens principaux
2. L'affichage des messages flash
3. Un pied de page
4. Un block `title` et un block `body`

---

## 11h45 - 12h30 | Module 4 — Doctrine ORM : les Bases

### 4.1 — Qu'est-ce qu'un ORM ?

Un **ORM (Object-Relational Mapper)** fait le lien entre vos objets PHP et vos tables de base de données.

```
Classe PHP (Entity)   ←→   Table SQL
Propriété PHP         ←→   Colonne SQL
Instance d'objet      ←→   Ligne (row)
```

**Avantages de Doctrine :**
- Pas de SQL manuel (ou presque)
- Indépendant du SGBD (MySQL, PostgreSQL, SQLite...)
- Migrations automatiques
- Système de cache intégré

### 4.2 — Créer une Entité

```bash
php bin/console make:entity User
```

Doctrine pose des questions pour chaque champ. Voici l'entité générée et enrichie :

```php
<?php
// src/Entity/User.php
namespace App\Entity;

use App\Repository\UserRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[ORM\Table(name: 'users')]
#[ORM\HasLifecycleCallbacks]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180, unique: true)]
    private ?string $email = null;

    #[ORM\Column(length: 50, unique: true)]
    private ?string $username = null;

    #[ORM\Column]
    private ?string $password = null;

    #[ORM\Column(type: 'text', nullable: true)]
    private ?string $bio = null;

    #[ORM\Column(length: 255, nullable: true)]
    private ?string $avatarUrl = null;

    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    // Le constructeur initialise les valeurs par défaut
    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }

    // Getters et Setters (générés par make:entity)
    public function getId(): ?int { return $this->id; }

    public function getEmail(): ?string { return $this->email; }
    public function setEmail(string $email): static
    {
        $this->email = $email;
        return $this;
    }

    public function getUsername(): ?string { return $this->username; }
    public function setUsername(string $username): static
    {
        $this->username = $username;
        return $this;
    }

    public function getBio(): ?string { return $this->bio; }
    public function setBio(?string $bio): static
    {
        $this->bio = $bio;
        return $this;
    }

    // ... autres getters/setters
}
```

### 4.3 — Types de Colonnes Doctrine

```php
#[ORM\Column]                                    // string(255) par défaut
#[ORM\Column(length: 100)]                       // varchar(100)
#[ORM\Column(type: 'text')]                      // TEXT
#[ORM\Column(type: 'integer')]                   // INT
#[ORM\Column(type: 'float')]                     // FLOAT
#[ORM\Column(type: 'boolean')]                   // TINYINT(1)
#[ORM\Column(type: 'datetime_immutable')]        // DATETIME
#[ORM\Column(type: 'date_immutable')]            // DATE
#[ORM\Column(type: 'json')]                      // JSON
#[ORM\Column(nullable: true)]                    // Colonne nullable
#[ORM\Column(unique: true)]                      // UNIQUE constraint
#[ORM\Column(name: 'nom_colonne_sql')]           // Nom SQL personnalisé
```

### 4.4 — Migrations : Synchroniser la BDD

```bash
# 1. Générer la migration (compare les entités avec la BDD)
php bin/console make:migration

# 2. Vérifier le SQL généré dans migrations/VersionXXX.php
# 3. Exécuter la migration
php bin/console doctrine:migrations:migrate

# Voir le statut des migrations
php bin/console doctrine:migrations:status

# Revenir en arrière (rollback)
php bin/console doctrine:migrations:migrate prev
```

> **Attention :** Toujours vérifier le contenu de la migration avant de l'exécuter en production ! Doctrine peut proposer de supprimer des colonnes si vous avez supprimé une propriété.

### 4.5 — EntityManager : Persister des Données

```php
// src/Controller/PostController.php
use Doctrine\ORM\EntityManagerInterface;

class PostController extends AbstractController
{
    public function __construct(
        private EntityManagerInterface $entityManager
    ) {}

    #[Route('/post/create-demo', name: 'app_post_demo')]
    public function createDemo(): Response
    {
        // 1. Créer l'objet
        $post = new Post();
        $post->setContent('Mon premier post sur SymfoConnect !');
        $post->setAuthor($this->getUser());

        // 2. "Programmer" l'insertion (ne touche pas encore la BDD)
        $this->entityManager->persist($post);

        // 3. Exécuter toutes les opérations en attente (INSERT SQL)
        $this->entityManager->flush();

        $this->addFlash('success', 'Post créé avec succès !');
        return $this->redirectToRoute('app_home');
    }

    public function deletePost(Post $post): Response
    {
        $this->entityManager->remove($post);
        $this->entityManager->flush();

        return $this->redirectToRoute('app_home');
    }
}
```

### 4.6 — Repositories : Lire des Données

```php
// src/Repository/PostRepository.php
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

class PostRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Post::class);
    }

    // Méthodes disponibles par défaut (héritées)
    // $repo->find(42)                      → Post par ID
    // $repo->findAll()                     → Tous les posts
    // $repo->findBy(['author' => $user])   → Posts d'un auteur
    // $repo->findOneBy(['slug' => 'test']) → Un post par critère
    // $repo->count([])                     → Nombre total

    // Méthode personnalisée
    public function findLatest(int $limit = 10): array
    {
        return $this->createQueryBuilder('p')
            ->orderBy('p.createdAt', 'DESC')
            ->setMaxResults($limit)
            ->getQuery()
            ->getResult();
    }
}
```

Utiliser un repository dans un controller :

```php
// Injection automatique via autowiring
public function index(PostRepository $postRepository): Response
{
    $latestPosts = $postRepository->findLatest(10);

    return $this->render('home/index.html.twig', [
        'posts' => $latestPosts,
    ]);
}
```

### 4.7 — À Vous de Jouer ! — Entités SymfoConnect

Créez les entités `User` et `Post` :

```bash
# Créer l'entité User
php bin/console make:entity User
# Champs : email (string, unique), username (string 50, unique), bio (text, nullable), avatarUrl (string, nullable)

# Créer l'entité Post
php bin/console make:entity Post
# Champs : content (text), createdAt (datetime_immutable)

# Générer et exécuter les migrations
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

> Les relations (User → Post) seront ajoutées lors de l'évaluation.

---

## 12h30 - 13h30 | 🍽️ Pause Déjeuner

---

## 13h30 - 14h00 | Récapitulatif & Q&A

### Résumé des Concepts Clés

**Module 1 — Symfony 7**
- Architecture MVC : Controller → Service → Template
- Structure projet : `src/`, `config/`, `templates/`, `public/`
- `bin/console` : outil central du développeur Symfony

**Module 2 — Routing & Controllers**
- `#[Route('/url', name: 'nom', methods: ['GET'])]` sur les méthodes
- `Request` : accès aux paramètres GET/POST/headers
- `Response`, `JsonResponse`, `redirectToRoute()`
- Paramètres de route convertis automatiquement en entités (ParamConverter)

**Module 3 — Twig**
- `{{ }}` affiche, `{% %}` logique, `{# #}` commentaire
- `extends` / `block` pour l'héritage de templates
- Filtres : `|date`, `|upper`, `|length`, `|default`
- Variables globales : `app.user`, `app.request`

**Module 4 — Doctrine ORM**
- Entités = classes PHP mappées sur des tables
- `persist()` + `flush()` pour sauvegarder
- Migrations : `make:migration` puis `doctrine:migrations:migrate`
- Repositories pour requêter la BDD

### Points de Vigilance

> **Attention — flush() oublié :** `persist()` seul ne sauvegarde rien ! N'oubliez jamais `flush()`.

> **Attention — Migrations en production :** Toujours vérifier le SQL de migration avant de l'exécuter. Une migration mal écrite peut supprimer des données.

> **Attention — `|raw` dans Twig :** Ce filtre désactive l'échappement HTML. Ne l'utilisez jamais sur du contenu saisi par l'utilisateur (risque XSS).

> **Astuce — Profiler Symfony :** En mode `dev`, une barre d'outils apparaît en bas de page. Elle affiche les requêtes SQL, les performances, les logs, etc. Indispensable !

### Questions Fréquentes

**Q : Quelle différence entre `find()` et `findOneBy()` ?**
R : `find()` cherche par ID uniquement. `findOneBy()` accepte n'importe quel critère : `findOneBy(['email' => 'test@test.com'])`.

**Q : Comment afficher une valeur booléenne en Twig ?**
R : `{% if post.isPublished %}Publié{% else %}Brouillon{% endif %}`

**Q : Peut-on définir des routes dans YAML au lieu des attributs PHP ?**
R : Oui, dans `config/routes.yaml`. Mais les attributs PHP sont maintenant la convention recommandée dans Symfony 7.

---

## 14h00 - 17h00 | 📊 Évaluation Jour 1

L'évaluation complète est détaillée dans le fichier **`evaluation_jour1.md`**.

### Aperçu des Objectifs

Vous allez initialiser le projet **SymfoConnect** et mettre en place les fondations :

1. Installer le projet Symfony 7 et configurer la BDD
2. Créer les entités `User` et `Post` avec leur relation
3. Générer et exécuter les migrations
4. Créer un layout Twig de base
5. Implémenter la page d'accueil (liste des derniers posts)
6. Implémenter la page de profil d'un utilisateur
7. Créer un formulaire basique de création de post

**Durée :** 3 heures | **Livrable :** Code source fonctionnel

---

## 📚 Ressources du Jour 1

| Ressource | URL |
|-----------|-----|
| Documentation Symfony 7 | https://symfony.com/doc/current/ |
| Doctrine ORM | https://www.doctrine-project.org/projects/doctrine-orm/en/latest/ |
| Twig Documentation | https://twig.symfony.com/doc/ |
| SymfonyCasts (tutoriels vidéo) | https://symfonycasts.com/ |
| Symfony CLI | https://symfony.com/download |

---

*Bonne journée et bon courage pour l'évaluation ! 💪*
