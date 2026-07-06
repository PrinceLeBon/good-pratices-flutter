# 6. File de synchronisation (replay & réconciliation)

Quand une écriture se fait hors ligne (ou sur erreur réseau), on applique
l'effet **localement** et on enfile une **opération de sync** qui sera **rejouée**
à la reconnexion. C'est ce qui rend l'app utilisable sans réseau, sans perte.

## 6.1 Modèles d'opération — `SyncXxx`

Une op de sync décrit **quoi rejouer**. Un modèle par entité, persisté (Hive).

```dart
class SyncAliment {
  final String id;          // id de l'op (uuid)
  final String type;        // 'create' | 'update' | 'delete'
  final Aliment aliment;    // payload complet à rejouer
  final DateTime timestamp; // ordre de rejeu (FIFO)
  bool synced;              // déjà rejoué ?
}
```

Toutes les ops vivent dans **une box `sync_queue`** (ordre FIFO par timestamp).

## 6.2 Enfiler (côté Repository, branche offline)

```dart
final op = SyncAliment(
  id: const Uuid().v8(),
  type: 'create',
  aliment: newAliment,       // le modèle avec son id LOCAL
  timestamp: DateTime.now(),
  synced: false,
);
await syncService.addToSyncQueue(type: 'create', syncOperation: op);
```

## 6.3 Rejeu — `SyncService.performSync()`

À la reconnexion, on rejoue la file **dans l'ordre** :

```
pour chaque op non synchronisée (FIFO) :
   rejouer via le Repository, mais SANS ré-appliquer l'effet local
   (applyLocalEffects: false) — l'effet a déjà été appliqué offline.
   si succès réseau : marquer synced = true, réconcilier les ids
   si erreur réseau : arrêter (laisser pending, on réessaiera)
   si erreur métier : marquer en échec / logguer (op invalide)
```

Points cruciaux :

- **`applyLocalEffects: false` au rejeu** : sinon l'effet est appliqué **deux
  fois** (une fois offline, une fois au replay). Le repo doit accepter un flag
  `isReplay`/`applyLocalEffects` pour sauter l'effet local pendant la sync.
- **`isReplay: true`** : ne pas ré-enfiler l'op (elle est déjà dans la file) et
  ne pas recréer offline si toujours hors ligne → renvoyer `Left` « pending ».
- **Ordre FIFO** : les ops dépendantes (créer un lot puis une vente sur ce lot)
  doivent se rejouer dans l'ordre de création.

## 6.4 Réconciliation des ids local ↔ serveur

Offline, on a généré un **id local (uuid)**. Au replay, le serveur renvoie son
**id définitif**. Deux stratégies :

1. **Mapping d'ids** (`IdMappingService`, box `id_mappings`) : on stocke
   `idLocal → idServeur` et on remplace les références. Nécessaire quand d'autres
   entités référencent cet id.
2. **Miroir complet** : pour les familles « id généré serveur sans référence
   externe » (ex. réajustements), on **remplace la liste locale** par la liste
   serveur au refresh — pas besoin de mapping.

Astuce **forward de l'id local** : si le backend accepte de créer avec l'id
fourni, envoyer l'id local → `idLocal == idServeur`, plus de réconciliation à
faire (utilisé pour les référentiels comme les catégories).

## 6.5 Hydratation à la reconnexion / au login

Après un `performSync()` réussi, **réconcilier l'état** en re-tirant les données
serveur (les lectures passent par [`readOnlineFirst`](03-lecture-offline-first.md)).
Un `HydrationService` orchestre le pull de tous les référentiels au login /
retour réseau.

## 6.6 Idempotence (indispensable)

Le rejeu peut arriver **plusieurs fois** (crash, double reconnexion). Donc :

- Effets en **valeur absolue** quand possible (`stock = X` plutôt que `+= n`).
- Le backend doit tolérer un rejeu (id fourni → upsert, ou détection de doublon).
- Une op déjà `synced` n'est jamais rejouée.

## 6.7 Pièges connus

- **Double effet** : oublier `applyLocalEffects:false` au replay.
- **Doublon** : recréer offline pendant un replay (oublier `isReplay`).
- **Op fantôme** : enfiler une op qui échouera côté serveur (validation offline
  absente) → elle « disparaît » au refresh. Valider offline comme le backend.
- **createdAt à minuit** : un item créé offline avec `createdAt` = minuit passe
  sous les items serveur du même jour dans un tri par `createdAt`. Utiliser un
  timestamp complet en UTC à la création offline.
