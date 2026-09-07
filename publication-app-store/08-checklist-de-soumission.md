# 8. Checklist de soumission

À dérouler avant **chaque** envoi, y compris les suivants.

## 8.1 Décision et code

- [ ] Modèle de monétisation tranché ([01](01-modele-de-monetisation.md)) et
      écrit dans les App Review Notes.
- [ ] Aucun prix ni bouton d'achat visible sur iOS, **y compris sur iPad**.
- [ ] Le CTA est retiré de l'arbre de widgets, pas désactivé.
- [ ] Les écrans atteints en état pré-ouvert (gardes, notifications) respectent
      aussi la règle.
- [ ] Aucun message ne promet une action impossible sur iOS.
- [ ] Textes sensibles centralisés en une source unique.

```
grep -rn "Play Store\|Android\|notre site" lib/   → rien dans un texte affiché
grep -rn "https\?://" lib/ | grep -i "pay\|abonn" → aucune URL de paiement
```

## 8.2 App Store Connect

- [ ] Section *In-App Purchases and Subscriptions* de la version **vide**
      (modèles A, C, D), ou grille `3.1.2(c)` satisfaite dans l'app (modèle B).
- [ ] Aucun produit en `Waiting for Review` dans Monetization.
- [ ] **Privacy Policy URL** renseignée et ouverte dans un navigateur pour
      vérification.
- [ ] **Support URL** menant à une vraie page de support.
- [ ] **EULA** en place : lien Apple standard dans la description, ou texte
      personnalisé dans *App Information*.
- [ ] **App Review Notes** remplies : métier, modèle de monétisation, où
      trouver les documents légaux.

## 8.3 Comptes de test

- [ ] Compte nominal testé sur un **build release**, depuis un appareil vierge.
- [ ] Abonnement du compte nominal couvrant largement la période de revue.
- [ ] Second compte fourni pour l'état limite (abonnement expiré, essai
      consommé).
- [ ] Jeu de données minimal mais crédible sur le compte nominal.

## 8.4 Build

- [ ] Identité **Apple Distribution** présente :
      `security find-identity -v -p codesigning`.
- [ ] `DEVELOPMENT_TEAM` commité dans le projet Xcode.
- [ ] Numéro de build **incrémenté**.
- [ ] Fichier `.ipa` **vérifié sur le disque** — le code de sortie ne prouve rien.
- [ ] Version embarquée confirmée dans l'`Info.plist` de l'archive si un doute
      subsiste.

## 8.5 Dans l'app

- [ ] CGU et politique de confidentialité atteignables en peu de gestes depuis
      l'écran principal.
- [ ] Les deux écrans affichent du contenu réel, y compris si le backend ne sert
      pas encore les textes.
