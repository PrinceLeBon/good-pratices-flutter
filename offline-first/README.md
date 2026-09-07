# Standards Offline-First — Flutter/Dart

Bonnes pratiques pour les couches **données** (providers remote/local) et
**Repository** d'une app **offline-first**, indépendantes du state management.
À appliquer sur tous mes projets Flutter.

## Sommaire

1. [Architecture en couches](01-architecture-couches.md)
2. [Réseau, erreurs & `Failure`](02-reseau-et-erreurs.md) — `ApiHttp`, `HttpException`, `isNetworkError`, taxonomie `Failure`
3. [Lecture offline-first](03-lecture-offline-first.md) — `readOnlineFirst`
4. [Écriture offline-first](04-ecriture-offline-first.md) — effet local optimiste
5. [Conventions](05-conventions.md) — `Either`, commentaires, DRY, tests
6. [File de synchronisation](06-file-de-sync.md) — `SyncXxx`, replay, réconciliation
7. [Service de connectivité](07-connectivite.md) — interface vs sonde serveur
8. [État agnostique](08-etat-agnostique.md) — un contrat, N state managers (Cubit/GetX/Riverpod…)
9. [Cache local Hive](09-cache-hive.md) — `LocalProvider`, `Box<X>` typée

## Le modèle en une image

```
UI
 │
State (Cubit / GetX / Riverpod / ChangeNotifier)   ← adaptateur mince
 │            consomme  Either<Failure, T>
Repository                                          ← orchestre online/offline
 ├── XxxApiProvider    (remote : HTTP via ApiHttp)
 └── XxxLocalProvider  (local  : Box<X> Hive)
```

## Les invariants non négociables

1. **Tout HTTP via un seul client avec timeout** (`ApiHttp`). Jamais de `http` brut.
2. **Une seule `HttpException` typée** (statusCode). Providers la lèvent sur non-2xx.
3. **Une seule `isNetworkError`**. Jamais de copie ni de `.contains('SocketException')`.
4. **Données = 2 providers** : `ApiProvider` (remote) + `LocalProvider` (local).
   Le Repository orchestre ; il ne touche jamais la box directement.
5. **Lecture = `readOnlineFirst`** : réseau/5xx → cache ; 4xx → `Failure`.
6. **Écriture offline-first** : réseau → effet local + file de sync ; 4xx/5xx → `Failure`.
7. **Contrat Repository = `Either<Failure, T>`** — agnostique du state management.

## Checklist (nouveau module)

- [ ] `data/models/`, `data/providers/`, `data/repositories/`.
- [ ] `XxxApiProvider` : appels via `ApiHttp`, lève `HttpException` sur non-2xx.
- [ ] `XxxLocalProvider` : `Box<X>` typée, ops granulaires (`put`/`putAll`/`get`/`delete`).
- [ ] Repository (lecture) : `readOnlineFirst`, renvoie `Either<Failure, T>`.
- [ ] Repository (écriture) : online-first + file de sync sur erreur réseau.
- [ ] State layer : `fold` du `Either<Failure, T>` → états (aucun string-matching).
- [ ] Un test du helper couvre la politique de lecture.
