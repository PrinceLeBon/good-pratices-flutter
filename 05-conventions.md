# 5. Conventions

## 5.1 `Either` pour les retours Repository

- `Either<String, T>` (dartz) : `Left` = message d'erreur, `Right` = succès.
- Le Provider **lève** ; le Repository **transforme** en `Either`.
- Comparer des listes en test via le contenu (`getOrElse`) et non l'objet
  `Right` (égalité par identité).

## 5.2 Commentaires : minimal, « pourquoi » seulement

- **Garder** : le *pourquoi* non évident (repli 5xx, re-lecture locale filtrée,
  déplier un `Either`, `on HttpException { rethrow }`).
- **Supprimer** : la narration qui redit le code ou la signature
  (« Politique offline-first centralisée » au-dessus d'un appel à
  `readOnlineFirst`, « Récupérer tous les X » au-dessus de `getX`).
- Pas de code commenté laissé en place (`//log.i(...)`).

## 5.3 DRY / source unique de vérité

- La politique (lecture, détection réseau, exception HTTP) vit à **un seul
  endroit** : `readOnlineFirst`, `isNetworkError`, `HttpException`.
- Aucune copie locale : `grep _isNetworkError` / `grep "class HttpException"` /
  `grep "contains('SocketException')"` doivent pointer un seul emplacement.

## 5.4 Migration progressive (pas de big-bang)

Pour introduire ces standards dans un projet existant :

1. **Expand** — créer le helper partagé + tests, migrer 1-2 cas pilotes.
2. **Migrate** — migrer les lectures visibles (listes avec pull-to-refresh),
   par petits lots reviewables.
3. **Contract** — dédupliquer (supprimer les copies locales), harmoniser les
   providers (lever `HttpException` partout).

Chaque étape = un commit/PR isolé, `analyze` + tests verts.

## 5.5 Tests

- Les **builders/calculs purs** (agrégations, fiches) : testés sans Hive ni
  réseau.
- Le **helper de lecture** : un test couvre succès / réseau KO / 5xx / 4xx /
  offline pour **tous** les repos.
- Fournir un faux service de connectivité (override `checkConnection`), pas de
  vrai plugin en test.

## 5.6 Dates (rappel utile)

- Dates « jour » stockées en **UTC minuit** ; comparaison via une **clé de jour
  canonique** (`année*10000 + mois*100 + jour`), jamais par split de chaîne.
- Tri des listes cache par `createdAt` **parsé en DateTime** (robuste aux
  fuseaux/formats local sans `Z` vs serveur UTC), tiebreaker `id`.
