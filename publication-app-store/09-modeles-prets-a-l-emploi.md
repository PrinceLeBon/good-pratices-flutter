# 9. Modèles prêts à l'emploi

À copier, puis adapter. Chaque modèle indique **quand l'utiliser** : hors de ce
cadre, il produit une déclaration fausse — et un rejet.

Les textes destinés au reviewer sont en anglais, sa langue de travail.

## 9.1 App Review Notes — services aux entreprises (modèle C)

**Quand l'utiliser :** l'app est vendue uniquement à des organisations, et aucun
particulier ne peut s'abonner seul (cf. [01 §1.3](01-modele-de-monetisation.md)).

```
[APP NAME] is a business tool sold exclusively to organizations
([customer type, e.g. car dealerships and fleet operators]) for use by their
staff. It is not sold to individual consumers.

In accordance with guideline 3.1.3(c) (Enterprise Services), the app offers no
in-app purchase and displays no price, purchase button or link to any external
purchase method. Access is granted through a subscription contracted directly
between [PUBLISHER] and the customer organization; the app only reads that
organization's subscription status.

Demo accounts
- Active subscription, full access: [login] / [password]
- Expired subscription, blocked state: [login] / [password]

The Terms of Use (EULA) and the Privacy Policy are available in the app from
the [Profile] tab, and online:
- [SITE]/cgu.html
- [SITE]/confidentialite.html
```

## 9.2 App Review Notes — app compagnon gratuite (modèle E)

**Quand l'utiliser :** l'app est gratuite, le service se souscrit sur le web, et
l'app ne contient ni achat ni incitation à acheter ailleurs.

```
[APP NAME] is a free companion app for [SERVICE], a paid web-based service.

In accordance with guideline 3.1.3(f) (Free Stand-alone Apps), the app contains
no purchasing and no call to action for purchase outside of the app. Existing
customers sign in with the account they created on [SERVICE].

Demo account: [login] / [password]

Terms of Use and Privacy Policy: [SITE]/cgu.html, [SITE]/confidentialite.html
```

## 9.3 Réponse au reviewer

**Quand l'utiliser :** après un rejet, une fois l'app **et** les métadonnées
corrigées (cf. [07](07-revue-et-reponses.md)). Un bloc par guideline, dans l'ordre
du courriel d'Apple ; supprime ceux qui ne te concernent pas.

```
Hello,

Thank you for your review. The points below have been addressed.

--- Guideline 2.1 ---
The demo accounts are provided in the App Review Information section:
- [State, e.g. expired subscription]: [login] / [password]
- [State, e.g. active subscription]: [login] / [password]

--- Guideline 3.1.2(c) ---
No auto-renewable subscription is included in this submission. [APP NAME] is
[a business tool sold to organizations / a free companion app], covered by
guideline [3.1.3(c) / 3.1.3(f)]. The app displays no price, no subscription
duration and no purchase button.
The Terms of Use (EULA) are available in App Information and at
[SITE]/cgu.html. The Privacy Policy URL is set in App Store Connect.

--- Guideline 1.5 ---
The Support URL now points to [SITE]/support.html, a support page with our
email address, phone number and answers to common questions.

A screen recording is attached, showing [sequence, see 9.9].

Thank you for your time.
```

## 9.4 Textes d'interface iOS

**Quand l'utiliser :** modèles C et E. Une seule source, que référencent tous les
écrans, popups et notifications.

```dart
import 'dart:io' show Platform;

/// Textes liés à l'abonnement. Aucun écran ne les recopie.
/// Sur iOS, ils constatent sans jamais orienter vers un achat.
class AbonnementTextes {
  AbonnementTextes._();

  static const String indisponibleSurIos =
      "La gestion de l'abonnement ne s'effectue pas depuis l'application iOS. "
      "Si votre organisation dispose déjà d'un abonnement, il est "
      "automatiquement pris en compte ici.";

  static String get expireMessage => Platform.isIOS
      ? "Votre abonnement est expiré. Contactez le support pour rétablir l'accès."
      : "Votre abonnement est expiré. Renouvelez-le depuis [écran de paiement].";

  static String get expireBouton =>
      Platform.isIOS ? 'Contacter le support' : 'Renouveler';

  /// Rappel avant échéance, en notification locale ou push.
  static String rappelMessage(int joursRestants) => Platform.isIOS
      ? "Votre abonnement expire dans $joursRestants jour(s)."
      : "Votre abonnement expire dans $joursRestants jour(s), renouvelez-le.";
}
```

En modèle E, remplace « votre organisation » par le nom du service web.

## 9.5 Encart à la place du bouton d'achat

**Quand l'utiliser :** modèles C et E, sur l'écran d'abonnement ou de
renouvellement.

```dart
class AbonnementIndisponibleNotice extends StatelessWidget {
  const AbonnementIndisponibleNotice({
    super.key,
    required this.onContactSupport,
  });

  /// Ouvre l'écran qui porte les coordonnées du support.
  final VoidCallback onContactSupport;

  @override
  Widget build(BuildContext context) {
    final scheme = Theme.of(context).colorScheme;
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: scheme.surfaceContainerHighest,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(Icons.info_outline, color: scheme.primary),
          const SizedBox(height: 10),
          const Text(
            AbonnementTextes.indisponibleSurIos,
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 6),
          TextButton(
            onPressed: onContactSupport,
            child: const Text('Contacter le support'),
          ),
        ],
      ),
    );
  }
}
```

## 9.6 Écran d'abonnement : la garde de plateforme

**Quand l'utiliser :** modèles C et E. Le drapeau couvre aussi l'ouverture
directe de l'écran en mode renouvellement (cf. [02 §2.3](02-interface-ios-sans-achat.md)).

```dart
// Ni tarif ni choix de durée sur iOS.
final bool afficherTarif = enRenouvellement && !Platform.isIOS;

// Dans la liste des enfants de l'écran :
EncartAvantages(afficherTarif: afficherTarif),
if (Platform.isIOS)
  AbonnementIndisponibleNotice(onContactSupport: ouvrirSupport)
else
  BoutonPaiement(
    onPressed: () {
      if (Platform.isIOS) return; // garde de sécurité, cf. 02 §2.6
      lancerPaiement();
    },
  ),
```

## 9.7 Annexe Apple d'un EULA personnalisé

**Quand l'utiliser :** dès que tu déposes tes propres CGU dans *App Information →
License Agreement* (cf. [04 §4.3](04-metadonnees-et-pages-publiques.md)). À placer
en fin de CGU ; remplace `[ÉDITEUR]` et `[CONTACT]`.

```markdown
## Annexe — Dispositions applicables aux téléchargements depuis l'App Store

1. **Parties au contrat.** Les présentes CGU sont conclues entre l'Utilisateur
   et [ÉDITEUR] seul, à l'exclusion d'Apple. [ÉDITEUR] est seul responsable de
   l'Application et de son contenu.
2. **Étendue de la licence.** La licence concédée est une licence personnelle,
   non transférable, d'utilisation de l'Application sur tout appareil de marque
   Apple que l'Utilisateur possède ou contrôle, dans les limites des Règles
   d'utilisation énoncées dans les Conditions des Services Média d'Apple.
3. **Maintenance et assistance.** [ÉDITEUR] est seul tenu de fournir la
   maintenance et l'assistance relatives à l'Application. Apple n'est soumise à
   aucune obligation à ce titre.
4. **Garantie.** [ÉDITEUR] assume seul toute garantie éventuelle. En cas de
   non-conformité à une garantie applicable, l'Utilisateur peut en informer
   Apple, qui pourra lui rembourser le prix d'achat éventuel de l'Application.
   Dans la limite permise par la loi, Apple n'assume aucune autre obligation de
   garantie.
5. **Réclamations.** [ÉDITEUR], et non Apple, traite les réclamations relatives
   à l'Application, notamment en matière de responsabilité du fait des produits,
   de conformité légale ou réglementaire, et de protection des consommateurs ou
   des données personnelles.
6. **Propriété intellectuelle.** En cas de réclamation d'un tiers alléguant une
   atteinte à ses droits de propriété intellectuelle, [ÉDITEUR], et non Apple,
   en assume seul la défense et les conséquences.
7. **Conformité légale.** L'Utilisateur déclare ne pas être situé dans un pays
   soumis à un embargo du gouvernement des États-Unis ou désigné par celui-ci
   comme soutenant le terrorisme, et ne pas figurer sur une liste de parties
   interdites ou soumises à restrictions.
8. **Coordonnées.** Toute question ou réclamation relative à l'Application peut
   être adressée à [ÉDITEUR] : [CONTACT].
9. **Conditions des tiers.** L'Utilisateur respecte les conditions des services
   tiers auxquels l'Application recourt.
10. **Bénéficiaire tiers.** Apple et ses filiales sont tiers bénéficiaires des
    présentes CGU et peuvent en poursuivre l'exécution à l'encontre de
    l'Utilisateur.
```

Texte complet de référence, pour un outil B2B avec paiement externe, facturation
normalisée et mode hors ligne : `CGU_FLASHCAR.md`, dans le dépôt FlashCar. Toute
adaptation reste à faire relire par un juriste.

## 9.8 Site public

**Quand l'utiliser :** toute app, dès la première soumission
(cf. [04 §4.2](04-metadonnees-et-pages-publiques.md)).

Implémentation de référence dans le dépôt FlashCar
(`waouhmonde229/app-mobile_flashcar-mobile`) :

```
website/                  à déposer tel quel sur Netlify
  index.html              accueil
  support.html            → Support URL
  confidentialite.html    → Privacy Policy URL
  cgu.html                → EULA
  style.css
website_src/
  _build.py               convertisseur Markdown → HTML
  _confidentialite.md     source de confidentialite.html
  README.md               commandes de régénération
```

Pour une nouvelle app : copie les deux dossiers, puis remplace le nom, les
couleurs de `style.css`, les coordonnées de `support.html` et les deux sources
Markdown, et régénère. Corrige toujours les sources, jamais le HTML généré.

## 9.9 Capture vidéo pour le reviewer

**Quand l'utiliser :** chaque fois qu'Apple la demande, et par défaut après un
rejet `3.1.x`.

1. Lancer l'app sur un appareil sans session ouverte.
2. Se connecter avec **le compte demandé par le reviewer**.
3. Compte actif : parcourir les fonctionnalités principales.
4. Ouvrir l'écran d'abonnement : ni prix, ni bouton d'achat, encart visible.
5. Déclencher les popups liés à l'abonnement, sans les esquiver.
6. Profil → CGU, puis Profil → politique de confidentialité, pages ouvertes.

Un seul plan continu, sans montage.
