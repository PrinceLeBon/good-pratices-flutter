# 3. Produits et abonnements dans App Store Connect

## 3.1 Ne déclare pas ce que tu ne vends pas

Déclarer un produit d'abonnement auto-renouvelable dans App Store Connect
**déclenche toute la grille `3.1.2(c)`**. L'app doit alors afficher, dans son
tunnel d'achat :

- le titre de l'abonnement ;
- sa durée ;
- son prix, et le prix par unité si pertinent ;
- des liens **fonctionnels** vers la politique de confidentialité et l'EULA.

Si l'app iOS ne vend rien, ce tunnel n'existe pas — et l'exigence est
**impossible à satisfaire**. Le rejet est alors garanti et se répète tant que le
produit reste déclaré.

## 3.2 Le piège du produit fantôme

Retirer un abonnement de la version **ne suffit pas toujours**. Un produit laissé
en état `Waiting for Review` ou `In Review` dans **Monetization → Subscriptions**
se réattache automatiquement à la soumission.

**Vérification obligatoire avant chaque envoi :** sur la page de la version, la
section *In-App Purchases and Subscriptions* doit être **vide**.

C'est l'explication la plus probable si Apple continue de t'appliquer la grille
des abonnements alors que tu as retiré tes produits.

## 3.3 Si tu vends vraiment dans l'app (modèle B)

La grille `3.1.2(c)` s'applique et se traite dans l'écran d'achat lui-même :

- **Titre** — peut reprendre le nom du produit d'achat intégré.
- **Durée** — explicite (« 1 mois », « 12 mois »), pas déduite du prix.
- **Prix** — affiché dans la devise locale servie par StoreKit, jamais codé en
  dur. Ajouter le prix par unité quand l'offre est pluriannuelle.
- **Deux liens fonctionnels**, dans le tunnel d'achat et non enfouis dans un
  menu : confidentialité et EULA. « Fonctionnel » signifie qu'ils ouvrent
  réellement une page — le reviewer clique.

Ces quatre éléments doivent être visibles **sans défilement supplémentaire** au
moment où l'utilisateur décide.

**Achats faits ailleurs.** Si l'offre se vend aussi sur le web ou sur une autre
plateforme, l'app peut reconnaître ces achats au titre de `3.1.3(b)` — à
condition que la même offre existe en achat intégré (cf.
[01 §1.3](01-modele-de-monetisation.md)).

## 3.4 Contrôle croisé métadonnées

Même quand tout est correct dans l'app, `3.1.2(c)` impose en parallèle, côté
métadonnées (cf. [04](04-metadonnees-et-pages-publiques.md)) :

- l'URL de politique de confidentialité dans le champ dédié ;
- l'EULA, soit par le lien Apple standard dans la description, soit par un texte
  personnalisé dans App Store Connect.

Les deux moitiés sont vérifiées séparément. Satisfaire l'app sans les
métadonnées produit exactement le même rejet.

## 3.5 Essai gratuit

Un essai gratuit **sans paiement dans l'app** ne constitue pas un achat et reste
utilisable sur iOS en modèles C et E. Il donne au reviewer un moyen d'explorer les
fonctionnalités sans compte pré-abonné.

Attention à sa condition d'affichage : si le bouton d'essai n'est rendu que dans
un état atteint via le CTA d'achat — retiré sur iOS — il devient **inatteignable**.
Vérifie que le chemin existe encore une fois le CTA supprimé.
