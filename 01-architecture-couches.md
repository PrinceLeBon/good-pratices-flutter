# 1. Architecture en couches

## Schéma

```
UI (screens / widgets)
      │
State (Cubit / Bloc / GetX / Riverpod / ChangeNotifier)   ← adaptateur
      │   consomme  Either<Failure, T>
Repository                        ← orchestre online/offline (le "chef d'atelier")
      ├─► XxxApiProvider    (remote : HTTP via ApiHttp, lève HttpException)
      └─► XxxLocalProvider  (local  : Box<X> Hive, ops granulaires)
```

Arborescence type (héritée de l'approche Clean / DataSource) :

```
lib/data/
  models/        aliment_model.dart          (+ .g.dart pour Hive)
  providers/     aliment_api_provider.dart   (remote)
                 aliment_local_provider.dart (local)
  repositories/  aliment_repository.dart
```

## Responsabilités

- **State layer** (Cubit/GetX/Riverpod…) : appelle le Repository « aveuglément »,
  `fold` le `Either<Failure, T>` en états. Ne sait **rien** de online/offline.
  → interchangeable (voir [08](08-etat-agnostique.md)).
- **Repository** : **seule couche qui décide** online/offline. Orchestre les deux
  providers, applique les effets (cache, file de sync). Renvoie
  **`Either<Failure, T>`**. Ne touche **jamais** la box directement.
- **ApiProvider (remote)** : un endpoint = une méthode. Renvoie un **modèle**,
  **lève** `HttpException` sur non-2xx (voir [02](02-reseau-et-erreurs.md)).
  Ne connaît pas le cache.
- **LocalProvider (local)** : parle à **une `Box<X>` Hive typée**. Lecture/écriture
  granulaires (`getAll`, `save`, `put`, `delete`). Renvoie des **modèles**, ne
  laisse jamais fuir de type Hive.

## Pourquoi ce découpage

- **Repository = pur orchestrateur** → la logique offline-first vit à un seul
  endroit, testable en mockant 2 sources.
- **`LocalProvider` encapsule tout le cache** → aucun accès Hive éparpillé.
- **`Either<Failure, T>`** rend le Repository indépendant du state management.

## Anti-patterns à bannir

- Un state (Cubit…) qui appelle `http` ou lit la box directement.
- Un Repository qui ouvre une `Box` Hive lui-même (→ passer par `LocalProvider`).
- Un provider qui fait les deux (réseau **et** cache).
- De la logique métier (calcul) dans un provider ou le Repository (→ builders purs).
