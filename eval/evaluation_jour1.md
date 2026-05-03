# Évaluation Jour 1 — Fondations de SymfoConnect

**Durée :** 3h (14h00 – 17h00) | **Barème :** /20 | **Mode :** Individuel ou binôme

---

## 🌐 Le Projet : SymfoConnect

SymfoConnect est un réseau social développé progressivement sur 3 jours. À l'issue de la formation, vous aurez une application Symfony 7 complète et fonctionnelle.

### Vision globale sur les 3 jours

| Jour | Ce qui est construit |
|------|---------------------|
| **Jour 1** | Base du projet, entités, pages publiques, formulaire de post |
| **Jour 2** | Authentification, follows, likes, fil d'actualité, sécurité |
| **Jour 3** | Messagerie privée, API, cache, tests, déploiement |

### Schéma de base de données final (pour information)

```
users ──< posts ──< likes (ManyToMany users)
  │
  └──< user_follows (ManyToMany self)
  └──< messages (sender / recipient)
  └──< notifications (recipient)
```

---

## 🎯 Objectifs Fonctionnels

À la fin de cette évaluation, l'application doit :

1. **Démarrer sans erreur** — le projet Symfony 7 est installé, la base de données est configurée et les migrations sont exécutées.

2. **Afficher une page d'accueil** (`/`) — liste des 10 derniers posts, triés par date décroissante, avec le nom de l'auteur et la date de publication.

3. **Afficher une page de profil** (`/profil/{username}`) — présente les informations de l'utilisateur (username, bio, avatar si renseigné) ainsi que ses posts. Retourne une erreur 404 si l'utilisateur n'existe pas.

4. **Permettre la création d'un post** (`/post/nouveau`) — formulaire avec validation (contenu obligatoire, longueur minimale). Après soumission valide, le post est sauvegardé, un message flash confirme la création et l'utilisateur est redirigé vers l'accueil.

5. **Avoir un layout cohérent** — toutes les pages héritent d'un template de base avec navigation et zone d'affichage des messages flash.

### Entités attendues

**User** — id, email (unique), username (unique), password, bio (nullable), avatarUrl (nullable), createdAt

**Post** — id, content, createdAt, author (ManyToOne → User)

---

## 📊 Barème (20 points)

| Critère | Points |
|---------|--------|
| Projet installé, BDD configurée, migrations sans erreur | 3 |
| Entité User : tous les champs, types corrects | 2 |
| Entité Post + relation ManyToOne vers User correcte | 2 |
| Page d'accueil fonctionnelle (liste, tri, auteur, date) | 4 |
| Page de profil fonctionnelle (infos user, ses posts, 404) | 3 |
| Formulaire de post avec validation, flash et redirection | 4 |
| Qualité du code (lisibilité, conventions PSR-12) | 2 |

---

## 📦 Livrable

Dossier du projet zippé ou lien vers dépôt Git.

---

*Bon courage ! 💪*
