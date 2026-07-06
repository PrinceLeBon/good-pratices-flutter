# 8. État agnostique du state management

Le Repository renvoie un **résultat en Dart pur** — `Either<Failure, T>` — et
n'importe **jamais** `flutter_bloc`, `get`, `riverpod`… La couche d'état est un
**adaptateur mince** qui mappe ce résultat vers ses propres états.

> Un seul contrat, N adaptateurs → tu changes de state management sans toucher au
> Repository ni aux providers.

## 8.1 Le contrat

```dart
// Repository — aucune dépendance à un state manager.
Future<Either<Failure, List<Aliment>>> getAliments();
```

Réaction fine possible grâce au `Failure` typé (cf. [02](02-reseau-et-erreurs.md)) :
`AuthFailure` → reconnexion, `NetworkFailure` → bandeau hors-ligne, etc.

## 8.2 Cubit (flutter_bloc)

```dart
sealed class AlimentState {}
class AlimentLoading extends AlimentState {}
class AlimentLoaded  extends AlimentState { final List<Aliment> items; AlimentLoaded(this.items); }
class AlimentNeedLogin extends AlimentState {}
class AlimentError   extends AlimentState { final String message; AlimentError(this.message); }

class AlimentCubit extends Cubit<AlimentState> {
  final AlimentRepository repo;
  AlimentCubit(this.repo) : super(AlimentLoading());

  Future<void> load() async {
    emit(AlimentLoading());
    final res = await repo.getAliments();
    res.fold(
      (f) => switch (f) {
        AuthFailure() => emit(AlimentNeedLogin()),
        _             => emit(AlimentError(f.message)),
      },
      (items) => emit(AlimentLoaded(items)),
    );
  }
}
```

## 8.3 GetX

```dart
class AlimentController extends GetxController {
  final AlimentRepository repo;
  AlimentController(this.repo);

  final items = <Aliment>[].obs;
  final error = RxnString();

  Future<void> load() async {
    final res = await repo.getAliments();
    res.fold(
      (f) => error.value = f.message,
      (data) { error.value = null; items.assignAll(data); },
    );
  }
}
```

## 8.4 Riverpod

```dart
final alimentsProvider = FutureProvider<List<Aliment>>((ref) async {
  final res = await ref.read(alimentRepoProvider).getAliments();
  return res.fold((f) => throw f, (data) => data); // AsyncError(Failure) sinon
});
```

## 8.5 ChangeNotifier / `provider`

```dart
class AlimentNotifier extends ChangeNotifier {
  final AlimentRepository repo;
  AlimentNotifier(this.repo);

  List<Aliment> items = [];
  String? error;

  Future<void> load() async {
    final res = await repo.getAliments();
    res.fold((f) => error = f.message, (data) { error = null; items = data; });
    notifyListeners();
  }
}
```

## 8.6 Règle d'or

- Le state layer **ne fait aucun string-matching** d'erreur : il `switch` sur le
  type de `Failure`.
- Aucune logique offline/online dans le state layer — tout est dans le Repository.
- Changer de state manager = réécrire **seulement** l'adaptateur.
