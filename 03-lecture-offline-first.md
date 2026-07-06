# 3. Lecture offline-first — `readOnlineFirst`

Toute lecture de liste/objet passe par **un seul helper** qui centralise la
politique. Objectif : **une liste ne se vide jamais** à cause d'un timeout.

## 3.1 Politique (table de décision)

| Situation | Retour |
|---|---|
| En ligne + succès | données **serveur** (+ mise en cache) |
| En ligne mais **réseau KO** (timeout/socket) | **cache local** |
| En ligne mais **serveur en panne (5xx)** | **cache local** |
| Hors ligne | **cache local** |
| **Erreur client (4xx)** ex. 401 session expirée | **erreur remontée** (`Left`) |

Pourquoi 5xx → local mais 4xx → erreur : une panne serveur est transitoire, on
préfère servir le cache ; une erreur client (401/400) doit être **vue** pour que
l'app réagisse (reconnexion, correction).

## 3.2 Le helper

```dart
import 'dart:async';
import 'package:dartz/dartz.dart';

bool _serveLocalOnFailure(Object e) => isNetworkError(e) || isServerError(e);

/// online-first ; réseau KO ou 5xx → cache local ; 4xx → Left.
Future<Either<String, T>> readOnlineFirst<T>({
  required ConnectivityService connectivity,
  required FutureOr<T> Function() readLocal,
  required Future<T> Function() fetchRemote,
  Future<void> Function(T remote)? cache,
  T Function(T value)? postProcess, // ex. tri, appliqué local ET distant
}) async {
  T finalize(T v) => postProcess != null ? postProcess(v) : v;
  try {
    if (await connectivity.checkConnection()) {
      try {
        final remote = await fetchRemote();
        if (cache != null) await cache(remote);
        return Right(finalize(remote));
      } catch (e) {
        if (_serveLocalOnFailure(e)) return Right(finalize(await readLocal()));
        rethrow; // 4xx -> Left via le catch externe
      }
    }
    return Right(finalize(await readLocal()));
  } catch (e, st) {
    return Left('read failed: $e, stackTrace: $st');
  }
}
```

## 3.3 Utilisation dans un Repository

```dart
Future<Either<String, List<Aliment>>> getAliments(String type) {
  return readOnlineFirst<List<Aliment>>(
    connectivity: connectivityService,
    readLocal: () => GetRequest.getAllAliment(type),
    fetchRemote: () => apiProvider.getAliments(
        GetRequest.getFarmId(), GetRequest.getUserToken(), type),
    cache: (list) => InsertRequest.insertAllAliments(list, type),
    postProcess: (list) { _sortByRecency(list); return list; },
  );
}
```

Cas particuliers :
- **Provider qui renvoie `Either`** : le déplier dans `fetchRemote`
  (`result.fold((e) => throw ..., (v) => v)`) — ou mieux, faire lever le provider.
- **Effet de bord intriqué** (ex. re-lecture locale filtrée après cache, ou
  écriture multi-cache) : le mettre dans `fetchRemote` et laisser `cache: null`.

## 3.4 Testabilité (un test pour tous les repos)

Le helper est **pur** → un seul test couvre la politique de **tous** les repos
qui l'utilisent. Fournir un faux `ConnectivityService` (override
`checkConnection`).

```dart
test('online + erreur réseau → repli local', () async {
  final r = await readOnlineFirst<List<int>>(
    connectivity: _FakeConnectivity(true),
    readLocal: () => [9, 9],
    fetchRemote: () async => throw Exception('TimeoutException'),
  );
  expect(r.getOrElse(() => []), [9, 9]);
});

test('online + 5xx → repli local', () async {
  final r = await readOnlineFirst<List<int>>(
    connectivity: _FakeConnectivity(true),
    readLocal: () => [5],
    fetchRemote: () async => throw HttpException(statusCode: 503, message: 'down'),
  );
  expect(r.getOrElse(() => []), [5]);
});

test('online + 4xx (401) → Left', () async {
  final r = await readOnlineFirst<List<int>>(
    connectivity: _FakeConnectivity(true),
    readLocal: () => [9],
    fetchRemote: () async => throw HttpException(statusCode: 401, message: 'unauth'),
  );
  expect(r.isLeft(), isTrue);
});
```

> Comparer le **contenu** (`getOrElse`) et non l'objet `Right` entier : l'égalité
> de listes se fait par identité, pas par contenu.
