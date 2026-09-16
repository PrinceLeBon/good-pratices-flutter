# Standards Publication App Store — iOS

Ce qu'il faut décider, coder et préparer pour qu'une app passe la revue
Apple **au premier essai**. Centré sur le point qui coince le plus :
**encaisser hors de l'achat intégré, et savoir dans quels cas Apple le permet**.

Issu de la mise en production de FlashCar (cf.
[retour-experience-flashcar.md](retour-experience-flashcar.md)), qui a coûté
trois passages en revue. À appliquer sur tous mes projets iOS.

## Sommaire

1. [Modèle de monétisation](01-modele-de-monetisation.md) — les six modèles et la guideline qui autorise chacun
2. [Interface iOS sans achat](02-interface-ios-sans-achat.md) — anti-steering, conditionnement par plateforme
3. [Produits et abonnements dans ASC](03-produits-et-abonnements.md) — ne pas déclarer ce qu'on ne vend pas
4. [Métadonnées et pages publiques](04-metadonnees-et-pages-publiques.md) — confidentialité, support, EULA
5. [Comptes de démonstration](05-comptes-de-demonstration.md) — le premier poste de retard
6. [Signature et build](06-signature-et-build.md) — certificat Distribution, pièges Flutter/Xcode
7. [Revue et réponses au reviewer](07-revue-et-reponses.md) — la file d'attente commande la stratégie
8. [Checklist de soumission](08-checklist-de-soumission.md) — à dérouler avant chaque envoi
9. [Modèles prêts à l'emploi](09-modeles-prets-a-l-emploi.md) — Review Notes, réponse, textes, code, EULA

## L'arbre de décision en une image

```
Que vend l'app ?
 │
 ├── Un bien physique, un service consommé hors de l'app ....... A · 3.1.3(e)
 │     paiement externe OBLIGATOIRE, interface libre
 │
 ├── Une prestation en direct entre deux personnes ............. D · 3.1.3(d)
 │     paiement externe autorisé (pas pour un cours collectif)
 │
 └── Du numérique consommé dans l'app
       │
       ├── vendu UNIQUEMENT à des organisations ................ C · 3.1.3(c)
       │     l'app iOS ne vend rien, elle lit un statut  ← FlashCar
       │
       ├── app gratuite, compagnon d'un service web payant ..... E · 3.1.3(f)
       │     ni achat, ni incitation à acheter ailleurs
       │
       ├── magazine, journal, livre, audio, musique, vidéo ..... F · 3.1.3(a)
       │
       └── vendu à des particuliers ............................ B · 3.1.1
             ACHAT INTÉGRÉ OBLIGATOIRE
             (3.1.3(b) : les achats faits ailleurs peuvent être
              reconnus, s'ils existent aussi en achat intégré)
```

## Les invariants non négociables

1. **Le modèle de monétisation se choisit avant la première ligne de code iOS**,
   et la guideline qui l'autorise est identifiée par son numéro.
2. **Une app qui vend du numérique à des particuliers passe par l'achat
   intégré.** L'app « qui ne vend rien sur iOS » n'est valable que dans les
   modèles C et E (cf. [01](01-modele-de-monetisation.md)).
3. **Si l'app iOS ne vend pas, elle n'affiche ni prix, ni bouton d'achat**, et
   ne les désactive pas : elle les retire de l'arbre de widgets.
4. **Aucune mention d'un canal d'achat externe** dans l'interface iOS — écrans,
   popups et notifications compris. Ni nom de plateforme, ni URL, ni
   « moins cher sur ».
5. **Aucun message ne promet une action impossible** sur la plateforme courante.
6. **Aucun produit d'abonnement déclaré** si aucun tunnel d'achat n'existe dans
   l'app — le déclarer déclenche des obligations qu'on ne peut pas satisfaire.
7. **Trois URL publiques existent avant la première soumission** :
   confidentialité, support, CGU. Elles sont vérifiées à chaque passage.
8. **Le compte de démonstration est testé sur un build release**, depuis un
   appareil vierge, avant l'envoi.
9. **Une demande d'information du reviewer se traite par une réponse**, jamais
   par un nouveau binaire (cf. [07](07-revue-et-reponses.md)).

## Checklist (nouvelle app)

- [ ] Modèle tranché, guideline citée dans les App Review Notes ([09](09-modeles-prets-a-l-emploi.md)).
- [ ] Écrans, popups et notifications touchant à l'argent conditionnés par plateforme.
- [ ] Textes sensibles centralisés en un seul endroit (pas de duplication).
- [ ] Site public en ligne : accueil, support, confidentialité, CGU.
- [ ] CGU et confidentialité atteignables depuis l'app elle-même.
- [ ] Deux comptes de démo prêts : nominal et état limite.
- [ ] Certificat *Apple Distribution* présent dans le trousseau.
- [ ] Checklist [08](08-checklist-de-soumission.md) déroulée avant l'envoi.

## Sources et mises à jour

Les extraits de la section `3.1` cités au chapitre 01 ont été vérifiés le
16/09/2026 sur les
[App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/).
Apple les révise régulièrement : relis la section avant chaque nouvelle app, et
avant de citer un numéro dans une réponse au reviewer.

**Correction du 16/09/2026.** La version précédente de ce standard classait
FlashCar comme « service multiplateforme », présenté comme une app iOS qui ne
vend rien. C'était faux : ce modèle exige que les offres existent aussi en achat
intégré. FlashCar relève des services aux entreprises, `3.1.3(c)` — voir
[01 §1.4](01-modele-de-monetisation.md).
