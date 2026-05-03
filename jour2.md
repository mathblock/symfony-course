# Formation Symfony 7 — Jour 2 : Intermédiaire → Avancé

**Niveau :** PHP intermédiaire | **Durée totale :** 7h | **Pré-requis :** Jour 1 complété

---

## 🎯 Objectifs du Jour 2

À la fin de cette journée, vous saurez :
- Modéliser des relations complexes avec Doctrine (ManyToMany, OneToMany)
- Sécuriser une application avec le système d'authentification Symfony
- Mettre en place une autorisation fine avec les Voters
- Créer des services réutilisables avec l'injection de dépendances
- Réagir aux événements applicatifs avec les Listeners/Subscribers

---

## 📋 Programme de la Journée

| Horaire | Activité | Durée |
|---------|----------|-------|
| **9h00 - 10h00** | Module 5 — Doctrine : Relations avancées | 1h |
| **10h00 - 11h15** | Module 6 — Sécurité Symfony | 1h15 |
| **11h15 - 11h30** | ☕ Pause | 15 min |
| **11h30 - 12h15** | Module 7 — Services & Injection de Dépendances | 45 min |
| **12h15 - 12h30** | Module 8 — Events, Listeners et Subscribers | 15 min |
| **12h30 - 13h30** | 🍽️ Pause déjeuner | 1h |
| **13h30 - 14h00** | Récapitulatif & Q&A | 30 min |
| **14h00 - 17h00** | 📊 Évaluation fil-rouge (voir `evaluation_jour2.md`) | 3h |

---

# ☀️ MATIN : THÉORIE + PRATIQUE (9h00 - 12h30)

---

## 09h00 - 10h00 | Module 5 — Doctrine : Relations Avancées

### 5.1 — Les Trois Types de Relations

Doctrine propose trois relations principales :

| Relation | Exemple concret | Table créée |
|----------|----------------|-------------|
| **ManyToOne** | Plusieurs posts pour un auteur | Clé étrangère dans Post |
| **OneToMany** | Un auteur a plusieurs posts | (côté inverse de ManyToOne) |
| **ManyToMany** | Un user suit plusieurs users | Table de jointure |

### 5.2 — ManyToOne / OneToMany (Relation Auteur ↔ Posts)

```php
<?php
// src/Entity/Post.php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\Collection;
use Doctrine\Common\Collections\ArrayCollection;

#[ORM\Entity(repositoryClass: PostRepository::class)]
class Post
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(type: 'text')]
    private ?string $content = null;

    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    // RELATION : Un post appartient à un utilisateur (ManyToOne)
    // "author" = propriété dans Post | "posts" = propriété dans User
    #[ORM\ManyToOne(targetEntity: User::class, inversedBy: 'posts')]
    #[ORM\JoinColumn(nullable: false, onDelete: 'CASCADE')]
    private ?User $author = null;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }

    public function getAuthor(): ?User { return $this->author; }
    public function setAuthor(?User $author): static
    {
        $this->author = $author;
        return $this;
    }

    // ... autres getters/setters
}
```

```php
<?php
// src/Entity/User.php — Côté inverse de la relation
use Doctrine\Common\Collections\Collection;
use Doctrine\Common\Collections\ArrayCollection;

class User
{
    // RELATION : Un utilisateur a plusieurs posts (OneToMany)
    // cascade: persist/remove propage les opérations aux posts
    // orphanRemoval: supprime les posts orphelins (sans auteur)
    #[ORM\OneToMany(
        mappedBy: 'author',
        targetEntity: Post::class,
        cascade: ['persist', 'remove'],
        orphanRemoval: true
    )]
    #[ORM\OrderBy(['createdAt' => 'DESC'])]
    private Collection $posts;

    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }

    public function getPosts(): Collection
    {
        return $this->posts;
    }

    public function addPost(Post $post): static
    {
        if (!$this->posts->contains($post)) {
            $this->posts->add($post);
            $post->setAuthor($this);
        }
        return $this;
    }

    public function removePost(Post $post): static
    {
        if ($this->posts->removeElement($post)) {
            if ($post->getAuthor() === $this) {
                $post->setAuthor(null);
            }
        }
        return $this;
    }
}
```

### 5.3 — ManyToMany (Follows et Likes)

```php
<?php
// src/Entity/User.php — Relation follows (User suit User)
class User
{
    // Un user peut suivre plusieurs users
    #[ORM\ManyToMany(targetEntity: self::class, inversedBy: 'followers')]
    #[ORM\JoinTable(
        name: 'user_follows',
        joinColumns: [new ORM\JoinColumn(name: 'follower_id', referencedColumnName: 'id')],
        inverseJoinColumns: [new ORM\JoinColumn(name: 'followed_id', referencedColumnName: 'id')]
    )]
    private Collection $following;

    // Un user peut avoir plusieurs followers
    #[ORM\ManyToMany(targetEntity: self::class, mappedBy: 'following')]
    private Collection $followers;

    public function __construct()
    {
        $this->following = new ArrayCollection();
        $this->followers = new ArrayCollection();
    }

    public function follow(User $user): void
    {
        if (!$this->following->contains($user) && $user !== $this) {
            $this->following->add($user);
        }
    }

    public function unfollow(User $user): void
    {
        $this->following->removeElement($user);
    }

    public function isFollowing(User $user): bool
    {
        return $this->following->contains($user);
    }

    public function getFollowing(): Collection { return $this->following; }
    public function getFollowers(): Collection { return $this->followers; }
    public function getFollowersCount(): int { return $this->followers->count(); }
    public function getFollowingCount(): int { return $this->following->count(); }
}
```

```php
// src/Entity/Post.php — Relation likes (User like Post)
class Post
{
    // Un post peut être liké par plusieurs users
    #[ORM\ManyToMany(targetEntity: User::class)]
    #[ORM\JoinTable(name: 'post_likes')]
    private Collection $likedBy;

    public function __construct()
    {
        $this->likedBy = new ArrayCollection();
    }

    public function addLike(User $user): void
    {
        if (!$this->likedBy->contains($user)) {
            $this->likedBy->add($user);
        }
    }

    public function removeLike(User $user): void
    {
        $this->likedBy->removeElement($user);
    }

    public function isLikedBy(User $user): bool
    {
        return $this->likedBy->contains($user);
    }

    public function getLikesCount(): int
    {
        return $this->likedBy->count();
    }

    public function getLikedBy(): Collection { return $this->likedBy; }
}
```

### 5.4 — QueryBuilder : Requêtes Complexes

```php
<?php
// src/Repository/PostRepository.php
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;

class PostRepository extends ServiceEntityRepository
{
    /**
     * Fil d'actualité : posts des utilisateurs suivis
     */
    public function findFeedForUser(User $user, int $limit = 20): array
    {
        return $this->createQueryBuilder('p')
            ->innerJoin('p.author', 'a')
            ->addSelect('a')                            // Évite le N+1 sur l'auteur
            ->where('a MEMBER OF :following')
            ->orWhere('p.author = :user')               // Inclure ses propres posts
            ->setParameter('following', $user->getFollowing())
            ->setParameter('user', $user)
            ->orderBy('p.createdAt', 'DESC')
            ->setMaxResults($limit)
            ->getQuery()
            ->getResult();
    }

    /**
     * Posts avec comptage de likes (exemple de sous-requête)
     */
    public function findWithLikesCount(): array
    {
        return $this->createQueryBuilder('p')
            ->leftJoin('p.likedBy', 'l')
            ->addSelect('COUNT(l) as likesCount')
            ->groupBy('p.id')
            ->orderBy('likesCount', 'DESC')
            ->getQuery()
            ->getResult();
    }

    /**
     * Recherche plein texte simple
     */
    public function search(string $term): array
    {
        return $this->createQueryBuilder('p')
            ->where('p.content LIKE :term')
            ->setParameter('term', '%' . $term . '%')
            ->orderBy('p.createdAt', 'DESC')
            ->getQuery()
            ->getResult();
    }
}
```

### 5.5 — Lazy Loading vs Eager Loading

```php
// ❌ PROBLÈME N+1 : génère 1 + N requêtes SQL
$posts = $postRepository->findAll();
foreach ($posts as $post) {
    echo $post->getAuthor()->getUsername(); // 1 requête SQL par post !
}

// ✅ SOLUTION : charger les auteurs en même temps (JOIN FETCH)
$posts = $this->createQueryBuilder('p')
    ->leftJoin('p.author', 'a')
    ->addSelect('a')   // Charge l'auteur dans la même requête
    ->getQuery()
    ->getResult();

// ✅ Pour les grosses collections : EXTRA_LAZY
#[ORM\OneToMany(mappedBy: 'author', targetEntity: Post::class, fetch: 'EXTRA_LAZY')]
private Collection $posts;
// Avec EXTRA_LAZY, $user->getPosts()->count() ne charge PAS tous les posts
```

> **Bonne pratique :** Utilisez le **Profiler Symfony** (`/_profiler`) pour détecter les N+1 : regardez l'onglet Doctrine et comptez les requêtes SQL.

### 5.6 — À Vous de Jouer ! — Relations SymfoConnect

```bash
# Ajouter les relations dans les entités existantes
php bin/console make:entity User
# → Ajouter : following (relation ManyToMany vers User)

php bin/console make:entity Post
# → Ajouter : author (relation ManyToOne vers User)
# → Ajouter : likedBy (relation ManyToMany vers User)

# Générer la migration
php bin/console make:migration

# Exécuter la migration
php bin/console doctrine:migrations:migrate
```

---

## 10h00 - 11h15 | Module 6 — Sécurité Symfony

### 6.1 — Architecture de Sécurité

Le composant Security de Symfony fonctionne en couches :

```
Requête HTTP
     ↓
Firewall → Authentification → Autorisation
               ↓                   ↓
         LoginForm          access_control
         JWT, OAuth         #[IsGranted]
                            Voters
```

### 6.2 — Configurer security.yaml

```yaml
# config/packages/security.yaml
security:
    # Algorithme de hashage des mots de passe
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface:
            algorithm: auto   # bcrypt par défaut, migre automatiquement

    # Fournisseurs d'utilisateurs (depuis quelle source charger l'user ?)
    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email   # Identifiant de connexion

    firewalls:
        # Zone de développement (pas de sécurité)
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        # Zone principale de l'application
        main:
            lazy: true
            provider: app_user_provider

            # Formulaire de connexion
            form_login:
                login_path: app_login
                check_path: app_login
                enable_csrf: true
                default_target_path: app_home

            # Déconnexion
            logout:
                path: app_logout
                target: app_home

            # Se souvenir de moi
            remember_me:
                secret: '%kernel.secret%'
                lifetime: 604800   # 7 jours

    # Contrôle d'accès global (par URL)
    access_control:
        - { path: ^/login,      roles: PUBLIC_ACCESS }
        - { path: ^/register,   roles: PUBLIC_ACCESS }
        - { path: ^/admin,      roles: ROLE_ADMIN }
        - { path: ^/post/new,   roles: ROLE_USER }
        # Le reste est accessible sans connexion

    # Hiérarchie des rôles
    role_hierarchy:
        ROLE_ADMIN: ROLE_USER
        ROLE_MODERATOR: ROLE_USER
        ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_MODERATOR]
```

### 6.3 — UserInterface : L'Entité User Sécurisée

```bash
# Générer un User compatible avec Symfony Security
php bin/console make:user
```

```php
<?php
// src/Entity/User.php — Version sécurisée complète
namespace App\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;

#[ORM\Entity(repositoryClass: UserRepository::class)]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    // ... colonnes id, email, username, bio, avatarUrl, createdAt

    #[ORM\Column]
    private array $roles = [];

    #[ORM\Column]
    private ?string $password = null;

    // Identifiant unique (pour Symfony Security)
    public function getUserIdentifier(): string
    {
        return (string) $this->email;
    }

    // Rôles de l'utilisateur (ROLE_USER toujours présent)
    public function getRoles(): array
    {
        $roles = $this->roles;
        $roles[] = 'ROLE_USER';
        return array_unique($roles);
    }

    public function setRoles(array $roles): static
    {
        $this->roles = $roles;
        return $this;
    }

    // Mot de passe hashé
    public function getPassword(): ?string { return $this->password; }
    public function setPassword(string $password): static
    {
        $this->password = $password;
        return $this;
    }

    // Nettoyer les données sensibles temporaires (non nécessaire ici)
    public function eraseCredentials(): void {}
}
```

### 6.4 — Inscription (Registration)

```bash
# Générer le formulaire d'inscription
php bin/console make:registration-form
```

```php
<?php
// src/Form/RegistrationFormType.php
namespace App\Form;

use App\Entity\User;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\PasswordType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\CheckboxType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints as Assert;

class RegistrationFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('email', EmailType::class, [
                'label' => 'Adresse email',
                'constraints' => [
                    new Assert\NotBlank(),
                    new Assert\Email(),
                ],
            ])
            ->add('username', TextType::class, [
                'label' => 'Nom d\'utilisateur',
                'constraints' => [
                    new Assert\NotBlank(),
                    new Assert\Length(['min' => 3, 'max' => 30]),
                    new Assert\Regex([
                        'pattern' => '/^[a-zA-Z0-9_]+$/',
                        'message' => 'Lettres, chiffres et underscores uniquement',
                    ]),
                ],
            ])
            ->add('plainPassword', PasswordType::class, [
                'mapped' => false,   // Non mappé sur l'entité (on hashera manuellement)
                'label' => 'Mot de passe',
                'constraints' => [
                    new Assert\NotBlank(),
                    new Assert\Length(['min' => 8]),
                ],
            ])
            ->add('agreeTerms', CheckboxType::class, [
                'mapped' => false,
                'label' => 'J\'accepte les conditions d\'utilisation',
                'constraints' => [new Assert\IsTrue(['message' => 'Vous devez accepter les CGU.'])],
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults(['data_class' => User::class]);
    }
}
```

```php
<?php
// src/Controller/RegistrationController.php
namespace App\Controller;

use App\Entity\User;
use App\Form\RegistrationFormType;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;
use Symfony\Component\Routing\Attribute\Route;

class RegistrationController extends AbstractController
{
    #[Route('/register', name: 'app_register')]
    public function register(
        Request $request,
        UserPasswordHasherInterface $passwordHasher,
        EntityManagerInterface $entityManager
    ): Response {
        $user = new User();
        $form = $this->createForm(RegistrationFormType::class, $user);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // Hasher le mot de passe
            $hashedPassword = $passwordHasher->hashPassword(
                $user,
                $form->get('plainPassword')->getData()
            );
            $user->setPassword($hashedPassword);

            $entityManager->persist($user);
            $entityManager->flush();

            $this->addFlash('success', 'Compte créé ! Vous pouvez vous connecter.');
            return $this->redirectToRoute('app_login');
        }

        return $this->render('registration/register.html.twig', [
            'registrationForm' => $form,
        ]);
    }
}
```

### 6.5 — Connexion (Login Form)

```bash
# Générer le formulaire de connexion
php bin/console make:auth
# → Choisir : Login form authenticator
```

```php
<?php
// src/Controller/SecurityController.php
namespace App\Controller;

use Symfony\Component\Security\Http\Authentication\AuthenticationUtils;
use Symfony\Component\Routing\Attribute\Route;

class SecurityController extends AbstractController
{
    #[Route('/login', name: 'app_login')]
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // Récupérer l'erreur de connexion (si présente)
        $error = $authenticationUtils->getLastAuthenticationError();
        // Pré-remplir l'email saisi
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/login.html.twig', [
            'last_username' => $lastUsername,
            'error' => $error,
        ]);
    }

    #[Route('/logout', name: 'app_logout')]
    public function logout(): void
    {
        // Ce code n'est jamais exécuté — Symfony intercepte /logout
        throw new \LogicException('Géré par Symfony Security');
    }
}
```

```twig
{# templates/security/login.html.twig #}
{% extends 'base.html.twig' %}

{% block body %}
    <div class="login-form">
        <h1>Connexion à SymfoConnect</h1>

        {% if error %}
            <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
        {% endif %}

        <form method="post">
            <label for="inputEmail">Email</label>
            <input type="email" id="inputEmail" name="_username" value="{{ last_username }}" required autofocus>

            <label for="inputPassword">Mot de passe</label>
            <input type="password" id="inputPassword" name="_password" required>

            <label>
                <input type="checkbox" name="_remember_me"> Se souvenir de moi
            </label>

            {# Token CSRF — obligatoire #}
            <input type="hidden" name="_csrf_token" value="{{ csrf_token('authenticate') }}">

            <button type="submit">Connexion</button>
        </form>

        <a href="{{ path('app_register') }}">Pas encore de compte ? S'inscrire</a>
    </div>
{% endblock %}
```

### 6.6 — Autorisation : Protéger les Routes

```php
// Méthode 1 : Attribut #[IsGranted] sur la méthode
use Symfony\Component\Security\Http\Attribute\IsGranted;

#[Route('/post/new', name: 'app_post_new')]
#[IsGranted('ROLE_USER')]
public function new(): Response { ... }

// Méthode 2 : Dans le code du controller
public function deletePost(Post $post): Response
{
    $this->denyAccessUnlessGranted('ROLE_USER');
    // ou
    if (!$this->isGranted('ROLE_ADMIN')) {
        throw $this->createAccessDeniedException();
    }
    // ...
}
```

```twig
{# Dans les templates Twig #}
{% if is_granted('ROLE_USER') %}
    <a href="{{ path('app_post_new') }}">Nouveau post</a>
{% endif %}

{% if app.user %}
    <span>Bonjour {{ app.user.username }}</span>
{% endif %}
```

### 6.7 — Voters : Autorisation Fine

Les Voters permettent de créer une logique d'autorisation métier : "peut-on modifier CE post spécifique ?"

```bash
php bin/console make:voter PostVoter
```

```php
<?php
// src/Security/Voter/PostVoter.php
namespace App\Security\Voter;

use App\Entity\Post;
use App\Entity\User;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class PostVoter extends Voter
{
    // Définir les actions supportées
    public const EDIT   = 'POST_EDIT';
    public const DELETE = 'POST_DELETE';
    public const VIEW   = 'POST_VIEW';

    // Ce voter est-il compétent pour cet attribut + sujet ?
    protected function supports(string $attribute, mixed $subject): bool
    {
        return in_array($attribute, [self::EDIT, self::DELETE, self::VIEW])
            && $subject instanceof Post;
    }

    // Renvoie true si l'accès est autorisé
    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();

        // Tout le monde peut voir
        if ($attribute === self::VIEW) {
            return true;
        }

        // Pour éditer/supprimer : il faut être connecté
        if (!$user instanceof User) {
            return false;
        }

        // L'auteur peut tout faire
        if ($subject->getAuthor() === $user) {
            return true;
        }

        // Les admins peuvent aussi supprimer
        if ($attribute === self::DELETE && in_array('ROLE_ADMIN', $user->getRoles())) {
            return true;
        }

        return false;
    }
}
```

Utiliser le Voter :

```php
#[Route('/post/{id}/edit', name: 'app_post_edit')]
public function edit(Post $post): Response
{
    // Utilise PostVoter::EDIT
    $this->denyAccessUnlessGranted('POST_EDIT', $post);

    // ou avec attribut
    // #[IsGranted('POST_EDIT', subject: 'post')]

    // ... traitement
}

#[Route('/post/{id}/delete', name: 'app_post_delete', methods: ['POST'])]
#[IsGranted('POST_DELETE', subject: 'post')]
public function delete(Post $post, EntityManagerInterface $em): Response
{
    $em->remove($post);
    $em->flush();
    return $this->redirectToRoute('app_home');
}
```

### 6.8 — À Vous de Jouer ! — Sécurité SymfoConnect

```bash
# 1. Mettre à jour l'entité User pour implémenter UserInterface
php bin/console make:user

# 2. Générer l'inscription
php bin/console make:registration-form

# 3. Générer la connexion
php bin/console make:auth

# 4. Mettre à jour les migrations
php bin/console make:migration
php bin/console doctrine:migrations:migrate

# 5. Tester : créer un compte sur /register, se connecter sur /login
```

---

## 11h30 - 12h15 | Module 7 — Services & Injection de Dépendances

### 7.1 — Concept de Service Container

Dans Symfony, un **service** est une classe instanciée et gérée par le **Service Container**. Le container crée et injecte automatiquement les dépendances.

```
Service Container
├── EntityManagerInterface  → Doctrine
├── PostRepository          → Repository des posts
├── MailerInterface         → Envoi d'emails
├── NotificationService     → Votre service
└── PostService             → Votre service
```

### 7.2 — Autowiring : Injection Automatique

Symfony détecte automatiquement les dépendances par **type-hinting** :

```php
<?php
// src/Service/PostService.php
namespace App\Service;

use App\Entity\Post;
use App\Entity\User;
use App\Repository\PostRepository;
use Doctrine\ORM\EntityManagerInterface;

class PostService
{
    // Symfony injecte automatiquement grâce au type-hinting
    public function __construct(
        private EntityManagerInterface $entityManager,
        private PostRepository         $postRepository,
    ) {}

    public function createPost(User $author, string $content): Post
    {
        if (empty(trim($content))) {
            throw new \InvalidArgumentException('Le contenu ne peut pas être vide.');
        }

        $post = new Post();
        $post->setContent(trim($content));
        $post->setAuthor($author);

        $this->entityManager->persist($post);
        $this->entityManager->flush();

        return $post;
    }

    public function toggleLike(Post $post, User $user): bool
    {
        if ($post->isLikedBy($user)) {
            $post->removeLike($user);
            $liked = false;
        } else {
            $post->addLike($user);
            $liked = true;
        }

        $this->entityManager->flush();
        return $liked;
    }

    public function getUserFeed(User $user, int $limit = 20): array
    {
        return $this->postRepository->findFeedForUser($user, $limit);
    }
}
```

### 7.3 — Service de Notification

```php
<?php
// src/Service/NotificationService.php
namespace App\Service;

use App\Entity\Notification;
use App\Entity\User;
use Doctrine\ORM\EntityManagerInterface;

class NotificationService
{
    public function __construct(
        private EntityManagerInterface $entityManager,
    ) {}

    public function createFollowNotification(User $follower, User $followed): void
    {
        $notification = new Notification();
        $notification->setRecipient($followed);
        $notification->setType('follow');
        $notification->setContent(
            sprintf('%s a commencé à vous suivre', $follower->getUsername())
        );

        $this->entityManager->persist($notification);
        $this->entityManager->flush();
    }

    public function createLikeNotification(User $liker, Post $post): void
    {
        if ($liker === $post->getAuthor()) {
            return; // Pas de notification pour ses propres likes
        }

        $notification = new Notification();
        $notification->setRecipient($post->getAuthor());
        $notification->setType('like');
        $notification->setContent(
            sprintf('%s a aimé votre post', $liker->getUsername())
        );

        $this->entityManager->persist($notification);
        $this->entityManager->flush();
    }
}
```

### 7.4 — Injecter les Services dans un Controller

```php
<?php
// src/Controller/FollowController.php
namespace App\Controller;

use App\Entity\User;
use App\Service\NotificationService;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Http\Attribute\IsGranted;

class FollowController extends AbstractController
{
    public function __construct(
        private EntityManagerInterface $entityManager,
        private NotificationService    $notificationService,
    ) {}

    #[Route('/user/{id}/follow', name: 'app_user_follow', methods: ['POST'])]
    #[IsGranted('ROLE_USER')]
    public function follow(User $userToFollow): Response
    {
        /** @var User $currentUser */
        $currentUser = $this->getUser();

        if ($currentUser === $userToFollow) {
            $this->addFlash('error', 'Vous ne pouvez pas vous suivre vous-même.');
            return $this->redirectToRoute('app_profile', ['username' => $userToFollow->getUsername()]);
        }

        if ($currentUser->isFollowing($userToFollow)) {
            // Unfollow
            $currentUser->unfollow($userToFollow);
            $message = 'Vous ne suivez plus ' . $userToFollow->getUsername();
        } else {
            // Follow
            $currentUser->follow($userToFollow);
            $this->notificationService->createFollowNotification($currentUser, $userToFollow);
            $message = 'Vous suivez maintenant ' . $userToFollow->getUsername();
        }

        $this->entityManager->flush();
        $this->addFlash('success', $message);

        return $this->redirectToRoute('app_profile', ['username' => $userToFollow->getUsername()]);
    }
}
```

### 7.5 — Configuration Avancée (services.yaml)

```yaml
# config/services.yaml
services:
    _defaults:
        autowire: true        # Injection automatique
        autoconfigure: true   # Configuration automatique (listeners, voters...)
        public: false         # Services privés par défaut

    App\:
        resource: '../src/'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'

    # Service avec un paramètre scalaire (non injectables automatiquement)
    App\Service\UploadService:
        arguments:
            $uploadDirectory: '%kernel.project_dir%/public/uploads'
```

---

## 12h15 - 12h30 | Module 8 — Events, Listeners et Subscribers

### 8.1 — EventDispatcher : Découpler le Code

Au lieu d'appeler directement `$notificationService->notify()` dans chaque controller, on peut **dispatcher un événement** et laisser des listeners réagir.

```
Controller → dispatch(UserFollowedEvent) → EventDispatcher → Listener 1 (envoyer email)
                                                            → Listener 2 (créer notification)
                                                            → Listener 3 (update stats)
```

### 8.2 — Créer un Événement Personnalisé

```php
<?php
// src/Event/UserFollowedEvent.php
namespace App\Event;

use App\Entity\User;
use Symfony\Contracts\EventDispatcher\Event;

class UserFollowedEvent extends Event
{
    public const NAME = 'user.followed';

    public function __construct(
        private User $follower,
        private User $followed,
    ) {}

    public function getFollower(): User { return $this->follower; }
    public function getFollowed(): User { return $this->followed; }
}
```

### 8.3 — Créer un EventSubscriber

```php
<?php
// src/EventSubscriber/NotificationSubscriber.php
namespace App\EventSubscriber;

use App\Event\UserFollowedEvent;
use App\Service\NotificationService;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class NotificationSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private NotificationService $notificationService,
    ) {}

    // Déclarer quels événements ce subscriber écoute
    public static function getSubscribedEvents(): array
    {
        return [
            UserFollowedEvent::NAME => 'onUserFollowed',
        ];
    }

    public function onUserFollowed(UserFollowedEvent $event): void
    {
        $this->notificationService->createFollowNotification(
            $event->getFollower(),
            $event->getFollowed()
        );
    }
}
```

### 8.4 — Dispatcher un Événement

```php
<?php
// src/Controller/FollowController.php
use Symfony\Component\EventDispatcher\EventDispatcherInterface;
use App\Event\UserFollowedEvent;

class FollowController extends AbstractController
{
    public function __construct(
        private EntityManagerInterface    $entityManager,
        private EventDispatcherInterface  $eventDispatcher,
    ) {}

    #[Route('/user/{id}/follow', name: 'app_user_follow', methods: ['POST'])]
    #[IsGranted('ROLE_USER')]
    public function follow(User $userToFollow): Response
    {
        $currentUser = $this->getUser();
        $currentUser->follow($userToFollow);
        $this->entityManager->flush();

        // Dispatcher l'événement → le Subscriber s'en occupe
        $this->eventDispatcher->dispatch(
            new UserFollowedEvent($currentUser, $userToFollow),
            UserFollowedEvent::NAME
        );

        return $this->redirectToRoute('app_profile', ['username' => $userToFollow->getUsername()]);
    }
}
```

### 8.5 — Événements Doctrine (Lifecycle Callbacks)

```php
<?php
// src/Entity/Post.php — Hooks Doctrine automatiques
#[ORM\Entity]
#[ORM\HasLifecycleCallbacks]  // ← Obligatoire pour activer les callbacks
class Post
{
    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    // Exécuté avant le premier INSERT
    #[ORM\PrePersist]
    public function onPrePersist(): void
    {
        $this->createdAt = new \DateTimeImmutable();
    }

    // Exécuté avant chaque UPDATE
    #[ORM\PreUpdate]
    public function onPreUpdate(): void
    {
        $this->updatedAt = new \DateTimeImmutable();
    }
}
```

---

## 12h30 - 13h30 | 🍽️ Pause Déjeuner

---

## 13h30 - 14h00 | Récapitulatif & Q&A

### Résumé des Concepts Clés

**Module 5 — Relations Doctrine**
- `ManyToOne` : clé étrangère dans l'entité "many"
- `OneToMany` : côté inverse, Collection de l'autre entité
- `ManyToMany` : table de jointure automatique
- `cascade` propage les opérations, `orphanRemoval` nettoie les orphelins
- QueryBuilder pour requêtes complexes + `addSelect()` contre le N+1

**Module 6 — Sécurité**
- `security.yaml` : firewalls, access_control, role_hierarchy
- `UserInterface` + `PasswordAuthenticatedUserInterface` sur l'entité User
- `make:auth` génère le formulaire de connexion
- `#[IsGranted]` et `denyAccessUnlessGranted()` pour l'autorisation
- **Voters** pour la logique métier fine (qui peut modifier CE post ?)

**Module 7 — Services & DI**
- Autowiring : Symfony détecte les dépendances par type-hinting
- Services réutilisables, testables, découplés
- Injectez dans les constructeurs, pas dans les méthodes

**Module 8 — Events**
- `EventDispatcher::dispatch()` pour déclencher un événement
- `EventSubscriberInterface::getSubscribedEvents()` pour écouter
- Lifecycle callbacks Doctrine : `#[PrePersist]`, `#[PreUpdate]`

### Points de Vigilance

> **Attention — Problème N+1 :**
> Toujours utiliser `addSelect()` dans le QueryBuilder pour éviter une requête par entité liée.
> Vérifiez avec le Profiler Symfony (onglet Doctrine → nombre de requêtes).

> **Attention — CSRF sur les formulaires de suppression :**
> Les actions destructives (delete) doivent utiliser un token CSRF dans un formulaire POST, pas un simple lien `<a href="/post/42/delete">`.

> **Attention — Voters :**
> Toujours vérifier que `$user instanceof User` avant d'accéder à ses méthodes. Un utilisateur anonyme est `null`.

> **Bonne pratique — Services :**
> Un service = une responsabilité. Ne créez pas un `MegaService` qui fait tout.

### Q&A

**Q : Quelle différence entre Listener et Subscriber ?**
R : Un **Listener** est une classe quelconque enregistrée via `services.yaml`. Un **Subscriber** implémente `EventSubscriberInterface` et déclare lui-même ses événements dans `getSubscribedEvents()`. Préférez le Subscriber.

**Q : Comment tester qu'un utilisateur est bien connecté ?**
R : Dans Twig : `{% if app.user %}`, dans PHP : `$this->getUser()` renvoie `null` si non connecté.

**Q : Peut-on avoir plusieurs firewalls ?**
R : Oui. Exemple courant : un firewall `api` (JWT, stateless) et un firewall `main` (session, form login).

---

## 14h00 - 17h00 | 📊 Évaluation Jour 2

L'évaluation complète est détaillée dans le fichier **`evaluation_jour2.md`**.

### Aperçu des Objectifs

Vous allez enrichir SymfoConnect avec les fonctionnalités sociales :

1. Implémenter le système d'inscription et de connexion
2. Créer les entités `Follow`, `Like` et `Notification`
3. Implémenter le bouton "Suivre / Ne plus suivre"
4. Implémenter le bouton "J'aime / Je n'aime plus"
5. Créer le fil d'actualité (`/feed`)
6. Implémenter le `PostVoter` (seul l'auteur peut supprimer)
7. Déclencher des notifications via un EventSubscriber

**Durée :** 3 heures | **Livrable :** Code source fonctionnel

---

## 📚 Ressources du Jour 2

| Ressource | URL |
|-----------|-----|
| Sécurité Symfony | https://symfony.com/doc/current/security.html |
| Relations Doctrine | https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/association-mapping.html |
| Service Container | https://symfony.com/doc/current/service_container.html |
| EventDispatcher | https://symfony.com/doc/current/event_dispatcher.html |
| Voters | https://symfony.com/doc/current/security/voters.html |

---

*Excellent travail ! Vous maîtrisez maintenant les piliers d'une architecture Symfony sécurisée et sociale. 🔐*
