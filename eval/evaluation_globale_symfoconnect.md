# Évaluation Globale — SymfoConnect

**Durée totale :** 9h (3 x 3h) | **Barème total :** /60 | **Mode :** Individuel ou binôme

---

## Le Projet : SymfoConnect

SymfoConnect est un réseau social développé progressivement sur 3 jours. À l'issue de la formation, vous aurez une application Symfony 7 complète et fonctionnelle.

### Vision globale sur les 3 parties

| Partie       | Ce qui est construit                                         |
| ------------ | ------------------------------------------------------------ |
| **Partie 1** | Base du projet, entités, pages publiques, formulaire de post |
| **Partie 2** | Authentification, follows, likes, fil d'actualité, sécurité  |
| **Partie 3** | Messagerie privée, API, cache, tests, déploiement            |

### Schéma de base de données final (pour information)

```text
users ──< posts ──< likes (ManyToMany users)
  │
  └──< user_follows (ManyToMany self)
  └──< messages (sender / recipient)
  └──< notifications (recipient)
```

---

## Partie 1 — Fondations de SymfoConnect

**Durée :** 3h (14h00 – 17h00) | **Barème :** /20

### Objectifs fonctionnels

À la fin de cette première partie, l'application doit :

1. **Démarrer sans erreur** — le projet Symfony 7 est installé, la base de données est configurée et les migrations sont exécutées.
2. **Afficher une page d'accueil** (`/`) — liste des 10 derniers posts, triés par date décroissante, avec le nom de l'auteur et la date de publication.
3. **Afficher une page de profil** (`/profil/{username}`) — présente les informations de l'utilisateur (username, bio, avatar si renseigné) ainsi que ses posts. Retourne une erreur 404 si l'utilisateur n'existe pas.
4. **Permettre la création d'un post** (`/post/nouveau`) — formulaire avec validation (contenu obligatoire, longueur minimale). Après soumission valide, le post est sauvegardé, un message flash confirme la création et l'utilisateur est redirigé vers l'accueil.
5. **Avoir un layout cohérent** — toutes les pages héritent d'un template de base avec navigation et zone d'affichage des messages flash.

### Entités attendues

**User** — id, email (unique), username (unique), password, bio (nullable), avatarUrl (nullable), createdAt

**Post** — id, content, createdAt, author (ManyToOne → User)

### Barème — Partie 1

| Critère                                                   | Points |
| --------------------------------------------------------- | ------ |
| Projet installé, BDD configurée, migrations sans erreur   | 3      |
| Entité User : tous les champs, types corrects             | 2      |
| Entité Post + relation ManyToOne vers User correcte       | 2      |
| Page d'accueil fonctionnelle (liste, tri, auteur, date)   | 4      |
| Page de profil fonctionnelle (infos user, ses posts, 404) | 3      |
| Formulaire de post avec validation, flash et redirection  | 4      |
| Qualité du code (lisibilité, conventions PSR-12)          | 2      |

---

## Partie 2 — Fonctionnalités Sociales de SymfoConnect

**Durée :** 3h (14h00 – 17h00) | **Barème :** /20 | **Pré-requis :** Partie 1 complétée

### Objectifs fonctionnels

En poursuivant le projet SymfoConnect de la Partie 1, l'application doit désormais :

1. **Inscription** — un visiteur peut créer un compte en renseignant son email, son username et un mot de passe. Le mot de passe est stocké de façon sécurisée (hashé). L'email et le username doivent être uniques.
2. **Connexion / Déconnexion** — un utilisateur peut se connecter avec son email et son mot de passe, puis se déconnecter. Le layout affiche son username quand il est connecté.
3. **Création de post sécurisée** — seul un utilisateur connecté peut créer un post. L'auteur est automatiquement l'utilisateur connecté.
4. **Suivre / Ne plus suivre** — un utilisateur connecté peut suivre ou ne plus suivre un autre utilisateur depuis sa page de profil. Il ne peut pas se suivre lui-même. La page de profil affiche le nombre de followers et de follows, ainsi que le bouton d'action correspondant à l'état actuel.
5. **Liker / Ne plus liker** — un utilisateur connecté peut liker ou retirer son like sur un post. Le nombre de likes est affiché sur chaque post.
6. **Fil d'actualité** (`/feed`) — accessible uniquement aux utilisateurs connectés. Affiche les posts des personnes que l'utilisateur suit, triés par date décroissante. Si l'utilisateur ne suit personne, un message l'invite à commencer à suivre des gens.
7. **Suppression sécurisée** — seul l'auteur d'un post peut le supprimer (contrôle via un Voter). Un autre utilisateur qui tente de supprimer un post qu'il ne possède pas reçoit une erreur 403.
8. **Notification de follow** — quand un utilisateur en suit un autre, une `Notification` est créée en base de données pour l'utilisateur suivi (type, contenu, destinataire, date).

### Nouvelles entités attendues

**Follow** — géré par une relation ManyToMany auto-référencée sur User (table `user_follows`)

**Like** — géré par une relation ManyToMany entre Post et User (table `post_likes`)

**Notification** — id, recipient (ManyToOne → User), type, content, isRead (défaut false), createdAt

### Barème — Partie 2

| Critère                                                         | Points |
| --------------------------------------------------------------- | ------ |
| Inscription fonctionnelle (validation, unicité, hash mdp)       | 3      |
| Connexion / déconnexion + layout adapté                         | 2      |
| Création de post réservée aux connectés (auteur = user courant) | 1      |
| Follow / unfollow (compteurs, protection anti-auto-follow)      | 3      |
| Like / unlike (affichage compteur et état)                      | 2      |
| Fil d'actualité fonctionnel et protégé                          | 3      |
| PostVoter : suppression réservée à l'auteur (403 sinon)         | 3      |
| Notification créée en BDD lors d'un follow                      | 3      |

---

## Partie 3 — Finalisation de SymfoConnect

**Durée :** 3h (14h00 – 17h00) | **Barème :** /20 | **Pré-requis :** Parties 1 et 2 complétées

### Objectifs fonctionnels

Pour finaliser SymfoConnect, l'application doit intégrer les fonctionnalités suivantes :

1. **Messagerie privée** — un utilisateur connecté peut envoyer un message privé à un autre utilisateur. Il dispose d'une page listant toutes ses conversations et d'une page affichant l'historique d'une conversation avec possibilité de répondre. Les messages reçus sont marqués comme lus à la lecture.
2. **API REST** — les posts sont exposés via une API JSON paginée. Il est possible de lister tous les posts (`GET /api/posts`), consulter un post précis (`GET /api/posts/{id}`) et créer un post si l'on est authentifié (`POST /api/posts`). Une documentation interactive (Swagger UI) est accessible. Des filtres permettent de rechercher par contenu ou par auteur.
3. **Cache sur le fil d'actualité** — le fil d'actualité (`/feed`) utilise un cache applicatif avec une durée d'expiration de 5 minutes. Le cache est invalidé dès qu'un nouveau post est créé.
4. **Traitement asynchrone** — lors de l'envoi d'un message privé, une notification par email est envoyée au destinataire de manière asynchrone via le composant Messenger (le worker peut être lancé manuellement pour vérifier).
5. **Tests automatisés** — le projet comporte au minimum 5 tests qui passent tous en vert :
   - 1 test unitaire sur une règle métier (entité ou service)
   - 1 test fonctionnel vérifiant qu'une page publique répond en 200
   - 1 test fonctionnel vérifiant que la création de post redirige vers `/login` si non connecté
   - 1 test fonctionnel vérifiant qu'un utilisateur connecté peut accéder au formulaire de post
   - 1 test fonctionnel sur l'API (`GET /api/posts` retourne du JSON valide)
6. **Configuration de production** — le projet contient un fichier `.env.prod.local.example` avec les variables nécessaires à un déploiement (BDD, mailer, secret, debug=0) et un script `deploy.sh` listant les commandes à exécuter lors d'un déploiement (installation des dépendances, migrations, warmup du cache, compilation des assets).

### Barème — Partie 3

| Critère                                                                | Points |
| ---------------------------------------------------------------------- | ------ |
| Messagerie privée (liste conversations, affichage, envoi, marquage lu) | 6      |
| API REST (liste, détail, création, filtres, documentation)             | 4      |
| Cache sur le fil d'actualité avec invalidation                         | 2      |
| Messenger : email asynchrone au destinataire d'un message              | 3      |
| 5 tests passant en vert                                                | 4      |
| Fichiers de configuration de production présents et cohérents          | 1      |

---

## Bilan Global du Projet

À l'issue des 3 parties, SymfoConnect couvre :

| Fonctionnalité                                 | Partie |
| ---------------------------------------------- | ------ |
| Structure du projet, routing, templates        | 1      |
| Entités User et Post, migrations, CRUD         | 1      |
| Pages publiques (accueil, profil), formulaires | 1      |
| Inscription, connexion, déconnexion            | 2      |
| Follows, likes, fil d'actualité                | 2      |
| Voters, notifications, EventSubscriber         | 2      |
| Messagerie privée                              | 3      |
| API REST avec API Platform                     | 3      |
| Cache, Messenger, tests, déploiement           | 3      |

---

## Livrable

Dossier du projet zippé ou lien vers dépôt Git.
