# Retour d'expérience — FlashCar

Source de ce standard. App de gestion de parc automobile, Flutter, abonnement
annuel encaissé par une passerelle de paiement locale. **Trois passages en revue**
avant la mise en production sur l'App Store.

Sur Android, aucun problème : le paiement était déjà en place et la publication
n'a soulevé aucune difficulté. Toute la friction est venue d'iOS.

## Chronologie

### Soumission 1 — rejetée

`3.1.2(c)` · `2.1` · `1.5`

- Des produits d'abonnement auto-renouvelables étaient déclarés dans App Store
  Connect, sans que l'app affiche les informations requises — et pour cause :
  aucun tunnel d'achat n'existait sur iOS.
- Le compte de démonstration fourni ne permettait pas de se connecter.
- La **Support URL** pointait vers le README d'un dépôt Git, ce qui n'est pas une
  page de support.

### Soumission 2 — rejetée

`3.1.2(c)` · `2.1`

Entre-temps : produits d'abonnement retirés de la soumission, parcours d'achat
supprimé de l'app iOS, site public mis en ligne. La Support URL disparaît de la
liste — donc acceptée.

Restait :

- l'**EULA absent des métadonnées** ;
- un compte de démonstration à l'abonnement **actif** : le reviewer n'atteignait
  jamais l'écran bloquant et réclamait un compte **expiré** pour voir le
  parcours complet.

### Soumission 3 — validée

EULA personnalisé déposé dans *App Information*, comptes actif **et** expiré
fournis tous les deux, capture vidéo à l'appui. Aucun nouveau binaire n'a été
soumis entre les soumissions 2 et 3 : seule la réponse et les métadonnées ont
changé.

## Le piège qu'on n'a pas vu venir

L'app affichait, quand un utilisateur iOS tentait de payer :

> « Le paiement de l'abonnement n'est pas encore disponible sur iOS. Vous pouvez
> payer depuis un téléphone Android en téléchargeant l'application sur le
> Play Store. »

C'est une infraction caractérisée à la règle anti-steering (`3.1.3`) : diriger
nommément l'utilisateur vers un canal d'achat externe. Le message paraissait
serviable ; il exposait l'app à un rejet indépendant de tous les autres.

Deuxième cas, plus subtil, trouvé après coup : un popup « Abonnement expiré »
invitait à « renouveler depuis Mon entreprise », avec un bouton *Renouveler* qui
menait à un écran affirmant que le renouvellement n'était pas possible sur iOS.
Pas une infraction, mais un parcours d'achat incomplet aux yeux d'un reviewer.

## Où le temps est réellement parti

| # | Cause | Coût |
|---|---|---|
| 1 | Compte de démonstration — identifiants morts, puis abonnement actif là où il fallait un expiré | 2 rejets |
| 2 | Métadonnées — Support URL invalide, puis EULA absent | 2 rejets |
| 3 | Une phrase dans un popup — la mention du Play Store | 1 refonte de l'écran d'abonnement |
| 4 | Certificat de signature — compte Apple absent de Xcode, échec silencieux à l'export | ~1 journée |

**Aucun de ces quatre points n'était un problème de code métier.** Sur iOS, le
goulot d'étranglement, ce sont les métadonnées, les accès de test et une poignée
de formulations dans l'interface.

## Ce qu'on ferait différemment

1. **Trancher le modèle de monétisation avant de coder l'app iOS.** Le paiement
   externe était déjà partout quand la question s'est posée, d'où les
   contorsions.
2. **Monter le site public dès le premier jour.** Quatre pages statiques
   couvrent trois champs obligatoires et resservent sur toutes les apps.
3. **Préparer deux comptes de démo dès le départ**, nominal et état limite.
4. **Vérifier le certificat Distribution avant le premier build de release**, et
   ne jamais se fier au code de sortie de `flutter build ipa`.
5. **Conditionner par plateforme dès l'écriture** tout écran qui parle d'argent,
   plutôt que de le rétrofiter sous la pression d'un rejet.
