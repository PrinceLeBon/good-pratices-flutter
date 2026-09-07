# 5. Comptes de démonstration

Guideline `2.1`. Le point le plus bête et le plus coûteux du processus : un
reviewer bloqué à la connexion ne regarde **rien d'autre** et rejette.

## 5.1 Les quatre règles

1. **Teste le compte toi-même sur un build release**, depuis un appareil ou un
   simulateur vierge — pas en debug avec ta session déjà ouverte. Un compte dont
   le mot de passe a changé depuis sa création est un rejet immédiat.
2. **Le compte donne accès à l'ensemble des fonctionnalités.** Un compte aux
   droits partiels fait conclure à une app incomplète.
3. **Le reviewer teste les états limites**, pas le chemin heureux. Il veut voir
   ce que vit un utilisateur bloqué.
4. **Si l'app est verrouillée par un abonnement, fournis deux comptes** — voir
   ci-dessous.

## 5.2 Deux comptes valent mieux qu'un

| Compte | Rôle | Où le mettre |
|---|---|---|
| **État nominal** | Abonnement actif, longue durée, données représentatives | App Review Notes |
| **État limite** | Abonnement expiré, essai consommé | Champs *Demo account* |

Avec un seul compte actif, le reviewer n'atteint jamais l'écran bloquant et
demande un compte expiré — un aller-retour perdu. Avec un seul compte expiré, il
ne voit aucune fonctionnalité et demande un accès complet. Fournir les deux clôt
la question.

**En modèle C, le compte expiré joue en ta faveur** : il montre, sans que tu aies
à l'argumenter, qu'aucun achat n'est possible sur iOS.

## 5.3 Préparer les données

Un compte de démo vide donne une mauvaise impression et empêche de juger les
fonctionnalités. Prévois un jeu de données minimal mais crédible : quelques
enregistrements par entité principale, sans donnée réelle de client.

## 5.4 Durée de vie

L'abonnement du compte nominal doit couvrir **largement** la période de revue et
les suivantes. Une date d'expiration à trois semaines transforme le compte en
compte expiré au milieu du cycle, et le rejet arrive sans qu'on comprenne
pourquoi.

## 5.5 Ce qui va dans les Notes

En plus des identifiants du compte nominal, trois lignes de contexte :

- ce que fait l'app et pour qui ;
- le modèle de monétisation (cf. [01](01-modele-de-monetisation.md)) ;
- où trouver les CGU et la politique de confidentialité dans l'app.

Apple demande explicitement d'y mettre ces informations « for future
submissions ». Rempli une fois, ce champ évite de refaire le tour à chaque
soumission.
