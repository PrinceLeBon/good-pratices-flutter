# Standards Publication App Store — iOS

Ce qu'il faut décider, coder et préparer pour qu'une app passe la revue
Apple **au premier essai**. Centré sur le point qui coince le plus :
**encaisser de l'argent quand on ne veut pas de l'achat intégré**.

Issu de la mise en production de FlashCar (cf.
[retour-experience-flashcar.md](retour-experience-flashcar.md)), qui a coûté
trois passages en revue. À appliquer sur tous mes projets iOS.

## Sommaire

1. [Modèle de monétisation](01-modele-de-monetisation.md) — la décision qui commande tout
2. [Interface iOS sans achat](02-interface-ios-sans-achat.md) — anti-steering, conditionnement par plateforme
3. [Produits et abonnements dans ASC](03-produits-et-abonnements.md) — ne pas déclarer ce qu'on ne vend pas
4. [Métadonnées et pages publiques](04-metadonnees-et-pages-publiques.md) — confidentialité, support, EULA
5. [Comptes de démonstration](05-comptes-de-demonstration.md) — le premier poste de retard
6. [Signature et build](06-signature-et-build.md) — certificat Distribution, pièges Flutter/Xcode
7. [Revue et réponses au reviewer](07-revue-et-reponses.md) — la file d'attente commande la stratégie
8. [Checklist de soumission](08-checklist-de-soumission.md) — à dérouler avant chaque envoi

## L'arbre de décision en une image

```
Que vend l'app ?
 │
 ├── Bien physique / service du monde réel
 │     → paiement externe OBLIGATOIRE (achat intégré interdit)
 │       Aucune contrainte. Cas le plus simple.
 │
 └── Numérique consommé dans l'app
       │
       ├── Vendu DANS l'app iOS
       │     → achat intégré Apple obligatoire (StoreKit / in_app_purchase)
       │       Commission Apple. Double pile de paiement si tu vends ailleurs.
       │
       └── Vendu AILLEURS (web, autre plateforme, commercial)
             → « service multiplateforme » : l'app iOS ne vend rien,
               elle lit un statut d'accès.
               Zéro commission, mais règles strictes → chapitre 02.
```

## Les invariants non négociables

1. **Le modèle de monétisation se choisit avant la première ligne de code iOS.**
   Le rétrofiter sous la pression d'un rejet coûte plusieurs semaines.
2. **Si l'app iOS ne vend pas, elle n'affiche ni prix, ni bouton d'achat**, et
   ne les désactive pas : elle les retire de l'arbre de widgets.
3. **Aucune mention d'un canal d'achat externe**, nulle part dans l'interface
   iOS. Ni nom de plateforme, ni URL, ni « moins cher sur ».
4. **Aucun message ne promet une action impossible** sur la plateforme courante.
5. **Aucun produit d'abonnement déclaré** si aucun tunnel d'achat n'existe dans
   l'app — le déclarer déclenche des obligations qu'on ne peut pas satisfaire.
6. **Trois URL publiques existent avant la première soumission** :
   confidentialité, support, CGU. Elles sont vérifiées à chaque passage.
7. **Le compte de démonstration est testé sur un build release**, depuis un
   appareil vierge, avant l'envoi.
8. **Une demande d'information du reviewer se traite par une réponse**, jamais
   par un nouveau binaire (cf. [07](07-revue-et-reponses.md)).

## Checklist (nouvelle app)

- [ ] Modèle de monétisation tranché et écrit dans les App Review Notes.
- [ ] Écrans touchant à l'argent conditionnés par plateforme dès l'écriture.
- [ ] Textes sensibles centralisés en un seul endroit (pas de duplication).
- [ ] Site public en ligne : accueil, support, confidentialité, CGU.
- [ ] CGU et confidentialité atteignables depuis l'app elle-même.
- [ ] Deux comptes de démo prêts : nominal et état limite.
- [ ] Certificat *Apple Distribution* présent dans le trousseau.
- [ ] Checklist [08](08-checklist-de-soumission.md) déroulée avant l'envoi.

## Note sur les numéros de guideline

Les numéros cités ici (`1.5`, `2.1`, `3.1.1`, `3.1.2(c)`, `3.1.3`) proviennent
de rejets réels. **Les lettres des sous-sections de 3.1.3 changent au fil des
révisions d'Apple** : les modèles sont donc désignés par leur nom, pas par leur
lettre. Vérifie la numérotation en vigueur sur les
[App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
avant de la citer dans une réponse au reviewer.
