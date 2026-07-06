# Standards Offline-First — Flutter/Dart

Bonnes pratiques pour les couches **Provider** (réseau) et **Repository**
(données) d'une app **offline-first**. À appliquer sur tous mes projets Flutter.

Tirées de décisions réelles : centralisation du réseau, repli local fiable,
détection d'erreurs unifiée, testabilité.

## Sommaire

1. [Architecture en couches](01-architecture-couches.md)
2. [Réseau & erreurs](02-reseau-et-erreurs.md) — `ApiHttp`, `HttpException`, `isNetworkError`
3. [Lecture offline-first](03-lecture-offline-first.md) — `readOnlineFirst`
4. [Écriture offline-first](04-ecriture-offline-first.md) — effet local optimiste
5. [Conventions](05-conventions.md) — `Either`, commentaires, DRY, tests
6. [File de synchronisation](06-file-de-sync.md) — `SyncXxx`, replay, réconciliation d'ids
7. [Service de connectivité](07-connectivite.md) — interface vs sonde serveur

## Les 5 invariants non négociables

1. **Tout HTTP passe par un seul client avec timeout** (`ApiHttp`). Jamais de
   `http` brut (sinon l'UI gèle quand le serveur est injoignable).
2. **Une seule `HttpException` typée** (avec `statusCode`). Les providers la
   lèvent sur non-2xx. Jamais de `throw Exception('...')` générique pour une
   erreur HTTP, jamais de classe dupliquée.
3. **Une seule fonction `isNetworkError`**. Jamais de copie locale ni de bloc
   `.contains('SocketException')` inline.
4. **Lecture = `readOnlineFirst`** : réseau KO ou 5xx → cache local ; 4xx →
   erreur remontée. Une liste ne se vide jamais à cause d'un timeout.
5. **Écriture offline-first** : erreur réseau → effet local + file de sync ;
   erreur métier → erreur remontée.

## Checklist (nouveau repo/provider)

- [ ] Provider : tous les appels via `ApiHttp` (timeout).
- [ ] Provider : lève `HttpException(statusCode, message, body)` sur non-2xx.
- [ ] Provider : `on HttpException { rethrow }` si un `catch` générique enveloppe.
- [ ] Repository (lecture) : passe par `readOnlineFirst`.
- [ ] Repository (écriture) : online-first + file de sync sur erreur réseau.
- [ ] Détection réseau : `isNetworkError` partagé, jamais recopié.
- [ ] Retour Repository : `Either<String, T>`.
- [ ] Commentaires : « pourquoi » seulement.
- [ ] Un test du helper couvre la politique de lecture.
