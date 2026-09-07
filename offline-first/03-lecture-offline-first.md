# 3. Lecture offline-first — `readOnlineFirst`

Toute lecture passe par **un seul helper** qui orchestre `remote` + `local`.
Objectif : **une liste ne se vide jamais** à cause d'un timeout.

## 3.1 Politique (table de décision)

| Situation | Retour |
|---|---|
| En ligne + succès | données **serveur** (+ `local.save`) |
| Réseau KO (timeout/socket) | **cache local** (`Right`) |
| Serveur en panne (5xx) | **cache local** (`Right`) |
| Hors ligne | **cache local** (`Right`) |
| Erreur client 4xx (ex. 401) | **`Left(Failure)`** |

Pourquoi 5xx → cache mais 4xx → `Failure` : une panne serveur est transitoire ;
une erreur client (401/400) doit remonter pour que l'app réagisse (reconnexion).

## 3.2 Le helper

```dart
import 'dart:async';
import 'package:dartz/dartz.dart';

bool _serveLocalOnFailure(Object e) =>
    isNetworkError(e) || (e is HttpException && e.isServerError);

/// online-first ; réseau/5xx → cache local ; 4xx → Left(Failure).
Future<Either<Failure, T>> readOnlineFirst<T>({
  required ConnectivityService connectivity,
  required FutureOr<T> Function() readLocal,   // ex. local.getAll()
  required Future<T> Function() fetchRemote,    // ex. remote.getAll()
  Future<void> Function(T remote)? cache,       // ex. local.save(data)
  T Function(T value)? postProcess,             // ex. tri, appliqué local ET distant
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
        return Left(toFailure(e)); // 4xx -> Failure typé (cf. 02)
      }
    }
    return Right(finalize(await readLocal()));
  } catch (e) {
    return Left(toFailure(e));
  }
}
```

## 3.3 Usage dans un Repository (remote + local)

```dart
class AlimentRepository {
  final AlimentApiProvider remote;
  final AlimentLocalProvider local;
  final ConnectivityService connectivity;
  AlimentRepository(this.remote, this.local, this.connectivity);

  Future<Either<Failure, List<Aliment>>> getAliments() {
    return readOnlineFirst<List<Aliment>>(
      connectivity: connectivity,
      readLocal: () => local.getAll(),
      fetchRemote: () => remote.getAliments(),
      cache: (list) => local.save(list),
    );
  }
}
```

> C'est la version factorisée + affinée du classique `try { api } catch { cache }`.
> Le `try/catch` maison sert le cache **même sur une vraie erreur** (ex. 401) ;
> `readOnlineFirst` distingue réseau/5xx (→ cache) de 4xx (→ `Failure`).

## 3.4 Testabilité (un test pour tous les repos)

Helper **pur** → un seul test couvre la politique de **tous** les repos. Faux
`ConnectivityService` (override `checkConnection`).

```dart
test('online + erreur réseau → cache local', () async {
  final r = await readOnlineFirst<List<int>>(
    connectivity: _FakeConnectivity(true),
    readLocal: () => [9, 9],
    fetchRemote: () async => throw Exception('TimeoutException'),
  );
  expect(r.getOrElse(() => []), [9, 9]);
});

test('online + 5xx → cache local', () async {
  final r = await readOnlineFirst<List<int>>(
    connectivity: _FakeConnectivity(true),
    readLocal: () => [5],
    fetchRemote: () async => throw HttpException(statusCode: 503, message: 'down'),
  );
  expect(r.getOrElse(() => []), [5]);
});

test('online + 401 → Left(AuthFailure)', () async {
  final r = await readOnlineFirst<List<int>>(
    connectivity: _FakeConnectivity(true),
    readLocal: () => [9],
    fetchRemote: () async => throw HttpException(statusCode: 401, message: 'x'),
  );
  expect(r.isLeft(), isTrue);
  r.fold((f) => expect(f, isA<AuthFailure>()), (_) {});
});
```

> Comparer le **contenu** (`getOrElse`) et non l'objet `Right` (égalité de listes
> par identité).
