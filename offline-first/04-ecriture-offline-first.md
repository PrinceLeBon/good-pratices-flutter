# 4. Écriture offline-first

Une écriture doit marcher **online ET offline**, avec le **même résultat visible**,
et se synchroniser sans doublon ni perte. Le Repository orchestre `remote` +
`local` + la file de sync.

## 4.1 Politique

| Situation | Comportement | Retour |
|---|---|---|
| En ligne + succès | `remote.createX` → `local.save` | `Right(x)` |
| En ligne, **erreur réseau** | effet local optimiste + **file de sync** | `Right(x)` |
| En ligne, **4xx/5xx** | rien (pas de fallback) | `Left(Failure)` |
| Hors ligne | effet local optimiste + file de sync | `Right(x)` |

## 4.2 Squelette

```dart
Future<Either<Failure, X>> createX(X input, {bool isReplay = false}) async {
  try {
    if (await connectivity.checkConnection()) {
      try {
        final created = await remote.createX(input); // lève HttpException sur non-2xx
        await local.save(created);
        return Right(created);
      } catch (e) {
        if (isNetworkError(e)) {
          if (isReplay) return Left(const NetworkFailure()); // replay : laisser pending
          return _createXOffline(input);
        }
        return Left(toFailure(e)); // 4xx/5xx : erreur métier, pas de fallback
      }
    }
    if (isReplay) return Left(const NetworkFailure());
    return _createXOffline(input);
  } catch (e) {
    return Left(toFailure(e));
  }
}

Future<Either<Failure, X>> _createXOffline(X input) async {
  // 1) Valider comme le backend (sinon on enfile une op qui sera rejetée).
  // 2) Générer un id local (uuid).
  // 3) Effet local optimiste : local.save(x) (+ effets liés, ex. décrément stock).
  // 4) Enfiler une op de sync (SyncXxx) — cf. 06.
}
```

## 4.3 Règles clés

- **`isReplay`** : au rejeu de la file, l'op existe déjà → ne pas la ré-enfiler
  ni recréer offline (sinon doublon). Renvoyer un `NetworkFailure` pour la laisser
  « pending » si toujours hors ligne.
- **Validation offline = validation backend** : rejouer les mêmes règles (stock,
  champ requis…) → ne pas enfiler une op qui échouera à la sync et
  « disparaîtra » au refresh.
- **Effets idempotents / valeurs absolues** (`stock = X` plutôt que `+= n`) → le
  rejeu multiple donne le même résultat.
- **Parité** : l'effet local optimiste produit le **même état** que le retour
  serveur.
