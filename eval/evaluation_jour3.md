# Évaluation Jour 3 — Finalisation de SymfoConnect

**Durée :** 3h (14h00 – 17h00) | **Barème :** /20 | **Pré-requis :** Évaluations Jours 1 & 2 complétées

---

## 🎯 Objectifs Fonctionnels

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

---

## 📊 Barème (20 points)

| Critère | Points |
|---------|--------|
| Messagerie privée (liste conversations, affichage, envoi, marquage lu) | 6 |
| API REST (liste, détail, création, filtres, documentation) | 4 |
| Cache sur le fil d'actualité avec invalidation | 2 |
| Messenger : email asynchrone au destinataire d'un message | 3 |
| 5 tests passant en vert | 4 |
| Fichiers de configuration de production présents et cohérents | 1 |

---

## 🎓 Bilan du Projet

À l'issue des 3 jours, SymfoConnect couvre :

| Fonctionnalité | Jour |
|----------------|------|
| Structure du projet, routing, templates | J1 |
| Entités User et Post, migrations, CRUD | J1 |
| Pages publiques (accueil, profil), formulaires | J1 |
| Inscription, connexion, déconnexion | J2 |
| Follows, likes, fil d'actualité | J2 |
| Voters, notifications, EventSubscriber | J2 |
| Messagerie privée | J3 |
| API REST avec API Platform | J3 |
| Cache, Messenger, tests, déploiement | J3 |

---

## 📦 Livrable

Dossier du projet zippé ou lien vers dépôt Git.

---

*Félicitations pour ces 3 jours ! 🎉*
