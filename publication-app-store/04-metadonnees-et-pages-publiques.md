# 4. Métadonnées et pages publiques

Relues à **chaque** soumission, réglables en dix minutes, et responsables de
deux de nos rejets. Autant les avoir justes dès le premier envoi.

## 4.1 Les champs vérifiés

| Champ | Ce qu'Apple attend | Piège |
|---|---|---|
| **Privacy Policy URL** | Page publique décrivant réellement les traitements | Obligatoire sans exception, même pour un outil interne |
| **Support URL** | Page où l'utilisateur peut poser une question | Un README de dépôt Git a valu un rejet `1.5` |
| **EULA** | Lien Apple standard dans la description, ou texte personnalisé dans *App Information* | Un EULA personnalisé doit porter les mentions minimales Apple |
| **App Review Notes** | Contexte métier en trois lignes | Réclamé « for future submissions » — et redemandé tant qu'il est vide |
| **Demo account** | Identifiants fonctionnels, accès complet | Cf. [05](05-comptes-de-demonstration.md) |

## 4.2 Le site public, une fois pour toutes

Monte un **site statique de quatre pages** dès le premier projet :

```
index.html            accueil, liens vers les trois autres
support.html          → Support URL
confidentialite.html  → Privacy Policy URL
cgu.html              → EULA (ou lien depuis la description)
```

Hébergement gratuit, aucune dépendance, et **il resservira à chaque app**. Trois
champs obligatoires couverts d'un coup.

La page de support doit permettre de **poser une question** : adresse
électronique, téléphone, et de préférence quelques réponses aux problèmes
courants. Une page qui se contente de décrire le produit ne passe pas.

## 4.3 EULA : standard ou personnalisé

- **Standard** — colle le lien des conditions Apple dans la description de
  l'app. Le plus rapide, suffisant dans la plupart des cas.
- **Personnalisé** — dépose ton texte dans *App Information → License
  Agreement*. Obligatoire dès que ton service a ses propres règles
  (responsabilité, données, obligations réglementaires).

Un EULA personnalisé **doit reprendre les mentions minimales exigées par Apple** :
contrat conclu avec l'éditeur seul et non avec Apple, licence limitée aux
appareils Apple, maintenance à la charge de l'éditeur, remboursement possible
par Apple en cas de non-conformité, responsabilité des réclamations et de la
propriété intellectuelle du côté de l'éditeur, conformité aux lois d'exportation,
coordonnées de contact, et **Apple tiers bénéficiaire du contrat**.

C'est précisément ce que le reviewer contrôle sur un EULA personnalisé.

## 4.4 Accès depuis l'app

Indépendamment des métadonnées, rends les **CGU et la politique de
confidentialité atteignables depuis l'app elle-même**, en peu de gestes depuis
l'écran principal. Deux niveaux d'enfouissement, c'est déjà trop : le reviewer
les cherche, et un utilisateur aussi.

Alimente ces écrans depuis le backend (un champ de configuration) plutôt qu'en
dur, avec un **texte de repli réellement exploitable** — pas un « contenu
provisoire ». Si l'API n'a pas encore le champ le jour de la revue, c'est le
repli que le reviewer lit.

## 4.5 Ordre des opérations

Renseigne les métadonnées **avant** de répondre au reviewer : il les consulte au
moment où il lit ton message. Une réponse qui annonce une URL pas encore en
ligne produit un nouveau rejet.
