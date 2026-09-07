# 7. Revue et réponses au reviewer

## 7.1 La règle qui commande tout

**Chaque nouveau binaire repart au fond de la file d'attente.**

Conséquence : quand le reviewer demande une **information** — compte de démo,
capture vidéo, métadonnée — on **répond**, on ne rebuilde pas. Uploader un
nouveau build dans ce cas ne corrige rien et rallonge le délai d'autant.

Un correctif de code identifié pendant un cycle de revue attend le prochain
build. Il se commite, il ne se soumet pas.

## 7.2 Structure d'une réponse

Une réponse efficace est **factuelle, courte, et organisée par numéro de
guideline**, dans l'ordre où Apple les a listés.

```
--- Guideline 2.1 ---
<ce qui a été fait, identifiants, où les trouver>

--- Guideline 3.1.2(c) ---
<ce qui a été fait, URL concernées>
```

- Un paragraphe par point, pas de plaidoirie.
- Décrire l'**état actuel** de l'app et des métadonnées, pas l'intention.
- Ne jamais contester la lecture du reviewer : expliquer ce qui a changé.
- Terminer en signalant la capture vidéo jointe.

## 7.3 La capture vidéo

Souvent réclamée, parfois de façon explicite (« reply with a screen recording »).

- Filme **exactement l'état dont il parle**, avec le compte qu'il a demandé.
- Un plan continu, sans montage.
- **N'esquive pas les écrans gênants** : la séquence complète raconte une
  histoire cohérente, une séquence tronquée éveille les soupçons.
- Termine par les écrans de CGU et de confidentialité, ouverts depuis l'app.

## 7.4 Ce qu'un rejet t'apprend vraiment

Un rejet répété sur la **même** guideline après correction signifie presque
toujours que le problème est ailleurs que là où tu l'as cherché :

- `3.1.2(c)` qui revient → un produit d'abonnement est encore attaché à la
  soumission (cf. [03](03-produits-et-abonnements.md)) ;
- `2.1` qui revient → le compte fourni ne montre pas l'état que le reviewer veut
  voir (cf. [05](05-comptes-de-demonstration.md)) ;
- une guideline qui **disparaît** de la liste est réglée : Apple ne répète pas
  les points résolus. C'est le seul accusé de réception que tu obtiendras.

## 7.5 Lire l'en-tête du courriel

Le bloc *Review Environment* indique l'appareil et la version relus. Deux usages :

- **l'appareil** (souvent un iPad) dit où reproduire le problème ;
- **la version et le build** confirment que c'est bien ton dernier envoi qui a
  été examiné — sinon, inutile de chercher un bug dans du code qui n'y était pas.

## 7.6 Après validation

Reporte immédiatement dans les **App Review Notes** ce qui a débloqué la
situation. Le prochain reviewer n'aura pas l'historique de la conversation, et
c'est ce champ qui évite de refaire le tour.
