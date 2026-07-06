# 4. Écriture offline-first

Une écriture (create/update/delete) doit marcher **online ET offline**, avec le
**même résultat visible**, et se synchroniser sans doublon ni perte.

## 4.1 Politique

| Situation | Comportement |
|---|---|
| En ligne + succès | persiste le retour serveur en cache, renvoie `Right` |
| En ligne mais **erreur réseau** | effet local **optimiste** + **file de sync** (`SyncXxx`), renvoie `Right` |
| En ligne mais **erreur métier (4xx)** | `Left` (pas de fallback offline) |
| Hors ligne | effet local optimiste + file de sync, renvoie `Right` |

## 4.2 Squelette

```dart
Future<Either<String, T>> createX(X input, {bool isReplay = false}) async {
  try {
    if (await connectivity.checkConnection()) {
      try {
        final created = await apiProvider.createX(input); // lève sur non-2xx
        await cache.insert(created);
        return Right(created);
      } catch (e) {
        if (isNetworkError(e)) {
          // Au REPLAY d'une op déjà en file : ne pas re-créer offline (doublon).
          if (isReplay) return const Left('Network/Offline during replay');
          return _createXOffline(input);
        }
        return Left(e.toString()); // erreur métier -> pas de fallback
      }
    }
    if (isReplay) return const Left('Offline during replay');
    return _createXOffline(input);
  } catch (e, st) {
    return Left('createX failed: $e, $st');
  }
}

Future<Either<String, T>> _createXOffline(X input) async {
  // 1) Valider comme le backend (sinon on enfile une op qui sera rejetée).
  // 2) Générer un id local (uuid).
  // 3) Appliquer l'effet local (insert + effets de bord, ex. décrément stock).
  // 4) Enfiler une op de sync (SyncXxx) qui sera rejouée à la reconnexion.
  ...
}
```

## 4.3 Règles clés

- **`isReplay`** : quand `SyncService` rejoue la file, l'op est déjà en file →
  ne pas la ré-enfiler ni recréer offline (sinon doublon). Renvoyer `Left` pour
  la laisser « pending » si toujours hors ligne.
- **Validation offline = validation backend** : rejouer les mêmes règles (stock
  insuffisant, champ requis…) pour ne pas enfiler une op qui échouera à la sync
  et « disparaîtra » au prochain refresh serveur.
- **Effets idempotents / valeurs absolues** quand possible (ex. `stock = X`
  plutôt que `stock += n`) → rejouer plusieurs fois donne le même résultat.
- **Réconciliation des ids** local↔serveur au replay (mapping d'ids, ou miroir
  complet en remplaçant la liste).
- **Parité** : l'effet local optimiste doit produire le **même état** que le
  retour serveur (mêmes champs mis à jour).
