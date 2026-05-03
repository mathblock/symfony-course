# Formation Symfony 7 — Jour 3 : Avancé → Expert

**Niveau :** Avancé | **Durée totale :** 7h | **Pré-requis :** Jours 1 & 2 complétés

---

## 🎯 Objectifs du Jour 3

À la fin de cette journée, vous saurez :
- Exposer une API RESTful avec Symfony et API Platform 3
- Traiter des tâches lourdes en arrière-plan avec Messenger
- Optimiser les performances grâce au cache et aux bonnes pratiques Doctrine
- Écrire des tests unitaires et fonctionnels avec PHPUnit
- Préparer et déployer une application Symfony en production

---

## 📋 Programme de la Journée

| Horaire | Activité | Durée |
|---------|----------|-------|
| **9h00 - 9h45** | Module 9 — API avec Symfony & API Platform | 45 min |
| **9h45 - 10h30** | Module 10 — Messenger & Traitement asynchrone | 45 min |
| **10h30 - 10h45** | ☕ Pause | 15 min |
| **10h45 - 11h15** | Module 11 — Cache & Performance | 30 min |
| **11h15 - 12h30** | Module 12 — Tests avec PHPUnit | 1h15 |
| **12h30 - 13h30** | 🍽️ Pause déjeuner | 1h |
| **13h30 - 14h00** | Récapitulatif 3 jours & Ressources | 30 min |
| **14h00 - 17h00** | 📊 Évaluation fil-rouge (voir `evaluation_jour3.md`) | 3h |

---

# ☀️ MATIN : THÉORIE + PRATIQUE (9h00 - 12h30)

---

## 09h00 - 09h45 | Module 9 — API avec Symfony & API Platform

### 9.1 — Pourquoi une API ?

SymfoConnect doit pouvoir être consommé par :
- Une application mobile (iOS / Android)
- Un frontend JavaScript (React, Vue.js)
- Des services tiers

Une **API RESTful JSON** est la réponse standard à ces besoins.

### 9.2 — JSON natif avec Symfony

Sans dépendance supplémentaire, Symfony peut retourner du JSON :

```php
<?php
// src/Controller/Api/PostController.php
namespace App\Controller\Api;

use App\Repository\PostRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/api')]
class PostController extends AbstractController
{
    // Liste des posts (GET /api/posts)
    #[Route('/posts', name: 'api_posts_list', methods: ['GET'])]
    public function list(PostRepository $postRepository): JsonResponse
    {
        $posts = $postRepository->findLatest(20);

        // Sérialisation manuelle (basique)
        $data = array_map(fn($post) => [
            'id'        => $post->getId(),
            'content'   => $post->getContent(),
            'createdAt' => $post->getCreatedAt()->format(\DateTimeInterface::ATOM),
            'author'    => [
                'id'       => $post->getAuthor()->getId(),
                'username' => $post->getAuthor()->getUsername(),
            ],
            'likesCount' => $post->getLikesCount(),
        ], $posts);

        return $this->json(['data' => $data, 'total' => count($data)]);
    }

    // Un post (GET /api/posts/{id})
    #[Route('/posts/{id}', name: 'api_posts_show', methods: ['GET'])]
    public function show(Post $post): JsonResponse
    {
        return $this->json([
            'id'        => $post->getId(),
            'content'   => $post->getContent(),
            'author'    => $post->getAuthor()->getUsername(),
            'createdAt' => $post->getCreatedAt()->format(\DateTimeInterface::ATOM),
        ]);
    }
}
```

### 9.3 — Serializer Component & Groupes

Le **Serializer** de Symfony gère la sérialisation complexe (relations, groupes, formats) :

```php
<?php
// src/Entity/Post.php — Ajouter les groupes de sérialisation
use Symfony\Component\Serializer\Attribute\Groups;

#[ORM\Entity]
class Post
{
    #[ORM\Id]
    #[ORM\Column]
    #[Groups(['post:read', 'feed:read'])]
    private ?int $id = null;

    #[ORM\Column(type: 'text')]
    #[Groups(['post:read', 'post:write', 'feed:read'])]
    private ?string $content = null;

    #[ORM\Column]
    #[Groups(['post:read', 'feed:read'])]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\ManyToOne(targetEntity: User::class)]
    #[Groups(['post:read', 'feed:read'])]
    private ?User $author = null;

    // Propriété calculée (non mappée en BDD)
    #[Groups(['post:read'])]
    public function getLikesCount(): int
    {
        return $this->likedBy->count();
    }
}
```

```php
<?php
// src/Entity/User.php — Groupes sur l'User
class User
{
    #[Groups(['post:read', 'feed:read', 'user:read'])]
    private ?int $id = null;

    #[Groups(['post:read', 'feed:read', 'user:read'])]
    private ?string $username = null;

    // Le mot de passe n'est JAMAIS exposé
    // private ?string $password = null;  ← pas de Group ici
}
```

```php
<?php
// Utiliser le Serializer dans un controller
use Symfony\Component\Serializer\SerializerInterface;

#[Route('/api/posts', name: 'api_posts_list', methods: ['GET'])]
public function list(
    PostRepository $postRepository,
    SerializerInterface $serializer
): JsonResponse {
    $posts = $postRepository->findLatest(20);

    // Sérialiser avec le groupe 'post:read' uniquement
    $json = $serializer->serialize($posts, 'json', ['groups' => ['post:read']]);

    return JsonResponse::fromJsonString($json);
}
```

### 9.4 — API Platform 3 : API Complète Automatique

**API Platform** génère automatiquement une API CRUD complète avec documentation OpenAPI.

#### Installation

```bash
composer require api-platform/core
```

#### Décorer une entité

```php
<?php
// src/Entity/Post.php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post as ApiPost;
use ApiPlatform\Metadata\Delete;
use ApiPlatform\Metadata\ApiFilter;
use ApiPlatform\Doctrine\Orm\Filter\SearchFilter;
use ApiPlatform\Doctrine\Orm\Filter\OrderFilter;

#[ApiResource(
    shortName: 'Post',
    description: 'Les posts du réseau social SymfoConnect',
    operations: [
        // GET /api/posts
        new GetCollection(
            normalizationContext: ['groups' => ['post:read']],
        ),
        // GET /api/posts/{id}
        new Get(
            normalizationContext: ['groups' => ['post:read']],
        ),
        // POST /api/posts (authentifié uniquement)
        new ApiPost(
            denormalizationContext: ['groups' => ['post:write']],
            security: "is_granted('ROLE_USER')",
            securityMessage: 'Vous devez être connecté pour publier.',
        ),
        // DELETE /api/posts/{id} (auteur uniquement via Voter)
        new Delete(
            security: "is_granted('POST_DELETE', object)",
        ),
    ],
    paginationItemsPerPage: 20,
    paginationMaximumItemsPerPage: 100,
    paginationClientItemsPerPage: true,
)]
// Filtres automatiques sur les requêtes
#[ApiFilter(SearchFilter::class, properties: ['content' => 'partial', 'author.username' => 'exact'])]
#[ApiFilter(OrderFilter::class, properties: ['createdAt' => 'desc'])]
#[ORM\Entity(repositoryClass: PostRepository::class)]
class Post
{
    // ... propriétés avec #[Groups]
}
```

#### Documentation automatique

API Platform génère automatiquement :
- `GET /api` → Documentation interactive (Swagger UI)
- `GET /api/docs.json` → Spécification OpenAPI 3
- `GET /api/docs.jsonld` → Spécification JSON-LD

```bash
# Vérifier la configuration
php bin/console debug:router | grep api
```

### 9.5 — Filtres et Pagination

```
# Exemples de requêtes avec API Platform
GET /api/posts                          → 20 premiers posts
GET /api/posts?page=2                   → Page 2
GET /api/posts?itemsPerPage=5           → 5 par page
GET /api/posts?content=symfony          → Recherche dans le contenu
GET /api/posts?author.username=alice    → Posts d'Alice
GET /api/posts?order[createdAt]=asc     → Par date croissante
```

### 9.6 — Sécuriser l'API avec JWT

```bash
composer require lexik/jwt-authentication-bundle
php bin/console lexik:jwt:generate-keypair
```

```yaml
# config/packages/security.yaml
firewalls:
    api:
        pattern: ^/api
        stateless: true
        jwt: ~

    main:
        # ... (formulaire de connexion normal)
```

```yaml
# config/packages/lexik_jwt_authentication.yaml
lexik_jwt_authentication:
    secret_key: '%env(resolve:JWT_SECRET_KEY)%'
    public_key: '%env(resolve:JWT_PUBLIC_KEY)%'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    token_ttl: 3600  # 1 heure
```

```bash
# .env.local
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=votre_passphrase_securisee
```

Usage : `POST /api/login_check` avec `{"username": "...", "password": "..."}` retourne un token JWT à inclure dans le header `Authorization: Bearer <token>`.

### 9.7 — À Vous de Jouer ! — API SymfoConnect

1. Installer API Platform
2. Décorer l'entité `Post` avec `#[ApiResource]`
3. Ajouter les filtres SearchFilter et OrderFilter
4. Tester via Swagger UI : `http://localhost:8000/api`
5. Tester un `GET /api/posts` avec curl ou Postman

```bash
curl -X GET http://localhost:8000/api/posts \
     -H "Accept: application/json"
```

---

## 09h45 - 10h30 | Module 10 — Messenger & Traitement Asynchrone

### 10.1 — Pourquoi l'Asynchrone ?

Certaines opérations sont **lentes** et ne doivent pas bloquer la réponse HTTP :

| Opération | Temps typique |
|-----------|--------------|
| Envoi d'email | 500ms – 2s |
| Redimensionnement d'image | 1s – 5s |
| Appel API externe | 500ms – 10s |
| Génération de PDF | 1s – 3s |

La solution : **exécuter ces tâches en arrière-plan** avec le composant Messenger.

```
Controller → dispatch(Message) → Transport (queue) → Worker → Handler
                                                            ↓
                                                     Traitement lent
```

### 10.2 — Créer un Message et un Handler

```php
<?php
// src/Message/SendNotificationEmailMessage.php
namespace App\Message;

// Le Message est un simple DTO (Data Transfer Object)
// Il doit être sérialisable (pas d'entités Doctrine directement !)
final class SendNotificationEmailMessage
{
    public function __construct(
        public readonly int    $recipientId,
        public readonly string $subject,
        public readonly string $body,
    ) {}
}
```

```php
<?php
// src/MessageHandler/SendNotificationEmailHandler.php
namespace App\MessageHandler;

use App\Message\SendNotificationEmailMessage;
use App\Repository\UserRepository;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;
use Symfony\Component\Mime\Email;

#[AsMessageHandler]
final class SendNotificationEmailHandler
{
    public function __construct(
        private UserRepository $userRepository,
        private MailerInterface $mailer,
    ) {}

    // __invoke est la méthode appelée par le worker
    public function __invoke(SendNotificationEmailMessage $message): void
    {
        $user = $this->userRepository->find($message->recipientId);

        if (!$user) {
            // L'utilisateur a peut-être été supprimé entre-temps
            return;
        }

        $email = (new Email())
            ->from('noreply@symfoconnect.local')
            ->to($user->getEmail())
            ->subject($message->subject)
            ->html($message->body);

        $this->mailer->send($email);
    }
}
```

### 10.3 — Configuration des Transports

```yaml
# config/packages/messenger.yaml
framework:
    messenger:
        # Transport asynchrone (persisté en BDD)
        transports:
            async:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                retry_strategy:
                    max_retries: 3
                    delay: 1000       # 1 seconde avant le 1er retry
                    multiplier: 2     # Délai x2 à chaque retry (1s, 2s, 4s)
                    max_delay: 0
                options:
                    use_notify: true
                    check_delayed_interval: 60000

            # Messages échoués après tous les retries
            failed: 'doctrine://default?queue_name=failed'

        # Règles de routage : quel message → quel transport
        routing:
            App\Message\SendNotificationEmailMessage: async
            App\Message\ResizeImageMessage: async
            # Les messages sans règle sont traités de façon synchrone
```

```bash
# .env
MESSENGER_TRANSPORT_DSN=doctrine://default
# En production avec Redis :
# MESSENGER_TRANSPORT_DSN=redis://localhost:6379/messages
```

> **Astuce :** En développement, utilisez `doctrine://default` (stockage en BDD). En production, préférez Redis ou RabbitMQ pour les performances.

### 10.4 — Dispatcher un Message

```php
<?php
// src/Controller/FollowController.php
use Symfony\Component\Messenger\MessageBusInterface;
use App\Message\SendNotificationEmailMessage;

class FollowController extends AbstractController
{
    public function __construct(
        private MessageBusInterface $bus,
        // ...
    ) {}

    #[Route('/user/{id}/follow', name: 'app_user_follow', methods: ['POST'])]
    #[IsGranted('ROLE_USER')]
    public function follow(User $userToFollow): Response
    {
        $currentUser = $this->getUser();
        $currentUser->follow($userToFollow);
        $this->entityManager->flush();

        // Dispatcher le message asynchrone (ne bloque pas !)
        $this->bus->dispatch(new SendNotificationEmailMessage(
            recipientId: $userToFollow->getId(),
            subject: sprintf('%s vous suit maintenant !', $currentUser->getUsername()),
            body: sprintf(
                '<p>Bonjour %s,</p><p><strong>%s</strong> a commencé à vous suivre sur SymfoConnect.</p>',
                $userToFollow->getUsername(),
                $currentUser->getUsername()
            ),
        ));

        // La réponse est renvoyée immédiatement, sans attendre l'email
        $this->addFlash('success', 'Vous suivez maintenant ' . $userToFollow->getUsername());
        return $this->redirectToRoute('app_profile', ['username' => $userToFollow->getUsername()]);
    }
}
```

### 10.5 — Lancer le Worker

```bash
# Consommer les messages en continu (development)
php bin/console messenger:consume async -vv

# Limiter le nombre de messages traités (production)
php bin/console messenger:consume async --limit=100

# Limiter le temps d'exécution (redémarre le worker proprement)
php bin/console messenger:consume async --time-limit=3600

# Voir les messages en attente
php bin/console messenger:stats

# Rejeter les messages échoués (DLQ)
php bin/console messenger:failed:show
php bin/console messenger:failed:retry
```

> **Bonne pratique :** En production, utilisez **Supervisor** ou **systemd** pour garder les workers actifs et les relancer en cas de crash.

```ini
; /etc/supervisor/conf.d/symfoconnect-worker.conf
[program:symfoconnect-worker]
command=php /var/www/symfoconnect/bin/console messenger:consume async --time-limit=3600
user=www-data
numprocs=2
autostart=true
autorestart=true
```

### 10.6 — À Vous de Jouer ! — Email Asynchrone SymfoConnect

1. Créer le message `NewMessageNotification` (nouveau message privé)
2. Implémenter le handler qui envoie un email
3. Dispatcher le message lors de l'envoi d'un message privé
4. Lancer le worker et observer les logs

```bash
# Lancer le worker en mode verbose
php bin/console messenger:consume async -vv
```

---

## 10h45 - 11h15 | Module 11 — Cache & Performance

### 11.1 — Identifier les Problèmes de Performance

Avant d'optimiser, **mesurez** avec le Profiler Symfony :

```
http://localhost:8000/_profiler  → Liste des requêtes récentes
→ Cliquer sur une requête
→ Onglet "Doctrine" : voir le nombre de requêtes SQL
→ Onglet "Performance" : voir le temps par section
```

**Signaux d'alerte :**
- Plus de 20 requêtes SQL par page → suspicion N+1
- Requêtes > 100ms → à optimiser
- Même requête répétée plusieurs fois → pas de cache

### 11.2 — Cache Component

```php
<?php
// src/Repository/PostRepository.php
use Symfony\Contracts\Cache\CacheInterface;
use Symfony\Contracts\Cache\ItemInterface;

class PostRepository extends ServiceEntityRepository
{
    public function __construct(
        ManagerRegistry $registry,
        private CacheInterface $cache,  // Injecté automatiquement
    ) {
        parent::__construct($registry, Post::class);
    }

    public function findFeedForUser(User $user, int $limit = 20): array
    {
        $cacheKey = sprintf('feed_%d', $user->getId());

        return $this->cache->get($cacheKey, function (ItemInterface $item) use ($user, $limit) {
            // Ce callback n'est exécuté que si le cache est vide/expiré
            $item->expiresAfter(300); // Cache de 5 minutes

            return $this->createQueryBuilder('p')
                ->innerJoin('p.author', 'a')
                ->addSelect('a')
                ->where('a MEMBER OF :following')
                ->orWhere('p.author = :user')
                ->setParameter('following', $user->getFollowing())
                ->setParameter('user', $user)
                ->orderBy('p.createdAt', 'DESC')
                ->setMaxResults($limit)
                ->getQuery()
                ->getResult();
        });
    }

    // Invalider le cache quand un nouveau post est créé
    public function invalidateFeedCache(User $user): void
    {
        $this->cache->delete(sprintf('feed_%d', $user->getId()));

        // Invalider aussi le cache des followers de cet utilisateur
        foreach ($user->getFollowers() as $follower) {
            $this->cache->delete(sprintf('feed_%d', $follower->getId()));
        }
    }
}
```

### 11.3 — Configuration du Cache

```yaml
# config/packages/cache.yaml
framework:
    cache:
        # Adapter par défaut (système de fichiers en dev)
        app: cache.adapter.filesystem

        pools:
            # Cache dédié aux requêtes BDD
            doctrine.result_cache_pool:
                adapter: cache.adapter.filesystem

            # Cache applicatif général
            cache.app:
                adapter: cache.adapter.filesystem
                default_lifetime: 3600

# En production : passer sur Redis
# app: cache.adapter.redis
# default_redis_provider: redis://localhost
```

### 11.4 — Tags de Cache (Invalidation Ciblée)

```php
use Symfony\Contracts\Cache\TagAwareCacheInterface;

class PostRepository extends ServiceEntityRepository
{
    public function __construct(
        ManagerRegistry $registry,
        private TagAwareCacheInterface $cache,
    ) {
        parent::__construct($registry, Post::class);
    }

    public function findFeedForUser(User $user): array
    {
        return $this->cache->get(
            sprintf('feed_%d', $user->getId()),
            function (ItemInterface $item) use ($user) {
                $item->expiresAfter(300);
                // Tag : permet d'invalider tous les caches liés à cet user
                $item->tag(['user_' . $user->getId(), 'feeds']);
                return $this->computeFeed($user);
            }
        );
    }

    // Invalider tout ce qui touche à un utilisateur
    public function invalidateUserCache(int $userId): void
    {
        $this->cache->invalidateTags(['user_' . $userId]);
    }
}
```

### 11.5 — Optimisations Doctrine

```php
// ✅ Utiliser addSelect() pour éviter les N+1
$posts = $this->createQueryBuilder('p')
    ->leftJoin('p.author', 'a')
    ->addSelect('a')  // Charge l'auteur dans la même requête
    ->leftJoin('p.likedBy', 'l')
    ->addSelect('l')  // Charge les likes aussi
    ->orderBy('p.createdAt', 'DESC')
    ->setMaxResults(20)
    ->getQuery()
    ->getResult();

// ✅ Index Doctrine pour les colonnes souvent filtrées
#[ORM\Index(name: 'idx_post_created_at', columns: ['created_at'])]
#[ORM\Index(name: 'idx_post_author', columns: ['author_id'])]
#[ORM\Entity]
class Post { ... }

// ✅ Pagination avec KnpPaginatorBundle (alternative à l'API Platform)
// composer require knplabs/knp-paginator-bundle
```

---

## 11h15 - 12h30 | Module 12 — Tests avec PHPUnit

### 12.1 — Philosophie des Tests

```
Tests unitaires    → Tester une classe isolée (service, entité)
Tests d'intégration → Tester avec de vraies dépendances (BDD de test)
Tests fonctionnels → Tester un controller HTTP bout en bout
```

```bash
# Installation (inclus dans --webapp)
composer require --dev phpunit/phpunit symfony/test-pack

# Créer la BDD de test
php bin/console --env=test doctrine:database:create
php bin/console --env=test doctrine:migrations:migrate
```

```xml
<!-- phpunit.xml.dist -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         colors="true"
         bootstrap="vendor/autoload.php">
    <php>
        <ini name="display_errors" value="1"/>
        <env name="APP_ENV"   value="test"/>
        <env name="APP_DEBUG" value="1"/>
        <env name="DATABASE_URL" value="mysql://root:password@127.0.0.1:3306/symfoconnect_test?serverVersion=8.0"/>
    </php>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Functional">
            <directory>tests/Functional</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

### 12.2 — Tests Unitaires

Les tests unitaires testent une **classe isolée** sans base de données ni HTTP.

```php
<?php
// tests/Unit/Entity/UserTest.php
namespace App\Tests\Unit\Entity;

use App\Entity\User;
use PHPUnit\Framework\TestCase;

class UserTest extends TestCase
{
    private User $user1;
    private User $user2;

    protected function setUp(): void
    {
        $this->user1 = new User();
        $this->user1->setUsername('alice');

        $this->user2 = new User();
        $this->user2->setUsername('bob');
    }

    public function testUserCanFollowAnother(): void
    {
        $this->user1->follow($this->user2);

        $this->assertTrue($this->user1->isFollowing($this->user2));
        $this->assertCount(1, $this->user1->getFollowing());
    }

    public function testUserCannotFollowHimself(): void
    {
        $this->user1->follow($this->user1);

        // Un user ne peut pas se suivre lui-même (notre règle métier)
        $this->assertFalse($this->user1->isFollowing($this->user1));
        $this->assertCount(0, $this->user1->getFollowing());
    }

    public function testUserCanUnfollow(): void
    {
        $this->user1->follow($this->user2);
        $this->user1->unfollow($this->user2);

        $this->assertFalse($this->user1->isFollowing($this->user2));
    }

    public function testFollowIsIdempotent(): void
    {
        // Suivre deux fois ne doit pas créer un doublon
        $this->user1->follow($this->user2);
        $this->user1->follow($this->user2);

        $this->assertCount(1, $this->user1->getFollowing());
    }
}
```

```php
<?php
// tests/Unit/Service/PostServiceTest.php
namespace App\Tests\Unit\Service;

use App\Entity\Post;
use App\Entity\User;
use App\Repository\PostRepository;
use App\Service\PostService;
use Doctrine\ORM\EntityManagerInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class PostServiceTest extends TestCase
{
    private EntityManagerInterface&MockObject $entityManager;
    private PostRepository&MockObject         $postRepository;
    private PostService                        $postService;

    protected function setUp(): void
    {
        // Créer des mocks (faux objets)
        $this->entityManager  = $this->createMock(EntityManagerInterface::class);
        $this->postRepository = $this->createMock(PostRepository::class);

        $this->postService = new PostService(
            $this->entityManager,
            $this->postRepository,
        );
    }

    public function testCreatePostPersistsAndFlushes(): void
    {
        $author = new User();
        $author->setUsername('alice');

        // Vérifier que persist() et flush() sont appelés exactement une fois
        $this->entityManager->expects($this->once())->method('persist');
        $this->entityManager->expects($this->once())->method('flush');

        $post = $this->postService->createPost($author, 'Hello SymfoConnect !');

        $this->assertInstanceOf(Post::class, $post);
        $this->assertSame('Hello SymfoConnect !', $post->getContent());
        $this->assertSame($author, $post->getAuthor());
    }

    public function testCreatePostWithEmptyContentThrowsException(): void
    {
        $author = new User();

        $this->expectException(\InvalidArgumentException::class);
        $this->postService->createPost($author, '   ');
    }
}
```

### 12.3 — Fixtures : Données de Test

```bash
composer require --dev doctrine/doctrine-fixtures-bundle
composer require --dev fakerphp/faker
```

```php
<?php
// src/DataFixtures/UserFixtures.php
namespace App\DataFixtures;

use App\Entity\User;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;
use Faker\Factory;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

class UserFixtures extends Fixture
{
    public function __construct(
        private UserPasswordHasherInterface $passwordHasher,
    ) {}

    public function load(ObjectManager $manager): void
    {
        $faker = Factory::create('fr_FR');

        // Créer un admin connu pour les tests
        $admin = new User();
        $admin->setEmail('admin@symfoconnect.local');
        $admin->setUsername('admin');
        $admin->setPassword($this->passwordHasher->hashPassword($admin, 'password'));
        $admin->setRoles(['ROLE_ADMIN']);
        $manager->persist($admin);
        $this->addReference('user_admin', $admin);

        // Créer des utilisateurs normaux
        for ($i = 0; $i < 20; $i++) {
            $user = new User();
            $user->setEmail($faker->unique()->safeEmail());
            $user->setUsername($faker->unique()->userName());
            $user->setBio($faker->optional()->sentence());
            $user->setPassword($this->passwordHasher->hashPassword($user, 'password'));
            $manager->persist($user);
            $this->addReference('user_' . $i, $user);
        }

        $manager->flush();
    }
}
```

```php
<?php
// src/DataFixtures/PostFixtures.php
namespace App\DataFixtures;

use App\Entity\Post;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\DataFixtures\DependentFixtureInterface;
use Doctrine\Persistence\ObjectManager;
use Faker\Factory;

class PostFixtures extends Fixture implements DependentFixtureInterface
{
    public function load(ObjectManager $manager): void
    {
        $faker = Factory::create('fr_FR');

        for ($i = 0; $i < 50; $i++) {
            $post = new Post();
            $post->setContent($faker->paragraph(3));
            $post->setAuthor($this->getReference('user_' . rand(0, 19)));
            $manager->persist($post);
        }

        $manager->flush();
    }

    // Charger UserFixtures avant PostFixtures
    public function getDependencies(): array
    {
        return [UserFixtures::class];
    }
}
```

```bash
# Charger les fixtures (efface la BDD de test)
php bin/console --env=test doctrine:fixtures:load
```

### 12.4 — Tests Fonctionnels (Controllers)

```php
<?php
// tests/Functional/Controller/HomeControllerTest.php
namespace App\Tests\Functional\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class HomeControllerTest extends WebTestCase
{
    public function testHomePageLoads(): void
    {
        $client = static::createClient();
        $client->request('GET', '/');

        $this->assertResponseIsSuccessful();
        $this->assertSelectorExists('.post-card');  // Des posts sont affichés
    }

    public function testHomePageContainsTitle(): void
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/');

        $this->assertSelectorTextContains('h1', 'Fil d\'actualité');
    }
}
```

```php
<?php
// tests/Functional/Controller/PostControllerTest.php
namespace App\Tests\Functional\Controller;

use App\Entity\User;
use App\Repository\UserRepository;
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class PostControllerTest extends WebTestCase
{
    public function testCreatePostRedirectsIfNotLoggedIn(): void
    {
        $client = static::createClient();
        $client->request('GET', '/post/nouveau');

        // Doit rediriger vers la page de connexion
        $this->assertResponseRedirects('/login');
    }

    public function testLoggedInUserCanAccessPostForm(): void
    {
        $client = static::createClient();

        // Récupérer un user depuis la BDD de test
        $userRepository = static::getContainer()->get(UserRepository::class);
        $user = $userRepository->findOneByUsername('admin');

        // Simuler la connexion (sans passer par le formulaire)
        $client->loginUser($user);

        $client->request('GET', '/post/nouveau');
        $this->assertResponseIsSuccessful();
    }

    public function testLoggedInUserCanCreatePost(): void
    {
        $client = static::createClient();
        $userRepository = static::getContainer()->get(UserRepository::class);
        $user = $userRepository->findOneByUsername('admin');
        $client->loginUser($user);

        // Soumettre le formulaire
        $client->request('POST', '/post/nouveau', [
            'post_form' => [
                'content' => 'Mon test de post !',
                '_token'  => 'csrf_placeholder',  // Géré par WebTestCase
            ],
        ]);

        // Doit rediriger après création
        $this->assertResponseRedirects();
    }

    public function testOnlyAuthorCanDeletePost(): void
    {
        $client = static::createClient();
        $userRepository = static::getContainer()->get(UserRepository::class);

        // Connecté en tant qu'utilisateur lambda
        $user = $userRepository->findOneBy(['username' => 'user_1']);
        $client->loginUser($user);

        // Tenter de supprimer un post qui n'appartient pas à cet utilisateur
        $client->request('POST', '/post/1/delete', ['_token' => 'any']);

        // Doit recevoir un 403 Forbidden
        $this->assertResponseStatusCodeSame(403);
    }
}
```

### 12.5 — Tests de l'API

```php
<?php
// tests/Functional/Api/PostApiTest.php
namespace App\Tests\Functional\Api;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class PostApiTest extends WebTestCase
{
    public function testGetPostsReturnsJsonList(): void
    {
        $client = static::createClient();
        $client->request('GET', '/api/posts', [], [], [
            'HTTP_ACCEPT' => 'application/json',
        ]);

        $this->assertResponseIsSuccessful();
        $this->assertResponseHeaderSame('content-type', 'application/json');

        $data = json_decode($client->getResponse()->getContent(), true);
        $this->assertArrayHasKey('member', $data);  // API Platform retourne 'member'
        $this->assertIsArray($data['member']);
    }

    public function testCreatePostRequiresAuthentication(): void
    {
        $client = static::createClient();
        $client->request('POST', '/api/posts', [], [], [
            'HTTP_ACCEPT'  => 'application/json',
            'CONTENT_TYPE' => 'application/json',
        ], json_encode(['content' => 'Test']));

        $this->assertResponseStatusCodeSame(401);
    }
}
```

### 12.6 — Lancer les Tests

```bash
# Tous les tests
php bin/phpunit

# Un fichier spécifique
php bin/phpunit tests/Unit/Entity/UserTest.php

# Un groupe de tests (avec @group dans les annotations)
php bin/phpunit --group unit

# Avec rapport de couverture (nécessite Xdebug ou PCOV)
php bin/phpunit --coverage-html var/coverage

# Mode verbeux
php bin/phpunit -v

# Arrêter au premier échec
php bin/phpunit --stop-on-failure
```

---

## 12h30 - 13h30 | 🍽️ Pause Déjeuner

---

## 13h30 - 14h00 | Récapitulatif 3 Jours & Déploiement

### Synthèse des 3 Jours

#### Jour 1 — Les Fondamentaux
- Architecture MVC, Routing, Controllers
- Twig : layouts, héritage, filtres
- Doctrine : entités, migrations, repositories
- Formulaires : validation, traitement

#### Jour 2 — Intermédiaire → Avancé
- Relations Doctrine : ManyToOne, OneToMany, ManyToMany
- Sécurité : authentification, rôles, Voters
- Services et injection de dépendances
- Events et Subscribers

#### Jour 3 — Avancé → Expert
- API Platform 3 : CRUD automatique, filtres, documentation OpenAPI
- Messenger : messages asynchrones, workers, retry
- Cache : performance, invalidation par tags
- Tests : unitaires, fonctionnels, mocks, fixtures

### Déploiement en Production

#### Variables d'environnement

```bash
# .env.local.php (production — plus rapide que .env)
# Généré avec : composer dump-env prod
<?php return [
    'APP_ENV' => 'prod',
    'APP_DEBUG' => '0',
    'DATABASE_URL' => 'mysql://user:pass@prod-db:3306/symfoconnect?serverVersion=8.0',
    'MESSENGER_TRANSPORT_DSN' => 'redis://localhost:6379/messages',
];
```

#### Script de déploiement

```bash
#!/bin/bash
# deploy.sh
set -euo pipefail

echo "🚀 Déploiement de SymfoConnect..."

# Mettre en maintenance (optionnel)
php bin/console maintenance:enable

# Récupérer le code
git pull origin main

# Installer les dépendances sans les outils de dev
composer install --no-dev --optimize-autoloader --no-interaction

# Nettoyer et réchauffer le cache de production
php bin/console cache:clear --env=prod
php bin/console cache:warmup --env=prod

# Exécuter les nouvelles migrations (avec confirmation)
php bin/console doctrine:migrations:migrate --no-interaction --env=prod

# Compiler les assets
php bin/console asset-map:compile --env=prod

# Redémarrer PHP-FPM
sudo systemctl reload php8.2-fpm

# Désactiver la maintenance
php bin/console maintenance:disable

echo "✅ Déploiement terminé !"
```

#### CI/CD avec GitHub Actions

```yaml
# .github/workflows/ci.yml
name: Tests & Déploiement

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  tests:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: symfoconnect_test
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP 8.2
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: intl, pdo_mysql, xdebug
          coverage: xdebug

      - name: Cache Composer
        uses: actions/cache@v3
        with:
          path: vendor
          key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}

      - name: Installer les dépendances
        run: composer install --prefer-dist --no-progress

      - name: Préparer la BDD de test
        run: |
          php bin/console --env=test doctrine:database:create --if-not-exists
          php bin/console --env=test doctrine:migrations:migrate --no-interaction
          php bin/console --env=test doctrine:fixtures:load --no-interaction
        env:
          DATABASE_URL: mysql://root:root@127.0.0.1:3306/symfoconnect_test?serverVersion=8.0

      - name: Lancer les tests
        run: php bin/phpunit --coverage-clover coverage.xml
        env:
          DATABASE_URL: mysql://root:root@127.0.0.1:3306/symfoconnect_test?serverVersion=8.0

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: coverage.xml

  deploy:
    needs: tests
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
      - name: Déployer via SSH
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          script: bash /var/www/symfoconnect/deploy.sh
```

### Bonnes Pratiques & Architecture

#### SOLID en Symfony

```php
// ✅ Single Responsibility : un service = une responsabilité
class PostCreationService { }    // Crée des posts
class PostLikeService { }        // Gère les likes
class FeedBuilderService { }     // Construit le fil d'actualité

// ✅ Dependency Inversion : dépendre d'abstractions
interface NotifierInterface { public function notify(User $user, string $message): void; }
class EmailNotifier implements NotifierInterface { }
class PushNotifier implements NotifierInterface { }

// ✅ Open/Closed : extensible sans modification
// Ajouter un nouveau notifier = nouvelle classe, pas de modification de l'existant
```

#### Structure recommandée pour aller plus loin

```
src/
├── Controller/           ← HTTP uniquement (pas de logique métier)
├── Entity/               ← Modèles Doctrine
├── Form/                 ← Types de formulaires
├── Repository/           ← Accès aux données
├── Service/              ← Logique métier
│   ├── Post/
│   │   ├── PostCreationService.php
│   │   └── FeedBuilderService.php
│   └── Notification/
│       └── NotificationService.php
├── Message/              ← Messages Messenger (async)
├── MessageHandler/       ← Handlers Messenger
├── Event/                ← Événements custom
├── EventSubscriber/      ← Subscribers
├── Security/
│   └── Voter/            ← Voters
└── DataFixtures/         ← Fixtures de test
```

### Pour Aller Plus Loin

| Sujet | Ressource |
|-------|-----------|
| Mercure (WebSockets temps réel) | https://mercure.rocks |
| Symfony UX (composants JS) | https://ux.symfony.com |
| API Platform avancé | https://api-platform.com/docs |
| DDD avec Symfony | "Domain-Driven Design in PHP" (livre) |
| Performance avancée | https://blackfire.io |
| Certification Symfony | https://certification.symfony.com |

### Communauté et Ressources

- **Documentation officielle :** https://symfony.com/doc/current/
- **SymfonyCasts (vidéos) :** https://symfonycasts.com
- **Symfony Slack :** https://symfony.com/slack
- **SensioLabs Blog :** https://sensiolabs.com/blog
- **SymfonyWorld Conference :** https://live.symfony.com

---

## 14h00 - 17h00 | 📊 Évaluation Jour 3

L'évaluation complète est détaillée dans le fichier **`evaluation_jour3.md`**.

### Aperçu des Objectifs

Pour finaliser SymfoConnect, vous allez :

1. **Messagerie privée** : entité Message, liste des conversations, envoi et lecture
2. **API REST** : exposer les posts et le profil via API Platform
3. **Cache** : mettre en cache le fil d'actualité (5 min)
4. **Messenger** : email asynchrone à la réception d'un message privé
5. **Tests** : au minimum 5 tests (unitaires + fonctionnels)
6. **Déploiement** : configuration de production (`.env.prod.local`, optimisations)

**Durée :** 3 heures | **Livrable :** Projet SymfoConnect complet et fonctionnel

---

*Félicitations pour ces 3 jours de formation ! Vous avez maintenant toutes les clés pour développer des applications Symfony professionnelles. 🎓*
