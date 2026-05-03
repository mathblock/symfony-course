# Évaluation Jour 2 — Fonctionnalités Sociales de SymfoConnect

**Durée :** 3h (14h00 – 17h00) | **Barème :** /20 | **Pré-requis :** Évaluation Jour 1 complétée

---

## 🎯 Objectifs Fonctionnels

En continuant le projet SymfoConnect du Jour 1, l'application doit désormais :

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

---

## 📊 Barème (20 points)

| Critère | Points |
|---------|--------|
| Inscription fonctionnelle (validation, unicité, hash mdp) | 3 |
| Connexion / déconnexion + layout adapté | 2 |
| Création de post réservée aux connectés (auteur = user courant) | 1 |
| Follow / unfollow (compteurs, protection anti-auto-follow) | 3 |
| Like / unlike (affichage compteur et état) | 2 |
| Fil d'actualité fonctionnel et protégé | 3 |
| PostVoter : suppression réservée à l'auteur (403 sinon) | 3 |
| Notification créée en BDD lors d'un follow | 3 |

---

## 📦 Livrable

Dossier du projet zippé ou lien vers dépôt Git.

---

*Bon courage ! 🔐*
