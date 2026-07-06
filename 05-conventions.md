# 5. Conventions

## 5.1 Contrat de retour = `Either<Failure, T>`

- `Either` (dartz) : `Left` = **`Failure` typé** (cf. [02](02-reseau-et-erreurs.md)),
  `Right` = succès.
- Le **provider lève** (`HttpException`) ; le **Repository transforme** en
  `Either<Failure, T>` (via `toFailure`).
- En test, comparer le **contenu** (`getOrElse`), pas l'objet `Right`.

## 5.2 Commentaires : minimal, « pourquoi » seulement

- **Garder** : le *pourquoi* non évident (repli 5xx, `on HttpException { rethrow }`,
  déplier un `Either`).
- **Supprimer** : la narration qui redit la signature (« Récupérer tous les X »
  au-dessus de `getX`).
- Pas de code commenté laissé en place (`//log.i(...)`).

## 5.3 DRY / source unique

La politique vit à **un seul endroit** : `readOnlineFirst`, `isNetworkError`,
`HttpException`, `Failure`. Aucune copie locale. Vérifs :

```
grep -r "_isNetworkError"            → 1 seul endroit
grep -r "class HttpException"        → 1 seul endroit
grep -r "contains('SocketException')" → aucun (utiliser isNetworkError)
```

## 5.4 Migration progressive (pas de big-bang)

1. **Expand** — créer les helpers partagés + tests, migrer 1-2 cas pilotes.
2. **Migrate** — migrer les lectures visibles (listes + pull-to-refresh), par lots.
3. **Contract** — dédupliquer, harmoniser les providers (lever `HttpException`).

Chaque étape = un commit isolé, `analyze` + tests verts.

## 5.5 Tests

- **Builders/calculs purs** (agrégations, fiches) : testés sans Hive ni réseau.
- **`readOnlineFirst`** : un test couvre succès / réseau / 5xx / 4xx / offline
  pour **tous** les repos.
- Faux `ConnectivityService` (override `checkConnection`), pas de vrai plugin.

## 5.6 Dates

- Dates « jour » en **UTC minuit** ; comparaison via une **clé de jour canonique**
  (`année*10000 + mois*100 + jour`).
- Tri des listes cache par `createdAt` **parsé en DateTime** (robuste aux fuseaux),
  tiebreaker `id`.
