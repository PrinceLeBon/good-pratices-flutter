# 2. Interface iOS sans achat

S'applique au **modèle C** (service multiplateforme) et au **modèle D**
(cf. [01](01-modele-de-monetisation.md)). C'est le chapitre où se perd le plus
de temps en revue.

## 2.1 Les quatre règles

1. **Aucun parcours d'achat.** Le bouton est **retiré de l'arbre de widgets**,
   pas désactivé. Un CTA grisé reste un parcours d'achat aux yeux du reviewer.
2. **Aucun tarif affiché.** Ni prix, ni sélecteur de durée, ni « à partir de ».
   La date d'expiration et la liste des avantages restent permises : c'est
   informatif, pas commercial.
3. **Aucune mention d'un canal externe.** Règle anti-steering (`3.1.3`) :
   interdiction de diriger l'utilisateur vers un moyen de paiement autre que
   l'achat intégré.
4. **Aucune promesse impossible.** Un message qui invite à renouveler alors que
   l'app iOS ne le permet pas mène à un écran qui dit le contraire. Ce n'est pas
   une infraction, mais un reviewer y lit un parcours d'achat incomplet.

## 2.2 Vocabulaire banni de l'interface iOS

Le nom de toute autre plateforme ou boutique, le nom de ton site, toute URL
menant à une page de paiement, et toute formulation du type « moins cher sur »,
« abonnez-vous sur », « payez depuis ».

**Le mot d'ordre : constater, jamais orienter.**

| Rejeté | Accepté |
|---|---|
| « Vous pouvez payer depuis un téléphone Android en téléchargeant l'application sur le Play Store. » | « La gestion de l'abonnement ne s'effectue pas depuis l'application iOS. » |
| « Renouvelez-le depuis Mon entreprise. » *(quand c'est impossible sur iOS)* | « Contactez le support pour rétablir l'accès. » |

## 2.3 Conditionner par plateforme

Tout endroit qui parle d'argent, de renouvellement ou d'achat est conditionné
**dès l'écriture**, pas rétrofité sous la pression d'un rejet.

```dart
// Le CTA est RETIRÉ sur iOS, pas désactivé : l'app iOS ne commercialise rien.
if (Platform.isIOS) const AbonnementIndisponibleNotice(),
if (!Platform.isIOS) BoutonAchat(...),
```

Le prix se coupe au même niveau, via un drapeau explicite plutôt qu'un test
dispersé dans l'arbre :

```dart
// Ni tarif ni sélecteur de durée sur iOS.
final bool afficherTarif = enRenouvellement && !Platform.isIOS;
```

**Attention aux écrans atteints en état pré-ouvert.** Si un écran d'abonnement
peut être poussé directement en mode « renouvellement » depuis un garde ou une
notification, le prix s'affiche d'emblée : le drapeau doit couvrir ce chemin
aussi, pas seulement le clic sur le CTA.

## 2.4 Centraliser les textes sensibles

Les messages liés à l'abonnement finissent toujours dupliqués entre l'écran de
connexion, les gardes d'écriture et les rappels. Duplication = divergence, et
une seule occurrence oubliée suffit à faire rejeter.

- **Une source unique** par message (une classe de textes, des getters
  conditionnés par plateforme).
- Les appelants référencent, ne recopient pas.

```dart
static String get abonnementExpireMessage => Platform.isIOS
    ? "Votre abonnement est expiré. Contactez le support pour rétablir l'accès."
    : "Votre abonnement est expiré. Renouvelez-le depuis Mon entreprise.";
```

## 2.5 Vérifications avant soumission

```
grep -rn "Play Store\|App Store\|Android\|notre site" lib/   → aucun résultat
                                                                dans un texte affiché
grep -rn "https\?://" lib/ | grep -i "pay\|checkout\|abonn"  → aucune URL de paiement
grep -rn "Platform.isIOS" lib/                               → couvre TOUS les écrans
                                                                touchant à l'argent
```

Le premier grep remonte aussi les commentaires : ce sont des faux positifs
acceptables, seul le texte **affiché** compte. Garde le commentaire qui explique
pourquoi la mention est interdite — il évite qu'on la réintroduise.

## 2.6 Garde de sécurité

Même avec le CTA retiré, conserve le garde `Platform.isIOS` dans le gestionnaire
d'appui : il ne coûte rien et protège si le parcours est un jour rouvert par
inadvertance.

## 2.7 Tester sur iPad

Le reviewer teste fréquemment sur iPad. `Platform.isIOS` couvre iPadOS, mais les
mises en page conditionnelles, elles, ne sont pas toujours vérifiées sur cette
taille d'écran. Lance au moins une fois sur simulateur iPad avant l'envoi.
