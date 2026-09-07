# 1. Modèle de monétisation

La décision structurante du projet iOS. Elle se prend **avant la première ligne
de code**, parce qu'elle contraint l'interface, le backend, les métadonnées et
la stratégie de réponse au reviewer.

## 1.1 La règle de base

Guideline `3.1.1` : **tout contenu ou service numérique consommé dans l'app doit
passer par l'achat intégré Apple**, avec sa commission. Les exceptions existent,
elles sont précises, et il faut savoir dans laquelle on se range avant de coder.

## 1.2 Les quatre modèles

### A. Bien physique ou service du monde réel

Livraison, réparation, transport, billetterie, prestation exécutée hors de
l'app. Le paiement externe est **non seulement autorisé mais obligatoire** :
l'achat intégré y est interdit.

> Beaucoup d'apps se compliquent la vie sans raison ici. Si ce que tu vends se
> consomme hors de l'écran, il n'y a **aucune** contrainte de paiement.

### B. Achat intégré Apple

`StoreKit`, ou le paquet `in_app_purchase` côté Flutter. Le seul moyen de vendre
du numérique **dans** l'app iOS.

À anticiper dès la conception :

- une **double pile de paiement** à maintenir si tu vends aussi hors iOS ;
- une **réconciliation d'états** entre les abonnements Apple et ton backend
  (renouvellements, remboursements, changements de plan) ;
- la grille `3.1.2(c)` à satisfaire dans le tunnel d'achat (cf. [03](03-produits-et-abonnements.md)).

### C. Service multiplateforme

L'app iOS **ne vend rien** : elle donne accès à un abonnement souscrit ailleurs
(web, force commerciale, autre plateforme) et se contente d'en **lire le
statut**. Modèle parfaitement admis, à condition de respecter les règles du
chapitre [02](02-interface-ios-sans-achat.md) à la lettre.

C'est le modèle retenu sur FlashCar. Zéro commission, mais la première
soumission est un champ de mines.

### D. Services aux entreprises

Outil vendu à des organisations pour leurs employés, jamais au grand public.
Voisin du modèle C, avec un positionnement B2B qui doit être **explicite dans la
fiche App Store**, pas seulement dans ta tête. Étudier aussi la distribution
privée via **Apple Business Manager**, qui sort complètement de l'App Store
public — pertinent si le nombre de clients est faible et identifié.

## 1.3 Comment trancher

| Question | Réponse | Modèle |
|---|---|---|
| Ce que je vends se consomme-t-il hors de l'app ? | oui | **A** |
| L'utilisateur doit-il pouvoir acheter depuis l'app iOS ? | oui | **B** |
| Mes clients sont-ils exclusivement des sociétés identifiées ? | oui | **D** |
| Sinon | — | **C** |

## 1.4 Conséquences par modèle

| | A | B | C | D |
|---|---|---|---|---|
| Commission Apple | non | oui | non | non |
| Prix affichable dans l'app iOS | oui | oui | **non** | non |
| Bouton d'achat dans l'app iOS | oui | oui | **non** | non |
| Produits déclarés dans ASC | non | oui | **non** | non |
| Grille `3.1.2(c)` applicable | non | oui | non | non |
| Effort d'implémentation | faible | élevé | faible | faible |
| Risque en revue | faible | faible | **élevé** | moyen |

## 1.5 Le piège du modèle C

Le risque n'est pas technique, il est **rédactionnel**. Retirer le paiement de
l'app iOS est trivial ; ne jamais laisser une seule phrase désigner le canal
externe l'est beaucoup moins. C'est l'objet du chapitre suivant.

## 1.6 Écrire la décision

Une fois tranchée, la décision se consigne à **deux endroits** :

1. le `README` du projet, en une phrase, pour l'équipe ;
2. le champ **App Review Notes** d'App Store Connect, en trois lignes, pour le
   reviewer — il évite qu'on t'applique la mauvaise grille de lecture.
