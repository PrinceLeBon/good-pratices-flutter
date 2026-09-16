# 1. Modèle de monétisation

La décision structurante du projet iOS. Elle se prend **avant la première ligne
de code**, parce qu'elle contraint l'interface, le backend, les métadonnées et
la stratégie de réponse au reviewer.

> Les extraits cités viennent des
> [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/),
> vérifiées le 16/09/2026. Apple les révise régulièrement : relis la section
> `3.1` avant de lancer une nouvelle app.

## 1.1 La règle de base

Guideline `3.1.1` :

> *If you want to unlock features or functionality within your app, (by way of
> example: subscriptions, in-game currencies, game levels, access to premium
> content, or unlocking a full version), you must use in-app purchase.*

**Débloquer une fonctionnalité contre paiement, c'est l'achat intégré.** Les
exceptions sont listées de façon limitative en `3.1.3`. Une app qui n'entre dans
aucune d'elles passe par l'achat intégré, quel que soit l'endroit où l'on
préférerait encaisser.

## 1.2 Les six modèles

| Modèle | Guideline | L'app… | Paiement hors App Store |
|---|---|---|---|
| **A** | `3.1.3(e)` | vend des biens physiques ou des services consommés hors de l'app | **obligatoire** |
| **B** | `3.1.1` | ne relève d'aucune exception | **interdit** : achat intégré |
| **C** | `3.1.3(c)` | est vendue **uniquement à des organisations**, pour leurs membres | autorisé |
| **D** | `3.1.3(d)` | vend des prestations **en direct entre deux personnes** | autorisé |
| **E** | `3.1.3(f)` | est **gratuite**, compagnon d'un outil web payant | autorisé |
| **F** | `3.1.3(a)` | donne accès à des magazines, journaux, livres, audio, musique ou vidéo | autorisé |

`3.1.3(g)` couvre aussi les apps de gestion de campagnes publicitaires — cas trop
spécifique pour être détaillé ici.

## 1.3 Détail des modèles

### A. Biens et services consommés hors de l'app — `3.1.3(e)`

> *If your app enables people to purchase physical goods or services that will
> be consumed outside of the app, you must use purchase methods other than
> in-app purchase.*

Livraison, réparation, transport, billetterie, location. Le paiement externe est
**obligatoire** : l'achat intégré y est interdit. Aucune contrainte
d'interface — prix et bouton de paiement s'affichent normalement.

### B. Achat intégré — `3.1.1`

`StoreKit`, ou le paquet `in_app_purchase` côté Flutter. Le cas par défaut. À
anticiper dès la conception :

- une **double pile de paiement** si tu vends aussi hors iOS ;
- une **réconciliation d'états** entre Apple et ton backend (renouvellements,
  remboursements, changements de plan) ;
- la grille `3.1.2(c)` dans le tunnel d'achat (cf. [03](03-produits-et-abonnements.md)).

**Accès multiplateforme — `3.1.3(b)`.** L'app peut reconnaître des abonnements
achetés sur le web ou sur une autre plateforme :

> *… provided those items are also available as in-app purchases within the app.*

C'est un **complément** du modèle B, pas une alternative : il suppose que les
mêmes offres existent aussi en achat intégré.

### C. Services aux entreprises — `3.1.3(c)`

> *If your app is only sold directly by you to organizations or groups for their
> employees or students (for example professional databases and classroom
> management tools), you may allow enterprise users to access
> previously-purchased content or subscriptions. Consumer, single user, or
> family sales must use in-app purchase.*

L'app iOS ne vend rien : elle lit le statut d'un abonnement contracté entre
l'éditeur et l'organisation cliente. C'est le modèle de **FlashCar**.

L'exception ne tient que si le caractère B2B est **réel et visible** :

- le client est une organisation — société, association, établissement — et non
  une personne qui agit seule ;
- le contrat et la facturation lient l'éditeur à cette organisation ;
- les comptes utilisateurs sont créés ou invités par l'organisation ;
- la fiche App Store et les App Review Notes le disent explicitement ;
- **aucun particulier ne peut s'abonner seul pour lui-même.** Si c'est possible,
  c'est une vente « single user », et l'achat intégré redevient obligatoire.

Les indépendants sont une zone grise. Ne t'appuie pas sur `3.1.3(c)` si
l'essentiel de ta clientèle est composé de personnes seules.

Variante de distribution : les **apps personnalisées via Apple Business
Manager**, qui sortent complètement de l'App Store public. Pertinent si les
clients sont peu nombreux et identifiés.

### D. Prestations en direct entre deux personnes — `3.1.3(d)`

> *If your app enables the purchase of real-time person-to-person services
> between two individuals (for example tutoring students, medical
> consultations, real estate tours, or fitness training), you may use purchase
> methods other than in-app purchase. One-to-few and one-to-many real-time
> services must use in-app purchase.*

Cours particulier, consultation, visite : paiement externe autorisé. Un cours
collectif ou un live en diffusion relève de l'achat intégré.

### E. App compagnon gratuite — `3.1.3(f)`

> *Free apps acting as a stand-alone companion to a paid web based tool (i.e.
> VoIP, Cloud Storage, Email Services, Web Hosting) do not need to use in-app
> purchase, provided there is no purchasing inside the app, or calls to action
> for purchase outside of the app.*

L'app est gratuite, le service se souscrit sur le web, et l'app ne contient
**ni achat, ni incitation à acheter ailleurs**. C'est la condition la plus
stricte sur l'interface : chapitre [02](02-interface-ios-sans-achat.md) à la
lettre.

### F. Apps de lecture — `3.1.3(a)`

Magazines, journaux, livres, audio, musique, vidéo. L'app peut proposer la
création d'un compte gratuit et la gestion du compte existant. L'entitlement
*External Link Account Entitlement* autorise un lien informatif vers le site de
l'éditeur pour créer ou gérer ce compte.

## 1.4 L'erreur à ne pas refaire

La première version de ce standard présentait un modèle « service
multiplateforme » où **l'app iOS ne vend rien** et se contente de lire un
abonnement payé ailleurs. **Ce modèle n'existe pas** : `3.1.3(b)` exige que les
offres soient *aussi* vendues en achat intégré.

FlashCar est passé parce que c'est un outil **vendu à des entreprises** — modèle
C. La même recette appliquée à une app grand public qui vend du numérique serait
rejetée au titre de `3.1.1`.

## 1.5 Parler de paiement : dans l'app, hors de l'app

Introduction de `3.1.3` :

> *Apps in this section cannot, within the app, encourage users to use a
> purchasing method other than in-app purchase, except for apps on the United
> States storefront and as set forth in 3.1.1(a) and 3.1.3(a). Developers can
> send communications outside of the app to their user base about purchasing
> methods other than in-app purchase.*

- **Dans l'app** : aucun bouton, lien ou message qui oriente vers un paiement
  extérieur (cf. [02](02-interface-ios-sans-achat.md)).
- **Hors de l'app** — courriel, SMS, WhatsApp, force commerciale : autorisé.
  C'est par ce canal que se font la facturation et les relances de
  renouvellement en modèle C.
- **Vitrine des États-Unis** : l'interdiction ne s'applique pas. Un même build
  étant distribué dans tous les pays, garde la règle stricte, sauf à gérer un
  comportement propre à chaque vitrine.
- `3.1.1(a)` prévoit des *External Purchase Link Entitlements* dans certaines
  régions. Ils supposent de proposer aussi l'achat intégré.

## 1.6 Comment trancher

Pose les questions dans cet ordre, et arrête-toi à la première réponse positive :

1. Ce que je vends se consomme-t-il **hors de l'app** ? → **A**
2. Est-ce une prestation **en direct entre deux personnes** ? → **D**
3. L'app est-elle vendue **uniquement à des organisations** ? → **C**
4. L'app est-elle **gratuite**, compagnon d'un service web payant, sans aucun
   achat ni incitation ? → **E**
5. Est-ce du **contenu de lecture ou d'écoute** ? → **F**
6. Sinon → **B**, achat intégré.

## 1.7 Conséquences par modèle

| | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| Commission Apple | non | oui | non | non | non | non |
| Prix et bouton d'achat dans l'app iOS | oui | oui | **non** | oui | **non** | non |
| Produits déclarés dans App Store Connect | non | oui | non | non | non | non |
| Grille `3.1.2(c)` | non | oui | non | non | non | non |
| Chapitre [02](02-interface-ios-sans-achat.md) | non | non | **oui** | non | **oui** | en partie |
| Risque en revue | faible | faible | **élevé** si le B2B n'est pas évident | moyen | élevé | moyen |

## 1.8 Écrire la décision

Une fois tranchée, la décision se consigne à **deux endroits** :

1. le `README` du projet, en une phrase, pour l'équipe ;
2. le champ **App Review Notes** d'App Store Connect, en **citant la guideline**
   qui autorise le modèle — modèles prêts à l'emploi au chapitre
   [09](09-modeles-prets-a-l-emploi.md). Sans cette mention, le reviewer applique
   la grille par défaut, celle des abonnements vendus dans l'app.
