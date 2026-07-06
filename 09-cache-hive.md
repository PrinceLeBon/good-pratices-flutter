# 9. Cache local — Hive (`LocalProvider`)

Le `LocalProvider` est le **spécialiste du stockage local** : il parle à **une
`Box<X>` Hive typée** et rien d'autre. Le Repository ne touche jamais Hive
directement (il passe par ce provider).

## 9.1 Modèle annoté + adapter généré

```dart
@HiveType(typeId: 1)
class Task extends Equatable {
  @HiveField(0) final int? id;
  @HiveField(1) final String title;
  @HiveField(2) final bool isCompleted;
  const Task({this.id, required this.title, this.isCompleted = false});
  @override List<Object?> get props => [id, title, isCompleted];
}
```

- `@HiveType(typeId)` : **unique dans toute l'app**.
- `@HiveField(index)` : **unique dans la classe**.
- Générer l'adapter : `dart run build_runner build`. Enregistrer au démarrage :
  `Hive.registerAdapter(TaskAdapter())` puis ouvrir la box.

## 9.2 Boîte typée + opérations granulaires (approche recommandée, 95%)

`Box<Task>` : la box n'accepte **que** des `Task` (sécurité de types). Ops
granulaires → chaque écriture n'affecte **qu'une entrée** (performant, scalable).

```dart
class TaskLocalProvider {
  static const _boxName = 'tasksBox';
  Box<Task> get _box => Hive.box<Task>(_boxName);

  Future<void> openBox() async => Hive.openBox<Task>(_boxName);

  List<Task> getAll() => _box.values.toList();

  /// Écriture en masse (liste venue de l'API). Clé = id métier.
  Future<void> save(List<Task> tasks) async =>
      _box.putAll({for (final t in tasks) t.id!: t});

  Future<void> put(Task t) async => _box.put(t.id, t);   // create/update
  Future<void> delete(int id) async => _box.delete(id);
  Future<void> clear() async => _box.clear();
}
```

**3 façons d'écrire** : `put(key, value)` (clé métier, la plus sûre — à
privilégier), `putAll(map)` (masse, ex. cache d'une liste API), `add(value)`
(clé auto-incrémentée — **à éviter** dès qu'on a un id métier : risque de
conflits).

## 9.3 Boîte non typée — réservée aux settings

`Box` (sans `<X>`) = tiroir fourre-tout. À réserver à une **box de config**
(`settings`) qui stocke des valeurs de types différents. À la lecture, vérifier
le type (`is`) si incertain.

## 9.4 Variante « Liste Complète » (Get-Modify-Put) — petites listes seulement

Stocker toute la `List<X>` sous **une seule clé**, cycle Get → Modify → Put.
Simple, mais **relit et réécrit toute la liste** à chaque modif.

```dart
Future<void> addTask(Task t) async {
  final box = Hive.box(_boxName);
  final list = (box.get(_key, defaultValue: <Task>[]) as List).cast<Task>();
  list.add(t);
  await box.put(_key, list);
}
```

| Critère | Ops granulaires (9.2) | Liste complète (9.4) |
|---|---|---|
| Performance | ⭐⭐⭐⭐⭐ (1 entrée) | ⭐⭐ (toute la liste) |
| Scalabilité | ⭐⭐⭐⭐⭐ | ⭐ |
| Sécurité de types | ⭐⭐⭐⭐⭐ (`Box<X>`) | ⭐⭐ (`List<dynamic>` + cast) |
| Simplicité | ⭐⭐⭐ | ⭐⭐⭐⭐ |

**Verdict** : ops granulaires **95% du temps** (collections qui grandissent :
tâches, produits, messages…). « Liste complète » uniquement pour de **petites
listes stables** (favoris, 5-10 catégories perso).

## 9.5 Intégration Repository

Le `LocalProvider` fournit `getAll` / `save` / `put` / `delete` au Repository,
qui les branche sur [`readOnlineFirst`](03-lecture-offline-first.md) :
`readLocal: local.getAll`, `cache: local.save`. Le provider renvoie des
**modèles**, jamais de types Hive.
